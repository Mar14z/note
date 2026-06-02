# Makefile 学习笔记

> 从零开始学习 Makefile，构建自动化工具

## 📋 目录

- [基础语法](#基础语法)
- [变量](#变量)
- [依赖关系](#依赖关系)
- [模式规则](#模式规则)
- [自动变量](#自动变量)
- [常用命令](#常用命令)
- [常见错误](#常见错误)
- [最佳实践](#最佳实践)

---

## 基础语法

### 基本结构

```makefile
目标: 依赖
    命令
```

**注意**：
- 命令前必须使用 **Tab 键**，不能使用空格
- 目标通常是需要生成的文件
- 依赖是目标文件需要的输入

### 简单示例

```makefile
hello: main.c
    gcc main.c -o hello
```

**执行流程**：
1. 检查 `hello` 是否存在
2. 检查 `main.c` 是否存在
3. 如果 `main.c` 比 `hello` 新，执行编译命令

### 多目标示例

```makefile
all: program1 program2

program1: p1.c
    gcc p1.c -o program1

program2: p2.c
    gcc p2.c -o program2

clean:
    rm -f program1 program2
```

---

## 变量

### 定义变量

```makefile
CC = gcc
CFLAGS = -Wall -g
TARGET = hello
```

**命名规则**：
- 只能包含字母、数字、下划线
- 区分大小写
- 推荐使用大写字母

### 使用变量

```makefile
CC = gcc
CFLAGS = -Wall -g

all:
    $(CC) $(CFLAGS) main.c -o hello
```

**展开规则**：
- `$(变量名)` 或 `${变量名}` 展开为变量值
- 递归展开，可以嵌套使用

### 内置变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `CC` | cc | C 编译器 |
| `CXX` | g++ | C++ 编译器 |
| `CFLAGS` | 空 | C 编译选项 |
| `CXXFLAGS` | 空 | C++ 编译选项 |
| `LDFLAGS` | 空 | 链接选项 |

### 变量替换

```makefile
# 把 .c 替换为 .o
SRC = foo.c bar.c
OBJ = $(SRC:.c=.o)
# 结果：foo.o bar.o

# 也可以使用 patsubst 函数
OBJ = $(patsubst %.c, %.o, $(SRC))
```

### 追加变量

```makefile
CFLAGS = -Wall
CFLAGS += -g  # 追加内容
```

---

## 依赖关系

### 显式规则

```makefile
main.o: main.c defs.h
    gcc -c main.c
```

**含义**：
- `main.o` 依赖 `main.c` 和 `defs.h`
- 如果任何一个依赖比 `main.o` 新，重新编译

### 隐式规则

Make 内置的规则，自动推导：

```makefile
# Make 会自动执行：
# main.o: main.c
#     gcc -c main.c
main.o: main.c
```

### 多级依赖

```makefile
app: main.o foo.o bar.o
    gcc main.o foo.o bar.o -o app

main.o: main.c defs.h
    gcc -c main.c

foo.o: foo.c defs.h
    gcc -c foo.c

bar.o: bar.c
    gcc -c bar.c
```

**Make 的自动解析**：
- 自动分析依赖关系
- 按正确顺序编译

---

## 模式规则

### 什么是模式规则

使用 `%` 通配符，匹配一组文件：

```makefile
%.o: %.c
    gcc -c $< -o $@
```

**含义**：
- 任何 `.c` 文件编译成对应的 `.o` 文件
- 例如：`foo.c` → `foo.o`

### 完整示例

```makefile
CC = gcc
CFLAGS = -Wall -g
TARGET = app
SRC = $(wildcard *.c)      # 获取所有 .c 文件
OBJ = $(SRC:.c=.o)         # 替换为 .o 文件

all: $(TARGET)

$(TARGET): $(OBJ)
    $(CC) $(OBJ) -o $(TARGET)

%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@

clean:
    rm -f $(OBJ) $(TARGET)

.PHONY: all clean
```

**特点**：
- `wildcard` 函数：获取匹配的文件
- 模式规则：通用编译规则
- 变量替换：`.c` → `.o`

---

## 自动变量

### 常用自动变量

| 变量 | 含义 | 示例 |
|------|------|------|
| `$@` | 目标文件名 | main.o |
| `$<` | 第一个依赖 | main.c |
| `$^` | 所有依赖 | main.c defs.h |
| `$+` | 所有依赖（保留顺序） | main.c defs.h |
| `$*` | 去除扩展名的目标 | main |
| `$?` | 比目标新的依赖 | main.c |

### 示例

```makefile
main.o: main.c defs.h utils.h
    # $@ = main.o
    # $< = main.c
    # $^ = main.c defs.h utils.h
    gcc -c $< -o $@
```

### 模式匹配的自动变量

```makefile
%.o: %.c
    # $< 展开为匹配的 .c 文件
    # $@ 展开为匹配的 .o 文件
    # $* 展开为去除扩展名的文件名
    gcc -c $< -o $@
```

---

## 常用命令

### make 命令

```bash
make              # 执行第一个目标
make all          # 执行 all 目标
make clean         # 执行 clean 目标
make TARGET        # 执行指定目标
```

### 常用选项

```bash
make -n            # 不执行，只显示要执行的命令
make -B            # 强制重新编译所有目标
make -j N          # 并行编译，最多 N 个任务
make -C DIR        # 切换到 DIR 目录后再执行
make -f FILE       # 使用指定的 Makefile
```

### 示例

```bash
# 查看编译命令但不执行
make -n

# 4 核并行编译
make -j4

# 使用其他 Makefile
make -f Makefile.linux
```

---

## 常见错误

### 错误 1：Tab vs 空格

```makefile
# ❌ 错误：使用空格
main.o: main.c
    gcc -c main.c  # 这里是空格

# ✅ 正确：使用 Tab
main.o: main.c
    gcc -c main.c  # 这里是 Tab
```

**检测方法**：
```bash
cat -A Makefile  # 显示 Tab 为 ^I
```

### 错误 2：文件名拼写错误

```makefile
# ❌ 错误：逗号而非点
src/main,c -o program

# ✅ 正确：点
src/main.c -o program
```

### 错误 3：循环依赖

```makefile
# ❌ 错误：自己依赖自己
A: B
B: A

# ✅ 正确：有向无环图
A: B
B: C
```

### 错误 4：忘记依赖

```makefile
# ❌ 错误：缺少头文件依赖
main.o: main.c
    gcc -c main.c

# ✅ 正确：包含所有依赖
main.o: main.c defs.h utils.h
    gcc -c main.c
```

### 错误 5：伪目标未声明

```makefile
# ❌ 可能出错：如果存在 clean 文件
clean:
    rm -f *.o

# ✅ 正确：声明为伪目标
.PHONY: clean
clean:
    rm -f *.o
```

---

## 最佳实践

### 1. 使用变量

```makefile
# ❌ 不推荐：硬编码
all:
    gcc -Wall -g src/main.c -o sysmonitor

# ✅ 推荐：使用变量
CC = gcc
CFLAGS = -Wall -g
TARGET = sysmonitor

all:
    $(CC) $(CFLAGS) src/main.c -o $(TARGET)
```

### 2. 使用模式规则

```makefile
# ❌ 不推荐：重复代码
main.o: main.c
    $(CC) $(CFLAGS) -c main.c
foo.o: foo.c
    $(CC) $(CFLAGS) -c foo.c

# ✅ 推荐：模式规则
%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@
```

### 3. 分离编译和链接

```makefile
# 分别编译 .o 文件，最后链接
$(TARGET): $(OBJ)
    $(CC) $(OBJ) -o $(TARGET)

%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@
```

### 4. 使用函数

```makefile
# 获取所有 .c 文件
SRC = $(wildcard *.c)

# 获取所有 .o 文件
OBJ = $(SRC:.c=.o)

# 模式替换
OBJ = $(patsubst %.c, %.o, $(SRC))
```

### 5. 声明伪目标

```makefile
.PHONY: all clean install test
```

### 6. 添加注释

```makefile
# 编译器设置
CC = gcc
CFLAGS = -Wall -g

# 目标文件
TARGET = app
OBJ = $(SRC:.c=.o)

# 默认目标
all: $(TARGET)

# 链接
$(TARGET): $(OBJ)
    $(CC) $(OBJ) -o $(TARGET)

# 编译
%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@

# 清理
clean:
    rm -f $(OBJ) $(TARGET)
```

### 7. 增量编译

```makefile
# 修改文件后，只重新编译需要的部分
# 未修改的文件不会被重新编译
# 提高编译效率
```

---

## 进阶主题

### 条件判断

```makefile
ifeq ($(DEBUG), 1)
    CFLAGS = -g -O0
else
    CFLAGS = -O2
endif
```

### 函数

```makefile
# 字符串替换
$(subst from,to,text)

# 模式替换
$(patsubst pattern,replacement,text)

# 查找文件
$(wildcard pattern)

# 过滤
$(filter pattern,text)

# 去重
$(words text)
$(sort text)
```

### 多 Makefile

```makefile
# 主 Makefile 包含其他 Makefile
include config.mk
include rules.mk
```

### 递归调用

```bash
# 在子目录中执行 make
$(MAKE) -C subdir
```

---

## 参考资料

- GNU Make 官方文档：https://www.gnu.org/software/make/
- 《Managing Projects with GNU Make》
- `make --help` 或 `man make`

---

## 标签

#Makefile #构建工具 #自动化 #C语言 #开发工具

---

**创建时间**：2026-05-16
**更新时间**：2026-05-16
