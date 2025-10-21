# 编译相关

##  Make

### -Werror问题：
Wrong -> error,  警告都会被当成编译错误，使得编译退出。
### .PHONY 是什么
`.PHONY` 伪目标”（phony targets）。
- **伪目标**：它们通常是一些操作命令，比如：`clean`, `install`, `all` 等。
- 使用 `.PHONY` 可以避免与真实文件名冲突，并让 `make` 正确地执行这些目标。
- 举例：如果你不加 `.PHONY`，当当前目录下有一个叫 `clean` 的文件时，`make clean` 就不会执行你定义的命令。

> 如何编译出一个 动态链接库(.so)文件？
> 如何利用.so文件编译新的程序？

make一个libprefetchus.so：
```shell
CC=g++

CFLAGS=`pkg-config --cflags glib-2.0` -Wall -g -fPIC

LDFLAGS=`pkg-config --libs glib-2.0` -shared -lstdc++

CPPFLAGS=-I./include -I./include/dataStructure/hashtable -IDAS/include

TARGET=libprefetchus.so
```
-fPIC：生成位置无关代码
-shared：决定库是否为动态（.a / .so)

使用某so进行make
```shell
CC=gcc
CFLAGS=-std=c99 -Wall -g
LDFLAGS=-ldl

test: test.c
	$(CC) $(CFLAGS) -o test test.c $(LDFLAGS)

clean:
	rm -f test

.PHONY: clean 
```


## Cmake
用 cmake 编译 C++工程时，如果改变了工程文件的位置，那么在 build 文件中运行 cmake … 时有可能会报错，我们删除 build 文件夹（cmake 没有 clear 功能）

```shell
#1.退出build目录：
cd ..
#2.删除build目录（要确保build中只存放了编译生成的中间文件）
rm -rf build
#3.新建build目录
mkdir build
#4.进入build
cd build
#5.重新cmake …
cmake ..
```

## autoconf (./configure)

### **编写 `Makefile.am`**
定义编译规则（由 Automake 处理）：
bin_PROGRAMS = myapp        # 要生成的可执行文件
myapp_SOURCES = main.c      # 源文件列表
###  **运行 Autotools 生成 `configure`**
依次执行以下命令：
```bash
aclocal                  # 生成 `aclocal.m4`（宏集合）
autoconf                 # 根据 `configure.ac` 生成 `configure` 脚本
automake --add-missing   # 生成 `Makefile.in` 和其他辅助文件
```
最终会生成：
- `configure` 脚本
- `Makefile.in`（模板，供 `configure` 生成最终的 `Makefile`）
- 
### **编译项目:**
```bash
./configure # 用户可以通过 `./configure --options` 定制编译参数。
make
sudo make install
```


# 其他linux常用调式命令

```shell
# 查看该动态库的符号
nm -D libprefetchus.so | grep -i "U " | head -20
# 查看某动态链接库导出的函数
nm -D libprefetchus.so | grep -E "(das_prefetchus|T )"
nm -D libprefetchus.so | grep das_prefetchus

```

