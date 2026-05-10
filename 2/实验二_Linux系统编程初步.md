# 实验2 Linux系统编程初步 - 傻瓜级操作手册

> 每一步都写清楚：按什么键、点什么按钮、输入什么命令。
> 凡是 `像这样` 的文字，就是你要照着输入的内容。
> 凡是【截图】的地方，按键盘上的 **PrtSc**（PrintScreen）键截图。

---

## 开工前准备（做一次就行）

1. 打开 VMware / VirtualBox，启动你的 Ubuntu 虚拟机
2. 等桌面出来后，按键盘 **Ctrl + Alt + T**，打开一个黑色终端窗口
3. 终端打开后，依次输入下面两行，每输完一行按 **回车键**：

```
mkdir -p ~/lab2
cd ~/lab2
```

现在你已经在工作目录了。后面所有操作都在这个终端里完成。

> **粘贴方法**：在终端里按 **Ctrl + Shift + V** 可以粘贴。从本文档复制文字后，到终端里按这个快捷键粘贴。

---

## 项目一：GDB 调试

### 步骤1：创建 greet.c 文件

在终端里输入以下整段（全部复制，然后 Ctrl+Shift+V 粘贴，按回车）：

```
cat > greet.c << 'EOF'
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int display1(char *string);
int display2(char *string);

int main (int argc,char **argv)
{
    char string[] = "Embedded Linux";

    display1 (string);
    display2 (string);

    return 0;
}

int display1 (char *string)
{
    printf ("The original string is %s \n", string);
}

int display2 (char *string1)
{
    char *string2;
    int size,i;

    size = strlen (string1);
    string2 = (char *) malloc (size+1);
    for (i = 0; i < size; i++)
        string2[size - i] = string1[i];
    string2[size+1] = '\0';
    printf("The string afterward is %s\n",string2);
    free(string2);
}
EOF
```

按回车后，终端回到 `$` 提示符，文件就创建好了。

### 步骤2：编译程序（带调试信息）

输入：

```
gcc -g greet.c -o greet
```

按回车。如果没有报错（没有输出任何东西），说明编译成功。

### 步骤3：运行程序，看 bug 现象

输入：

```
./greet
```

按回车。

你会看到输出了 `The original string is Embedded Linux`，但第二行是乱码或者没有正确倒序输出。

【截图】—— 这是报告里"运行结果"那张截图

### 步骤4：启动 GDB

输入：

```
gdb greet
```

按回车。你会看到一堆文字，最后出现 `(gdb)` 提示符。

> 从现在开始，所有命令都在 `(gdb)` 提示符后面输入。

### 步骤5：查看源代码

输入：

```
l
```

按回车。你会看到源代码的前10行。
再按一次 **回车**（会重复上一个命令），看到更多代码。
继续按回车，直到看到所有代码。

【截图】—— 这是报告里"查看源代码"那张截图

### 步骤6：设置断点

依次输入（每输一行按回车）：

```
b 30
```

```
b 33
```

```
info b
```

你会看到列出了两个断点，分别在第30行和第33行。

【截图】—— 这是报告里"断点设置"那张截图

### 步骤7：运行程序

输入：

```
r
```

按回车。程序会运行并在第30行断点处停下。

### 步骤8：单步执行

输入：

```
n
```

按回车。程序执行一行代码。

【截图】—— 这是报告里"单步运行"那张截图

### 步骤9：查看变量值

输入：

```
p string2[size - i]
```

按回车。

【截图】—— 这是报告里"查看变量"那张截图

### 步骤10：继续单步几次

输入 `n` 按回车，再输入：

```
p string2[size-1]
```

按回车。重复几次 `n` 再查看。

### 步骤11：继续运行到第二个断点

输入：

```
c
```

按回车。程序继续运行，在第33行（printf 前）停下。

### 步骤12：查看 string2 各元素

依次输入（每行按回车）：

```
p string2[0]
```

```
p string2[1]
```

```
p string2[2]
```

```
p string2[3]
```

你会发现 `string2[0]` 的值不对。

### 步骤13：在报告里写答案

**答案写这个：**

> display2 函数中 `string2[size - i]` 应改为 `string2[size - 1 - i]`。因为数组下标从 0 开始，字符串长度为 13，第一个字符 'E' 应放到 string2[12]，但原代码放到了 string2[13]，导致 string2[0] 始终未被赋值。另外 `string2[size+1]` 应改为 `string2[size]`，`'\0'` 应放在下标 size 处。

### 步骤14：退出 GDB

输入：

```
q
```

按回车。回到终端 `$` 提示符。

### 步骤15：修复程序并重新运行

复制粘贴以下整段到终端，按回车：

```
cat > greet.c << 'EOF'
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int display1(char *string);
int display2(char *string);

int main (int argc,char **argv)
{
    char string[] = "Embedded Linux";

    display1 (string);
    display2 (string);

    return 0;
}

int display1 (char *string)
{
    printf ("The original string is %s \n", string);
}

int display2 (char *string1)
{
    char *string2;
    int size,i;

    size = strlen (string1);
    string2 = (char *) malloc (size+1);
    for (i = 0; i < size; i++)
        string2[size - 1 - i] = string1[i];
    string2[size] = '\0';
    printf("The string afterward is %s\n",string2);
    free(string2);
}
EOF
```

然后编译运行：

```
gcc -g greet.c -o greet && ./greet
```

按回车。你会看到正确的倒序输出 `xuniL deddedbmE`。

【截图】—— 这是报告里"修改后正确运行结果"那张截图

---

## 项目二：位操作编程

### 步骤1：创建 wei1.c

在终端输入（复制整段粘贴，按回车）：

```
cat > wei1.c << 'EOF'
#include <stdio.h>
main ()
{
    int a=-1;
    printf("\t\t%d\n",sizeof(a));
    printf("\t\t%x\n",a);
}
EOF
```

### 步骤2：编译运行

```
gcc wei1.c -o wei1 && ./wei1
```

按回车。

你会看到输出：
```
        4
        ffffffff
```

【截图】—— 这是报告里"执行结果"截图

### 步骤3：回答问题

**答案：int 型变量是 4 字节。**（sizeof 输出了 4）

### 步骤4：低4位全部置0

输入以下整段，按回车：

```
cat > wei1.c << 'EOF'
#include <stdio.h>
main ()
{
    int a=-1;
    printf("\t\t%d\n",sizeof(a));
    printf("\t\t%x\n",a);
    a = a & ~0xF;
    printf("\t\t%x\n",a);
}
EOF
```

编译运行：

```
gcc wei1.c -o wei1 && ./wei1
```

按回车。输出 `fffffff0`。

【截图】—— 代码窗口截图 + 执行结果截图

### 步骤5：第28位、20位置为0

```
cat > wei1.c << 'EOF'
#include <stdio.h>
main ()
{
    int a=-1;
    printf("\t\t%d\n",sizeof(a));
    printf("\t\t%x\n",a);
    a = a & ~(1<<28);
    a = a & ~(1<<20);
    printf("\t\t%x\n",a);
}
EOF
```

```
gcc wei1.c -o wei1 && ./wei1
```

按回车。输出 `efffefff`。

【截图】—— 代码窗口截图 + 执行结果截图

### 步骤6：a置0，16~19位置1

```
cat > wei1.c << 'EOF'
#include <stdio.h>
main ()
{
    int a=0;
    printf("\t\t%d\n",sizeof(a));
    printf("\t\t%x\n",a);
    a = a | (0xF<<16);
    printf("\t\t%x\n",a);
}
EOF
```

```
gcc wei1.c -o wei1 && ./wei1
```

按回车。输出 `f0000`。

【截图】—— 代码窗口截图 + 执行结果截图

---

## 项目三：makefile应用

### (一) 简单makefile

#### 步骤1：创建5个源文件

把下面5段**全部**依次复制粘贴到终端（每段粘贴后按回车）：

**第1段：**

```
cat > main.c << 'EOF'
#include <stdio.h>
#include "mytool1.h"
#include "mytool2.h"
int main()
{
    mytool1_print("hello,mytool1!");
    mytool2_print("hello,mytool2!");
    return 0;
}
EOF
```

**第2段：**

```
cat > mytool1.c << 'EOF'
#include "mytool1.h"
void mytool1_print(char *print_str)
{
    printf("This is mytool1 print : %s ",print_str);
}
EOF
```

**第3段：**

```
cat > mytool1.h << 'EOF'
#ifndef _MYTOOL_1_H
#define _MYTOOL_1_H
void mytool1_print(char *print_str);
#endif
EOF
```

**第4段：**

```
cat > mytool2.c << 'EOF'
#include "mytool2.h"
void mytool2_print(char *print_str)
{
    printf("This is mytool2 print : %s ",print_str);
}
EOF
```

**第5段：**

```
cat > mytool2.h << 'EOF'
#ifndef _MYTOOL_2_H
#define _MYTOOL_2_H
void mytool2_print(char *print_str);
#endif
EOF
```

#### 步骤2：创建 makefile

> 注意：makefile 中缩进必须是 Tab 键。以下命令已正确处理。

复制粘贴以下整段，按回车：

```
cat > makefile << 'EOF'
main: main.o mytool1.o mytool2.o
	gcc -o main main.o mytool1.o mytool2.o
main.o: main.c
	gcc -c main.c
mytool1.o: mytool1.c mytool1.h
	gcc -c mytool1.c
mytool2.o: mytool2.c mytool2.h
	gcc -c mytool2.c
EOF
```

输入以下命令查看 makefile 内容：

```
cat makefile
```

【截图】—— vi 编辑界面截图（如果老师要求用 vi，可以输入 `vi makefile`，然后按 **Esc** 输入 `:q` 退出）

#### 步骤3：执行 make 编译

```
make
```

按回车。

你会看到三行 gcc 编译命令的输出，然后一行 gcc 链接命令。

【截图】—— 编译过程截图

#### 步骤4：运行生成的程序

```
./main
```

按回车。输出两行 hello 信息。

【截图】—— 运行结果截图

#### 步骤5：再次执行 make

```
make
```

按回车。

你会看到：`make: 'main' is up to date.`

【截图】

**原因：** 所有 .o 文件比源文件新，make 判断不需要重新编译。

### (二) makefile 变量的使用

#### 步骤1：备份并修改 makefile

```
cp makefile makefile.bak1
```

按回车。然后粘贴以下整段，按回车：

```
cat > makefile << 'EOF'
# makefile test for hello program
OBJS = main.o mytool1.o mytool2.o
CC = gcc
main: $(OBJS)
	$(CC) $(OBJS) -o main
main.o: main.c
	$(CC) -c main.c
mytool1.o: mytool1.c mytool1.h
	$(CC) -c mytool1.c
mytool2.o: mytool2.c mytool2.h
	$(CC) -c mytool2.c
EOF
```

#### 步骤2：删 .o 文件并重新编译

```
rm -f *.o main && make
```

按回车。

【截图】—— 编译过程

#### 步骤3：回答——为什么要删 .o 文件？

**原因：** 如果 .o 文件已存在且比源文件新，make 认为目标已是最新不会重新编译。删除后才能看到完整编译过程。

#### 步骤4：测试——$ 后能不能有空格？

```
cat > makefile << 'EOF'
OBJS = main.o mytool1.o mytool2.o
CC = gcc
main: $(OBJS)
	$(CC) $( OBJS) -o main
main.o: main.c
	$(CC) -c main.c
mytool1.o: mytool1.c mytool1.h
	$(CC) -c mytool1.c
mytool2.o: mytool2.c mytool2.h
	$(CC) -c mytool2.c
EOF
```

```
rm -f *.o main && make
```

按回车。会报错。

【截图】—— 报错信息

**答案：不可以有空格。**

#### 步骤5：测试——能不能不写括号？

```
cat > makefile << 'EOF'
OBJS = main.o mytool1.o mytool2.o
CC = gcc
main: $(OBJS)
	$(CC) $O -o main
main.o: main.c
	$(CC) -c main.c
mytool1.o: mytool1.c mytool1.h
	$(CC) -c mytool1.c
mytool2.o: mytool2.c mytool2.h
	$(CC) -c mytool2.c
EOF
```

```
rm -f *.o main && make
```

按回车。会报错或结果不对。

【截图】—— 报错信息

**答案：不可以不括。** `$O` 只取了变量名 `O`（单个字母），而不是 `OBJS`。

### (三) 自动变量及隐式规则

#### 步骤1：使用自动变量

```
cat > makefile << 'EOF'
# makefile test for hello program
OBJS = main.o mytool1.o mytool2.o
CC = gcc
main: $(OBJS)
	$(CC) $^ -o $@
main.o: main.c
	$(CC) -c $< -o $@
mytool1.o: mytool1.c mytool1.h
	$(CC) -c $< -o $@
mytool2.o: mytool2.c mytool2.h
	$(CC) -c $< -o $@
EOF
```

```
rm -f *.o main && make
```

按回车。

【截图】—— 编译输出

#### 步骤2：只保留链接规则（隐式规则）

```
cat > makefile << 'EOF'
# makefile test for hello program
OBJS = main.o mytool1.o mytool2.o
CC = gcc
main: $(OBJS)
	$(CC) $^ -o $@
EOF
```

```
rm -f *.o main && make
```

按回车。make 会自动用隐式规则把 .c 编译成 .o。

【截图】—— 编译输出

### (四) 自主设计编程

#### 第1题：只用自动变量，不用自定义变量

**makefile 内容：**

```
cat > makefile << 'EOF'
main: main.o mytool1.o mytool2.o
	gcc $^ -o $@
main.o: main.c
	gcc -c $< -o $@
mytool1.o: mytool1.c mytool1.h
	gcc -c $< -o $@
mytool2.o: mytool2.c mytool2.h
	gcc -c $< -o $@
EOF
```

```
rm -f *.o main && make
```

按回车。

【截图】—— makefile 内容 + 运行结果

#### 第2题：模式规则

**makefile 内容：**

```
cat > makefile << 'EOF'
CC = gcc
OBJS = main.o mytool1.o mytool2.o
main: $(OBJS)
	$(CC) $^ -o $@
%.o: %.c
	$(CC) -c $< -o $@
EOF
```

```
rm -f *.o main && make
```

按回车。

【截图】—— makefile 内容 + 运行结果

---

## 项目四：shell编程

### 4.1 简单示例 var.sh

#### 步骤1：创建脚本文件

```
cat > var.sh << 'EOF'
#!/bin/bash
myvar="Hello, world"
echo 1= $myvar
echo 2= "$myvar"
echo 3= '$myvar'
echo 4= \$myvar
echo 5= \'$myvar\'
echo 6= "'$myvar'"
echo 7= '"$myvar"'
echo 8= \"$myvar\"
EOF
```

#### 步骤2：加执行权限并运行

```
chmod +x var.sh
```

按回车。然后：

```
./var.sh
```

按回车。

【截图】—— 执行结果

### 4.2 计算1到100的和 + 输出被13整除的数

#### 步骤1：创建脚本

```
cat > sum13.sh << 'EOF'
#!/bin/bash
sum=0
for i in $(seq 1 100)
do
    sum=$((sum + i))
    if [ $((i % 13)) -eq 0 ]; then
        echo "$i"
    fi
done
echo "sum = $sum"
EOF
```

#### 步骤2：运行

```
chmod +x sum13.sh && ./sum13.sh
```

按回车。

你会看到输出：13、26、39、52、65、78、91，最后一行 `sum = 5050`。

【截图】—— 执行结果

### 4.3 建立 class 目录和 stu1~stu5

#### 步骤1：创建脚本

```
cat > mkclass.sh << 'EOF'
#!/bin/bash
mkdir -p ~/class
for i in 1 2 3 4 5
do
    mkdir -p ~/class/stu$i
    chmod 754 ~/class/stu$i
done
echo "Done."
EOF
```

#### 步骤2：运行

```
chmod +x mkclass.sh && ./mkclass.sh
```

按回车。输出 `Done.`

【截图】—— 执行结果

#### 步骤3：查看目录权限

```
ls -l ~/class/
```

按回车。

你会看到 5 个目录，每个权限都是 `drwxr-xr--`：
- 所有者：rwx（读、写、执行）
- 所在组：r-x（读、执行）
- 其他用户：r--（只读）

【截图】—— ls -l 输出结果

---

## 结束

实验完成。你可以用以下命令确认所有文件都在：

```
ls ~/lab2
```

按回车，你会看到所有创建的 .c、.h、.sh 和 makefile 文件。
