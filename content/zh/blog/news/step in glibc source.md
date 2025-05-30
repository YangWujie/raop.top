---
title: Step in Glibc Source Code
date: 2025-05-29
---


在 Ubuntu 22.04 下，想要在 gdb 调试时 step in 到 `printf` 函数内部，需要满足两个条件：

1. **glibc（libc）必须带有调试符号**，否则只能看到汇编或提示“no source available”。
2. **gdb 能找到 glibc 的源码路径**，这样才能显示 C 语言源码而不是汇编。

---

### 步骤一：安装调试符号和源码

```sh
sudo apt update
sudo apt install libc6-dbg glibc-source
```

---

### 步骤二：解压 glibc 源码

```sh
sudo tar -xf /usr/src/glibc/glibc-*.tar.xz -C /usr/src/glibc/
```
解压后源码路径通常为 `/usr/src/glibc/glibc-<version>/`

---

### 步骤三：配置 gdb 的源码路径映射

glibc 在编译时源码路径通常是 `/build/glibc-*/`，而你本地源码在 `/usr/src/glibc/glibc-<version>/`。  
你需要告诉 gdb：**把源码路径替换为本地实际路径**。

在 gdb 里输入：

```
set substitute-path /build/glibc-*/ /usr/src/glibc/glibc-<version>/
```
请将 `<version>` 替换为实际解压出来的目录名，比如 `2.35`.

注：上面的办法不管用。用下面的：

```
directory /usr/src/glibc/glibc-2.35/
```

有，可以通过以下方法让 gdb 每次自动加载源码路径，无需每次手动设置：

在你的主目录下创建或编辑 `~/.gdbinit` 文件，加入如下内容：

```
directory /usr/src/glibc/glibc-2.35/
```

这样每次启动 gdb 时都会自动添加该源码路径。

如果只想对某个项目生效，可以在项目根目录下创建 `.gdbinit` 文件，内容同上。gdb 启动时会自动读取当前目录下的 `.gdbinit`。

**注意：**  
- `~/.gdbinit` 影响所有 gdb 会话。  
- 项目目录下的 `.gdbinit` 只影响该目录下的调试。

---

### 步骤四：gdb 调试

1. 启动 gdb 并运行你的程序。
2. 当执行到 `printf` 时，使用 `step` 或 `stepi` 进入。
3. 如果一切正常，gdb 会显示 `printf` 的 glibc 源码。

---

### 说明

- `set substitute-path <编译时路径> <本地路径>`  
  这条命令告诉 gdb：当调试信息里出现 `<编译时路径>` 时，去 `<本地路径>` 查找源码。
- 如果还是进不去源码，可能是优化级别太高（如 `-O2`），可尝试用 `-O0 -g` 重新编译你的程序。

---

**总结：**  
只要安装了调试符号和源码，并正确配置 substitute-path，就可以 step in 到 `printf` 这样的 glibc 函数源码。