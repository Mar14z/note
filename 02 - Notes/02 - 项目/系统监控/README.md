---
title: 系统监控
type: 项目
created: 2026-05-16
updated: 2026-10-10
tags:
  - 项目
  - C语言
  - Linux
status: 进行中
---

# SysMonitor 项目笔记

> Linux系统监控工具项目学习记录

## 📋 项目信息

| 属性 | 内容 |
|------|------|
| 项目名称 | SysMonitor |
| 编程语言 | C (核心) + Python (辅助) |
| 目标平台 | Linux |
| 学习目的 | 掌握Linux系统编程 |

## 📁 项目结构

```
sysmonitor/
├── src/
│   ├── main.c              # 主程序入口
│   ├── collector/          # 数据采集模块
│   ├── cache/              # 数据缓存
│   ├── alert/              # 告警系统
│   ├── ui/                 # 用户界面
│   └── utils/              # 工具函数
├── include/
│   └── common.h            # 公共头文件
├── tests/                  # 测试代码
└── Makefile               # 构建脚本
```

## ✅ 已完成

### 阶段1：项目初始化

- [x] 创建目录结构
- [x] 创建公共头文件 `include/common.h`
- [x] 定义程序常量（名称、版本、路径长度）
- [x] 创建主程序框架 `src/main.c`

### 阶段2：Makefile 构建系统

- [x] 理解 Makefile 基本语法
- [x] 掌握变量定义与使用
- [x] 实现依赖关系与增量编译
- [x] 学会使用模式规则和自动变量
- [x] 修复常见错误（逗号 vs 点）

### 学到的知识点

详见：[[C语言基础]] 和 [[Makefile学习]]

## 🔄 进行中

### main.c 开发

**当前状态**：正在完善主程序框架

**待完成任务**：
1. 修正头文件引用方式（`<stdio.h>` 而非 `"stdio.h"`）
2. 添加命令行参数处理
3. 实现 `-h/--help` 帮助信息

## 📝 开发记录

### 2026-05-16

**完成内容**：
- 创建项目目录结构
- 编写 `common.h` 头文件
- 编写 `main.c` 基础框架

**遇到的问题**：
- Windows PowerShell 和 Linux Bash 语法差异
- 头文件引用方式混淆（`<>` vs `""`）

**解决方案**：
- 系统头文件使用尖括号 `<>`
- 本地头文件使用双引号 `""`

## 📚 参考资料

- `/proc/` 文件系统文档
- Linux 系统调用手册
- GCC 编译选项文档
- GNU Make 手册

---

## 📖 Makefile 核心知识点

### 1. 基本语法

```makefile
目标: 依赖
    命令（必须用 Tab 缩进）
```

**示例**：
```makefile
hello: main.c
    gcc main.c -o hello
```

### 2. 变量定义和使用

| 操作 | 语法 | 说明 |
|------|------|------|
| 定义 | `CC = gcc` | 使用 `=` 定义变量 |
| 使用 | `$(CC)` | 使用 `$(变量名)` 展开变量 |

**示例**：
```makefile
CC = gcc
CFLAGS = -Wall -g
TARGET = sysmonitor

all:
    $(CC) $(CFLAGS) src/main.c -o $(TARGET)
```

### 3. 常见编译选项

| 选项 | 含义 | 用途 |
|------|------|------|
| `-Wall` | 开启所有警告 | 及时发现潜在问题 |
| `-g` | 包含调试信息 | 支持 GDB 调试 |
| `-c` | 只编译不链接 | 生成 .o 目标文件 |
| `-o` | 指定输出文件名 | 控制生成的可执行文件名 |

### 4. 依赖关系与增量编译

**核心原理**：Make 会比较目标文件和依赖文件的时间戳，只重新编译修改过的文件。

**依赖格式**：
```makefile
目标文件: 依赖文件
    命令
```

**示例**：
```makefile
main.o: main.c common.h
    gcc -c main.c -o main.o
```

### 5. 模式规则与自动变量

**模式规则**：
```makefile
%.o: %.c
    gcc -c $< -o $@
```

**自动变量**：
| 变量 | 含义 | 示例 |
|------|------|------|
| `$@` | 目标文件 | main.o |
| `$<` | 第一个依赖 | main.c |
| `$^` | 所有依赖 | main.c common.h |

### 6. 变量替换

```makefile
SRC = src/main.c
OBJ = $(SRC:.c=.o)  # 把 .c 替换为 .o，结果：src/main.o
```

### 7. 伪目标声明

```makefile
.PHONY: all clean
```

**作用**：避免文件名冲突，确保目标总是被执行。

### 8. 常见 make 命令

| 命令 | 作用 |
|------|------|
| `make` | 执行默认目标（第一个） |
| `make all` | 执行 all 目标 |
| `make clean` | 清理生成的文件 |
| `make <目标>` | 执行指定目标 |

### 9. 遇到的问题与解决方案

#### 问题1：逗号 vs 点

```makefile
# ❌ 错误
src/main,c -o sysmonitor

# ✅ 正确
src/main.c -o sysmonitor
```

**教训**：`.c` 是点，不是逗号！

#### 问题2：Tab vs 空格

Makefile 的命令行必须使用 **Tab 键**，不能使用空格。

### 10. 最佳实践

1. **总是使用变量** - 提高可维护性
   ```makefile
   CC = gcc
   CFLAGS = -Wall -g
   ```

2. **使用模式规则** - 减少重复代码
   ```makefile
   %.o: %.c
       $(CC) $(CFLAGS) -c $< -o $@
   ```

3. **声明伪目标** - 避免冲突
   ```makefile
   .PHONY: clean all
   ```

4. **增量编译** - 提高效率
   - 源文件修改 → 只重新编译该文件
   - 未修改文件 → 不重新编译

### 11. 当前项目的 Makefile

```makefile
CC = gcc
CFLAGS = -Wall -g
TARGET = sysmonitor
SRC = src/main.c
OBJ = $(SRC:.c=.o)

all: $(TARGET)

$(TARGET): $(OBJ)
	@echo "Linking..."
	$(CC) $(OBJ) -o $(TARGET)
	@echo "Done!"

%.o: %.c
	@echo "Compiling $<..."
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJ) $(TARGET)

.PHONY: all clean
```

**特点**：
- 支持增量编译
- 使用模式规则自动编译所有 .c 文件
- 分离编译和链接步骤
- 使用伪目标避免冲突

---

**关键词**：#Makefile #C语言 #构建工具 #自动化 #SysMonitor

**相关笔记**：
- [[C语言基础]] - GCC 编译选项
- [[Linux系统编程]] - /proc 文件系统
- [[Makefile学习]] - Makefile 详细语法

**创建时间**：2026-05-16
**更新时间**：2026-05-16
