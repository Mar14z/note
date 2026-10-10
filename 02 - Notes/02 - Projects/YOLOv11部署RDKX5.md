# YOLOv11 部署到 RDK X5 项目笔记

> 将自训练 YOLOv11 目标检测模型部署到地平线 RDK X5 开发板
> 主机环境：Ubuntu 物理机 / WSL2 ｜ 板端：RDK X5（Bayes-e BPU, 10 TOPS）

## � 角色分工（必读）

| 设备 | 角色 | 装什么 | 做什么 |
|------|------|--------|--------|
| **开发机**（x86 Ubuntu / WSL2） | 工具链宿主 | ✅ Docker、✅ OE 工具链 | 模型转换、量化 |
| **RDK X5 板**（ARM aarch64） | 推理设备 | ❌ **不装 Docker**、装 RDK OS | 加载 .bin 跑推理 |

> [!warning] Docker 不装在板上
> OE 工具链的 Docker 镜像是 x86_64 的，X5 是 ARM aarch64，**架构不兼容**。官方设计就是"**PC 量化 → 板子推理**"。

**数据流**：`开发机: .pt → .onnx → .bin ──scp──> 板子: BPU 推理 → 结果`

## �📋 项目信息

| 属性 | 内容 |
|------|------|
| **项目名称** | YOLOv11 on RDK X5 |
| **任务** | 目标检测（人/车/物） |
| **起点** | 已有 PyTorch `.pt` 权重 |
| **起点模型** | YOLOv11n（2.6M 参数） |
| **目标板子** | RDK X5（Ubuntu 22.04） |
| **目标精度** | mAP50 ≥ 0.65 |
| **目标性能** | ≥ 30 FPS 端到端 |
| **工具链版本** | OE 天工开物 v1.2.8+ (Docker) |
| **BPU 架构** | Bayes-e |
| **状态** | 🟡 计划中 |

## 🎯 目标拆解

- [ ] ONNX 导出成功（用修改版 head）
- [ ] hb_mapper PTQ 转换无错误
- [ ] 校准数据准备完成（50~200 张）
- [ ] .bin 模型在板端加载成功
- [ ] 单张图片推理跑通
- [ ] USB 摄像头实时检测跑通
- [ ] 端到端 FPS ≥ 30
- [ ] 精度 mAP50 ≥ 0.65
- [ ] 整理一份 README 上传 GitHub

## 📁 目录结构（规划）

```
~/rdk_x5_yolo11/
├── models/                      # .pt / .onnx 源模型
│   ├── best.pt
│   └── best.onnx
├── calibration_data/            # 校准图片
│   └── calibration_f32/         # 转好的 .f32
├── output/                      # 量化产物
│   ├── yolo11n.bin
│   ├── yolo11n.yaml
│   └── logs/
├── tools/                       # 转换 / 验证脚本
│   ├── export_for_rdkx5.py      # 改 head 导 ONNX
│   ├── calibration_prep.py      # 校准数据预处理
│   └── infer_test.py            # 板端推理测试
├── docker/                      # 工具链 Docker
│   ├── start_oe.sh
│   └── mapper_config.yaml
└── README.md
```

---

## 🚀 部署流程

### 步骤 1：环境准备（Ubuntu 物理机/WSL2）

```bash
# 1.1 安装 Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo usermod -aG docker $USER  # 免 sudo

# 1.2 创建工作目录
mkdir -p ~/rdk_x5_yolo11/{models,calibration_data,output,tools,docker}
cd ~/rdk_x5_yolo11

# 1.3 下载 OE 工具链包
# 从 https://developer.d-robotics.cc/rdk_doc/Quick_start/download 下载
# horizon_x5_open_explorer_v1.2.8-py310_20240926.tar.gz

# 1.4 加载 Docker 镜像
tar -xzf horizon_x5_open_explorer_v1.2.8-py310_20240926.tar.gz
cd horizon_x5_open_explorer_v1.2.8-py310_20240926/docker_images
docker load < openexplorer_ai_toolchain_ubuntu_20_x5_cpu_v1.2.8-py310.tar.gz
docker images | grep openexplorer
```

### 步骤 2：导出 ONNX（⚠️ 关键）

```python
# tools/export_for_rdkx5.py
import os
os.environ["PYTORCH_ONNX_USE_LEGACY_EXPORTER"] = "1"
import warnings
warnings.filterwarnings("ignore")
import torch
from ultralytics import YOLO

# 关键：先卸载 ultralytics 再编辑 head.py
# pip uninstall ultralytics
# 编辑 ultralytics/nn/modules/head.py，替换 Detect.forward 为：

# def forward(self, x):
#     bboxes = [self.cv2[i](x[i]).permute(0, 2, 3, 1).contiguous() for i in range(self.nl)]
#     clses  = [self.cv3[i](x[i]).permute(0, 2, 3, 1).contiguous() for i in range(self.nl)]
#     return (bboxes, clses)

# 重新装回来
# pip install ultralytics

MODEL_PATH = "models/best.pt"
ONNX_PATH = "models/best.onnx"

model = YOLO(MODEL_PATH)
model.export(
    format="onnx",
    imgsz=640,
    opset=11,         # ⚠️ 必须是 10 或 11
    simplify=False,   # ⚠️ 不要 simplify
    dynamic=False,    # ⚠️ 关闭动态 shape
)
```

> [!warning] 关键参数
> - `opset=11` —— X5 只支持 opset 10/11
> - `simplify=False` —— onnx-simplifier 会破坏 head 结构
> - `dynamic=False` —— BPU 不支持动态 shape

### 步骤 3：准备校准数据

```python
# tools/calibration_prep.py
import os
import cv2
import numpy as np

src_dir = "datasets/images/val"   # 从训练集抽 100~200 张
dst_dir = "calibration_data/calibration_f32"
os.makedirs(dst_dir, exist_ok=True)

img_size = 640
files = sorted(os.listdir(src_dir))[:200]  # 取 200 张

for i, f in enumerate(files):
    img = cv2.imread(os.path.join(src_dir, f))
    img = cv2.resize(img, (img_size, img_size))
    # ⚠️ BPU 内部会 /255，这里不能再 /255
    img = img.astype(np.float32)  # 范围 0~255
    img = img.transpose(2, 0, 1)  # HWC → CHW
    img.tofile(os.path.join(dst_dir, f"{i:06d}.f32"))

print(f"Done: {len(files)} calibration files")
```

### 步骤 4：OE Docker 量化

```bash
# 4.1 启动 Docker
cat > docker/start_oe.sh << 'EOF'
#!/bin/bash
docker run -it --rm --shm-size=15g \
  -v $(pwd):/open_explore \
  -v $(pwd)/calibration_data:/data \
  openexplorer/ai_toolchain_ubuntu_20_x5_cpu:v1.2.8-py310
EOF
chmod +x docker/start_oe.sh
./docker/start_oe.sh

# 4.2 mapper 配置文件
cat > docker/mapper_config.yaml << 'EOF'
model_type: onnx
march: bayes-e
input_model: /open_explore/models/best.onnx
output_model: /open_explore/output/yolo11n.bin

input_shape: 1x3x640x640
input_type_rt: nv12
input_type_train: rgb

calibration_data: /data/calibration_f32
calibration_type: max

# YOLOv11 关键：DFL 必须在板端做
node_params:
  # 显式标记 DFL/Sigmoid 跑 CPU
  ...
EOF

# 4.3 校验 ONNX
hb_mapper checker --model-type onnx --march bayes-e \
  --model /open_explore/models/best.onnx

# 4.4 转 .bin
hb_mapper makertbin --model-type onnx \
  --config /open_explore/docker/mapper_config.yaml
```

### 步骤 5：板端部署

```bash
# 5.1 传 .bin 到板子
scp output/yolo11n.bin sunrise@<rdk_ip>:/home/sunrise/models/

# 5.2 SSH 登录板子
ssh sunrise@<rdk_ip>

# 5.3 验证 BPU 运行时
python3 -c "from hobot_dnn import pyeasy_dnn; print('hobot_dnn OK')"
# 或（OS ≥ 3.5.0）
python3 -c "import hbm_runtime; print('hbm_runtime OK')"
```

### 步骤 6：推理测试

**方式 A：hbm_runtime**（推荐，新版 OS）

```python
# tools/infer_test.py（在板子上跑）
import cv2
import numpy as np
import hbm_runtime

model = hbm_runtime.HB_HBMRuntime("/home/sunrise/models/yolo11n.bin")
model_name = model.model_names[0]

# 设置调度
model.set_scheduling_params(
    priority={model_name: 0},
    bpu_cores={model_name: [0]},
)

# 加载图片
img = cv2.imread("test.jpg")
img_nv12 = ...  # BGR → NV12 转换

# 推理
inputs = {model_name: {model.input_names[model_name][0]: img_nv12}}
outputs = model.run(inputs)

# 后处理（DFL decode + NMS）
# 参考 rdk_model_zoo 的 ultralytics_yolo wrapper
```

**方式 B：TROS 集成**（机器人系统）

```bash
# 安装 TROS YOLO 包
sudo apt install -y tros-humble-hobot-dnn
sudo apt install -y tros-humble-dnn-node

# 准备配置
cat > ~/yolo11_config.json << 'EOF'
{
  "model_file": "/home/sunrise/models/yolo11n.bin",
  "dnn_Parser": "yolov8",
  "model_input_type": "nv12",
  "input_width": 640,
  "input_height": 640,
  "model_output_count": 6,
  "reg_max": 16,
  "class_num": <你的类别数>,
  "strides": [8, 16, 32],
  "score_threshold": 0.25,
  "nms_threshold": 0.7,
  "nms_top_k": 300
}
EOF

# 启动 USB 摄像头 + YOLO 推理
ros2 launch dnn_node_example usb_websocket_display.launch.py \
  dnn_example_config_file:=/home/sunrise/yolo11_config.json
```

---

## 🐛 踩坑清单（边做边记）

### 性能类

- ❌ **YOLOv11 默认 ONNX 会让 BPU 性能暴跌到 7 FPS** —— 原因：DFL/Sigmoid/NMS 在 BPU 不支持，被切成十几个子图反复搬运。必须用改 head 后的 ONNX（社区实测 47 FPS）。
- ⚠️ **YOLOv11m 在 X5 上官方 28.95 FPS**——比 YOLOv8n 慢很多，因为 DFL 复杂。如对性能敏感建议用 YOLOv8n。
- ⚠️ **YOLOv12m 在 X5 上 74 FPS**——v12 反而比 v11 快，可考虑替换。

### 兼容性类

- ❌ **opset ≥ 12 会报错** —— X5 只支持 10/11
- ❌ **校准数据双重归一化** —— max < 2.0 表明被 /255 了
- ❌ **训练 < 100 epoch 量化会崩** —— 官方经验：cls 输出会全为强负数
- ❌ **dnn_Parser 写 yolov11** —— 应该是 `"yolov8"`（v11/12 复用）

### 板端 API 类

- ⚠️ **旧版（OS < 3.5.0）用 `hobot_dnn` / `pyeasy_dnn`**
- ⚠️ **新版（OS ≥ 3.5.0）用 `hbm_runtime`**——接口更清晰
- ⚠️ **TROS V2.5.2+ 才完整支持 YOLOv11/12 集成**

### 环境类

- ⚠️ **OS 升级后必须跑 `rdk-miniboot-update`** —— 否则新 kernel 不生效
- ⚠️ **Docker 镜像版本要匹配 OE 工具链版本** —— 用错版本会报 hbrt library 兼容性警告

---

## 📊 性能记录（实测填表）

| 模型 | 量化方式 | 输入 | 板端 FPS | mAP50 | 备注 |
|------|----------|------|----------|-------|------|
| YOLOv11n | PTQ | 640×640 | _待测_ | _待测_ | |
| | | | | | |

---

## 📚 参考资料

### 官方
- [RDK X5 Model Zoo 使用说明](https://developer.d-robotics.cc/rdk_doc/rdk_s/Algorithm_Application/model_zoo/rdk_x5_guide)
- [TROS YOLO 集成文档](https://developer.d-robotics.cc/rdk_doc/en/rdk_s/Robot_development/boxs/detection/yolo/)
- [rdk_model_zoo GitHub](https://github.com/D-Robotics/rdk_model_zoo) → `rdk_x5` 分支
- [OE 工具链手册](https://developer.d-robotics.cc/api/v1/fileData/x5_doc-v126cn/index.html)

### 社区实战
- [7→47 FPS 性能优化之路（CSDN）](https://blog.csdn.net/hueuyang/article/details/155410591)
- [YOLOv11 保姆级教程（官方论坛）](https://forum.d-robotics.cc/t/topic/34084)
- [万字长文部署教程（博客园）](https://www.cnblogs.com/SkyXZ/p/18681804)

### 笔记内链
- [[../03 - Areas/地平线BPU/旭日5算法工具链]] —— OE 工具链详细
- [[../03 - Areas/地平线BPU/NPU横向对比]] —— NPU 加速器横向对比
- [[../04 - Resources/AI模型部署/板端部署]] —— 板端部署方法论
- [[../04 - Resources/AI模型部署/模型量化]] —— PTQ/QAT 原理

---

## 📝 进展日志

### 2026-06-05
- 立项，确定目标：YOLOv11 → RDK X5，Ubuntu 主机
- 完成 OE 工具链调研
- 创建项目笔记

### （后续记录）
