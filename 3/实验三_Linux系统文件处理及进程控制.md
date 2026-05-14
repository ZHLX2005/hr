# 实验3 Linux系统文件处理及进程控制 - 傻瓜级操作手册

> 每一步都写清楚：按什么键、点什么按钮、输入什么命令。
> 凡是 `像这样` 的文字，就是你要照着输入的内容。
> 凡是【截图】的地方，按键盘上的 **PrtSc**（PrintScreen）键截图。

---

## 开工前准备（每次新开终端都要做）

1. 打开 VMware / VirtualBox，启动 Ubuntu 虚拟机
2. 等桌面出来后，按 **Ctrl + Alt + T**，打开终端
3. 依次输入下面两行，每行按 **回车**：

```
mkdir -p ~/lab3
cd ~/lab3
```

> **粘贴方法**：在终端里按 **Ctrl + Shift + V** 粘贴。从本文档复制文字后到终端里按这个快捷键。

---

## 项目一：编写 copy 程序完成文件复制

### 步骤1：创建 copy.c 文件

复制下面整段，粘贴到终端，按回车：

```
cat > copy.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#define maxsize 256

int main(int argc,char *argv[])
{
    int fd1,fd2;
    char buff[maxsize];
    int i;

    if(argc!=3)
    {
        printf("command error!\n");
        return -1;
    }

    fd1=open(argv[1],O_RDONLY);
    if(fd1==-1)
    {
        printf("file %s cannot open",argv[1]);
        return -1;
    }

    if((fd2=open(argv[2],O_WRONLY|O_CREAT|O_APPEND,0666))==-1)
    {
        printf("cannot creat file %s",argv[2]);
        return -1;
    }

    while(1)
    {
        i=read(fd1,buff,maxsize);
        write(fd2,buff,i);
        if(i!=maxsize) break;
    }

    close(fd1);
    close(fd2);
}
EOF
```

### 步骤2：创建测试用的源文件

先创建一个文本文件作为源文件（待复制文件）：

```
echo "This is a test file for copy program.
Line 2 content here.
Line 3 content here.
Line 4 content here.
Line 5 content here." > source.txt
```

按回车。

### 步骤3：编译 copy.c

```
gcc copy.c -o copy
```

按回车。没有报错说明编译成功。

### 步骤4：运行 copy 程序（第一次复制）

```
./copy source.txt dest1.txt
```

按回车。

### 步骤5：查看复制结果

```
cat dest1.txt
```

按回车。应该看到和 source.txt 一样的内容。

【截图】—— 程序运行结果

### 步骤6：回答问题——目标文件已存在会怎样？

**答案：** 会以追加模式（O_APPEND）打开，原内容不会被覆盖，新内容会追加到文件末尾。

### 步骤7：验证追加模式

往源文件再追加一行：

```
echo "Appended line." >> source.txt
```

按回车。然后再次复制：

```
./copy source.txt dest1.txt
```

按回车。再查看：

```
cat dest1.txt
```

按回车。你会看到之前的内容还在，后面又加了一份。

【截图】—— 追加后的文件内容

### 步骤8：回答问题——为什么读到的字节数不是 maxsize 就跳出循环？

**答案：** 因为文件末尾（EOF）时 read 返回的值会小于 maxsize，此时应该结束循环。如果仍按 maxsize 写入，会把垃圾数据写进去。

### 步骤9：测试没有 close(fd2) 的情况

修改程序去掉 close：

```
cat > copy_noclose.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#define maxsize 256

int main(int argc,char *argv[])
{
    int fd1,fd2;
    char buff[maxsize];
    int i;

    if(argc!=3)
    {
        printf("command error!\n");
        return -1;
    }

    fd1=open(argv[1],O_RDONLY);
    if(fd1==-1)
    {
        printf("file %s cannot open",argv[1]);
        return -1;
    }

    if((fd2=open(argv[2],O_WRONLY|O_CREAT|O_APPEND,0666))==-1)
    {
        printf("cannot creat file %s",argv[2]);
        return -1;
    }

    while(1)
    {
        i=read(fd1,buff,maxsize);
        write(fd2,buff,i);
        if(i!=maxsize) break;
    }

    close(fd1);
    /* 故意不 close(fd2) */
}
EOF
```

编译运行：

```
gcc copy_noclose.c -o copy_noclose && ./copy_noclose source.txt dest2.txt && cat dest2.txt
```

按回车。

**答案：** 程序仍然能正确运行，但会导致文件描述符泄漏。程序结束时内核会自动关闭未关闭的文件描述符，但这不是好的编程习惯。

【截图】—— 运行结果（看起来正常）

---

## 项目二：自编程序——读取文件内容并显示

### 步骤1：创建 readfile.c

```
cat > readfile.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#define maxsize 256

int main(int argc,char *argv[])
{
    int fd;
    char buff[maxsize];
    int i;

    if(argc!=2)
    {
        printf("Usage: %s <filename>\n", argv[0]);
        return -1;
    }

    fd=open(argv[1],O_RDONLY);
    if(fd==-1)
    {
        printf("cannot open file %s\n",argv[1]);
        return -1;
    }

    while((i=read(fd,buff,maxsize))>0)
    {
        write(STDOUT_FILENO,buff,i);
    }

    close(fd);
}
EOF
```

### 步骤2：编译运行

```
gcc readfile.c -o readfile
```

按回车。

```
./readfile source.txt
```

按回车。屏幕会显示 source.txt 的内容。

【截图】—— 程序代码截图（vi 或 cat 命令显示的代码）

【截图】—— 运行结果截图（屏幕上显示的文件内容）

### 步骤3：也可以读取 copy.c 本身

```
./readfile copy.c
```

按回车。会显示 copy.c 的源代码。

【截图】

---

## 项目三：调用 fork 函数创建新进程

### (一) 第一个 fork 程序

#### 步骤1：创建 fork1.c

```
cat > fork1.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
int a=5;
int main()
{
    pid_t pid;
    pid = fork();

    if(pid==0)
    {
        a=a+1;
        printf("child ,PID =%d \n",getpid());
    }
    else
    {
        a=a+2;
        printf("parent ,PID =%d \n",getpid());
    }
    printf("pid = %d ,a=%d \n", pid,a);
    return 0;
}
EOF
```

#### 步骤2：编译运行

```
gcc fork1.c -o fork1 && ./fork1
```

按回车。

**你会看到输出了4行内容**，顺序每次可能不同（父子进程竞争）。

【截图】—— 运行结果

#### 步骤3：分析——为什么输出4个 pid 值？

**答案：** fork 后父子进程都会执行后面的代码。printf("pid = %d ,a=%d \n", pid,a) 这行代码在父子进程中各执行了一次，所以一共4次输出。

#### 步骤4：分析——为什么 a 的值不同？

**答案：** 父子进程有独立的地址空间，fork 后变量互不影响。子进程中 a=6（5+1），父进程中 a=7（5+2）。

#### 步骤5：加入 wait 让子进程先执行完

```
cat > fork1_wait.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
int a=5;
int main()
{
    pid_t pid;
    pid = fork();

    if(pid==0)
    {
        a=a+1;
        printf("child ,PID =%d \n",getpid());
    }
    else
    {
        wait(NULL);
        a=a+2;
        printf("parent ,PID =%d \n",getpid());
    }
    printf("pid = %d ,a=%d \n", pid,a);
    return 0;
}
EOF
```

#### 步骤6：编译运行

```
gcc fork1_wait.c -o fork1_wait && ./fork1_wait
```

按回车。

【截图】—— 修改后的程序代码

【截图】—— 运行结果（先看到 child 输出，再看到 parent 输出）

#### 步骤7：把 wait 放在不同位置测试

把 wait 放在 if 判断之内：

```
cat > fork1_wait2.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
int a=5;
int main()
{
    pid_t pid;
    pid = fork();

    if(pid==0)
    {
        wait(NULL);
        a=a+1;
        printf("child ,PID =%d \n",getpid());
    }
    else
    {
        a=a+2;
        printf("parent ,PID =%d \n",getpid());
    }
    printf("pid = %d ,a=%d \n", pid,a);
    return 0;
}
EOF
```

```
gcc fork1_wait2.c -o fork1_wait2 && ./fork1_wait2
```

按回车。

**结果分析：** wait 放在子进程里没有意义，子进程等自己会导致死锁（子进程无法结束）。实际运行时可能只输出 child 那行就卡住。按 **Ctrl + C** 终止。

### (二) 循环3次创建子进程

#### 步骤1：创建 fork_loop.c

```
cat > fork_loop.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
int main()
{
    pid_t pid;
    int i;
    for(i=0;i<3;i++)
    {
        pid=fork();
        if(pid==0)
        {
            printf("child: i=%d, PID=%d, PPID=%d\n",i,getpid(),getppid());
        }
        else
        {
            printf("parent: i=%d, child PID=%d\n",i,pid);
        }
    }
    return 0;
}
EOF
```

#### 步骤2：编译运行

```
gcc fork_loop.c -o fork_loop && ./fork_loop
```

按回车。

【截图】—— 运行结果

#### 步骤3：分析输出数量

**如果每个进程都输出**，会有多少个输出？

**答案：** 共 15 行输出（parent 3次 + parent的3个子进程各3次 + 其中某些子进程的子进程）。准确说是 2^3 - 1 = 7 个子进程，加上父进程，共 8 个进程参与循环，每个进程都会执行 3 次 printf。

### (三) 创建两个子进程 son 和 daughter

#### 步骤1：创建 fork_son_daughter.c

```
cat > fork_son_daughter.c << 'EOF'
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
int main()
{
    pid_t pid;
    pid = fork();

    if(pid==0)
    {
        int i;
        for(i=0;i<10;i++)
        {
            printf("son: my PID=%d\n",getpid());
            sleep(1);
        }
        return 0;
    }
    else
    {
        pid_t pid2 = fork();
        if(pid2==0)
        {
            int i;
            for(i=0;i<10;i++)
            {
                printf("daughter: my PID=%d\n",getpid());
                sleep(1);
            }
            return 0;
        }
        else
        {
            int i;
            for(i=0;i<10;i++)
            {
                printf("parent: my PID=%d\n",getpid());
                sleep(1);
            }
            wait(NULL);
            wait(NULL);
        }
    }
    return 0;
}
EOF
```

#### 步骤2：编译运行

```
gcc fork_son_daughter.c -o fork_son_daughter && ./fork_son_daughter
```

按回车。会看到父子三人交替输出，每个输出10行，共30行。可以用 **Ctrl + C** 提前终止。

【截图】—— 程序代码

【截图】—— 运行结果（部分）

---

## 项目四：进程控制——system() 和 exec()

### (一) system() 函数的调用

#### 步骤1：创建 system_test.c

```
cat > system_test.c << 'EOF'
#include <stdio.h>
#include <stdlib.h>
int main()
{
    int ret;
    ret=system("ps -l");
    printf("Ok ret = %d \n", ret);
    return 0;
}
EOF
```

#### 步骤2：编译运行

```
gcc system_test.c -o system_test && ./system_test
```

按回车。

你会看到 `ps -l` 的输出，然后看到 `Ok ret = 0`。

【截图】—— 运行结果

### (二) exec() 函数族的调用

#### 步骤1：创建 exec_test.c

```
cat > exec_test.c << 'EOF'
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
    int ret;
    ret = execlp("ps", "ps", "-l", NULL);
    printf("ps Ok %d",ret);
    return 0;
}
EOF
```

#### 步骤2：编译运行

```
gcc exec_test.c -o exec_test && ./exec_test
```

按回车。

你会看到 `ps -l` 的输出，然后程序就结束了（没有输出 `ps Ok`）。

【截图】—— 运行结果

#### 步骤3：分析 system 和 exec 的区别

**答案：** system() 会创建子进程执行命令，执行完后返回父进程继续执行后面的代码（所以会输出 `Ok ret = 0`）。exec() 则直接用新程序替换当前进程，执行完后整个程序就结束了，不会执行后面的 printf（所以看不到 `ps Ok`）。

#### 步骤4：修改 exec 程序，让它输出 "Ok"

```
cat > exec_test2.c << 'EOF'
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
    int ret;
    ret = execlp("ps", "ps", "-l", NULL);
    printf("ps Ok %d",ret);
    return 0;
}
EOF
```

实际上 exec 之后后面的代码就不会执行了（因为进程被替换了）。如果要让 `ps Ok` 输出，需要在 exec 前打印，或者用 fork 创建子进程，在子进程里 exec：

```
cat > exec_test_fork.c << 'EOF'
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
int main()
{
    pid_t pid = fork();
    if(pid==0)
    {
        int ret = execlp("ps", "ps", "-l", NULL);
        printf("ps Ok %d",ret);
        return 0;
    }
    else
    {
        wait(NULL);
        printf("Ok\n");
    }
    return 0;
}
EOF
```

编译运行：

```
gcc exec_test_fork.c -o exec_test_fork && ./exec_test_fork
```

按回车。

【截图】—— 修改后的代码

【截图】—— 运行结果（能看到 "Ok" 输出了）

### (三) fork + exec 综合实验

#### 步骤1：创建 fork_exec.c

```
cat > fork_exec.c << 'EOF'
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>

int main()
{
    pid_t pid = fork();

    if(pid==0)
    {
        execlp("echo", "echo", "new program.", NULL);
        return 0;
    }
    else
    {
        wait(NULL);
        printf("child pid = %d finished\n", pid);
    }

    return 0;
}
EOF
```

#### 步骤2：编译运行

```
gcc fork_exec.c -o fork_exec && ./fork_exec
```

按回车。会看到 `new program.` 然后看到 `child pid = xxx finished`。

【截图】—— 运行结果

---

## 结束

所有实验完成。用以下命令确认：

```
ls ~/lab3
```

按回车，你会看到所有创建的文件。
