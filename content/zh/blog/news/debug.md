---
title: Debug
date: 2025-05-29
---

# 使用 GDB 调试 Hello World 程序

我将以一个简单的 C 语言 Hello World 程序为例，简洁地说明如何使用 GDB 调试。以下是步骤和关键命令：

## 1. 准备 Hello World 程序

创建 `hello.c`：

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    int x = 10;
    x = x + 5;
    printf("x = %d\n", x);
    return 0;
}
```

## 2. 编译程序
使用 `-g` 选项编译以包含调试信息：

```bash
gcc -g -o hello hello.c
```

## 3. 启动 GDB
运行 GDB 并加载程序：

```bash
gdb ./hello
```

进入 GDB 交互界面：

```bash
(gdb)
```

## 4. 调试步骤
以下是调试 hello.c 的过程：

### (1) 设置断点
在 main 函数设置断点：

```bash

(gdb) break main
Breakpoint 1 at 0x40111b: file hello.c, line 4.
```

### (2) 运行程序
运行程序，暂停在断点：
```bash

(gdb) run
Starting program: ./hello
Breakpoint 1, main () at hello.c:4
4           printf("Hello, World!\n");
```

### (3) 单步执行
逐行执行（next 不进入函数内部）：
```bash

(gdb) next
Hello, World!
5           int x = 10;
```

### (4) 查看变量
检查变量 x：
```bash

(gdb) print x
$1 = 10
```

继续执行：
```bash

(gdb) next
6           x = x + 5;
(gdb) next
7           printf("x = %d\n", x);
(gdb) print x
$2 = 15
```

### (5) 继续运行
继续执行直到结束：
```bash

(gdb) continue
Continuing.
x = 15
[Inferior 1 (process 12345) exited normally]
```

### (6) 退出 GDB
退出：
```bash

(gdb) quit
```



