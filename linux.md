### linux

#### 基本命令

```shell
# 用户权限设置
sudo -i #切换到root权限
exit #回到用户权限

# 文件操作
rm -f filename #删除文件 (rm: remove)
rm -rf filepath #删除目录及其以下的所有文件/文件夹 (-r:recursive 向下递归)


#lsof: list open file 查看打开进程的文件，打开文件的进程，进程打开的端口，
lsof -i|grep rssp
lsof -u ^root #-u:user 列出某个用户打开的文件信息
lsof -i tcp #列出tcp/udp的连接情况
lsof -p 8080 #通过进程号显示该进程打开的文件

```

- 目录操作：mkdir rmdir cd
- 文件操作：ls echo cat rm cp mv
- 文件内容操作：grep sed awk
- 网络命令：ipconfig ps netstat

打开系统监视器： gnome-system-monitor



##### ps：查看进程命令

 ```shell
#ps命令（Process Status）查看进程命令
ps a #显示现行终端机下的所有程序，包括其他用户的程序。
ps -aux,然后再利用一个管道符号导向到grep去查找特定的进程,然后再对特定的进程进行操作

ps -aux |more #可以用 | 管道和 more 连接起来分页查看。

#把所有进程显示出来，并输出到ps001.txt文件，然后再通过more 来分页查看
ps -aux > ps001.txt
more ps001.txt

#查看进程id
ps -ef | grep nginx
#ps -ef #表示显示所有进程的消息
#command1 | command2 # |:管道命令 将command1执行的结果交给command2处理
#grep nginx #在所有进程的消息查询名字为nginx的进程

#根据pid查看占用端口
lsof -i | grep pid
netstat -nap | grep pid

#根据端口号查看对应的进程名
lsof -i : port
netstat -nap | grep port

#grep:Linux下的文本过滤工具
grep -i abc test.txt #查找test.txt文件中的“abc”字符串，不区分大小写


 ```



#### 进程通信

##### 共享内存

> [mmap映射区和shm共享内存的区别总结](https://blog.csdn.net/hj605635529/article/details/73163513?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)

两个进程的一块虚拟地址空间映射到了同一块物理内存上

- Linux 实现共享内存的方式

1)mmap内存共享映射, 2) System V 3) POSIX

（SystemV和POSIX底层都是基于内存文件系统**tmpfs**实现，主要在接口设计上有差别，POSIX遵循了Linux系统一切皆文件的理念）

- 共享内存相关文件

 /proc/sys/kernel/shmmni：限制整个系统可创建共享内存段个数。 

 /proc/sys/kernel/shmall： 限制系统用在共享内存上的内存的页数。  

 /proc/sys/kernel/shmmax：限制一个共享内存段的最大长度，字节为单位。

- System V 共享内存

> https://blog.csdn.net/RayCongLiang/article/details/100027643

```c
#include <sys/ipc.h>
#include <sys/shm.h>
/* 0.ftok  系统建立IPC通讯（如消息队列、共享内存时）必须指定一个ID值。通常情况下，该id值通过ftok函数得到
参数：
pathname：一个指定的文件路径
proj_id：对应项目的id
通过返回的key_t类型让所有的进程都唯一映射到对应内存空间，失败返回-1
*/
key_t ftok (const char *pathname, int proj_id);

/*1.shmget 创建共享内存
参数：
key:段名，可以是自定义的整数，或者用字符串通过ftok() 生成（但每次生成的值可能不一样）
size:大小
shmflg:9个权限标志（与创建文件使用的mode模式标志一样）。
成功返回该内存段的标志码（非负整数），否则返回-1 
*/
int shmget(key_t key, int size, int shmflg);  

/*2.shmat 将共享内存段连接到进程地址空间
参数：
shmid:共享内存标识码
shmaddr:指定连接的地址(shmflg=SHM_RND时才可指定)，多数情况设为空指针，由系统自动选择地址
shmflg:SHM_RND 或 SHM_RDONLY（映射过来的地址只读）
成功返回指向映射地址的第一个字节的指针，失败返回-1
*/
void *shmat(int shmid, void *shmaddr, int shmflg);

/*3.shmdt 解除映射，将当前进程与共享内存段脱离
成功返回0，否则-1
*/
int shmdt(const void *shmaddr);

/*4.shmctl 控制共享内存
参数：
shmid:共享内存标志码
cmd:IPC_STAT:得到共享内存的状态，把共享内存的shmid_ds结构体复制到buf里
	IPC_SET：改变共享内存的状态，把buf所指的结构体中的uid,gid,mode,复制到共享内存的shmid_ds结构体内
	IPC_RMID:删除共享内存段
buf:指向一个保存共享内存的模式状态和访问权限的数据结构
成功返回0，否则-1
*/
int shmctl(int shmid, int cmd, struct shmid_ds *buf)
```

- 查看内存信息

```shell
~$ ipcs #显示进程通信消息（共享内存、消息队列、信号量）
~$ free #显示系统内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。
```

![image-20200819174527606](linux.assets/image-20200819174527606.png)





mmap（内存映射函数）

普通文件的读写

进程调用**read**或**write**<系统调用>后会陷入内核，进入系统调用，内核开始读写文件。对于读文件，内核首先把数据从磁盘copy到内核缓冲区（页缓冲）中，然后进程从内核态回到用户态，内核把读入内核内存的数据copy到进程的用户态内存空间。（实际上对同一份文件内容进行2次拷贝，磁盘->内核空间->用户空间）

把内核中特定部分的内存空间映射到用户级程序的内存空间（用户空间和内核空间共享一块相同的内存）

mmap, 它把文件内容映射到进程地址空间后，进程可以像访问内存的方式对文件进行访问，不需要其他系统调用(read,write)去操作。

> [mmap](https://blog.csdn.net/Holy_666/article/details/86532671)

1. 进程在用户空间调用库函数mmap，进程启动映射过程，并在虚拟地址空间中为映射创建虚拟映射区域
2. 调用内核空间的mmap函数，实现文件物理地址和进程虚拟地址的一一映射关系
3. 进程发起对该映射空间的访问，引发缺页异常，实现文件内容到物理内存的拷贝**mmap只是在虚拟内存分配了地址空间，只有在第一次访问虚拟内存的时候才分配物理内存。**

创建新的虚拟内存区域和建立文件磁盘地址和虚拟内存区域映射这两步，没有任何文件拷贝操作。而之后访问数据时发现内存中并无数据而发起的缺页异常过程，可以通过已经建立好的映射关系，只使用一次数据拷贝，就从磁盘中将数据传入内存的用户空间中，供进程使用。

#### GCC 命令

```shell
gcc #不加参数可以直接编译成可执行文件
gcc main.c #默认生成a.out可执行文件
gcc main.c -o main.exe #可以通过-o更改生成文件的名字

gcc -o <file> #(out)将输出放入<文件>
gcc -c main.c #(compile)编译和汇编，但不链接，生成main.o二进制目标文件
gcc -g main.c -o main#添加gdb调试选项

gcc main.o #自动链接上一步生成的main.o来生成最终可执行文件a.out

gcc main.c -o main
./main #运行可执行文件main

```



#### Core dump

> [详解coredump](https://blog.csdn.net/MOU_IT/article/details/88903668)

##### core dump配置

产生core dump核心转储文件

```shell
~$ ulimit -c #若显示0，表明系统的core文件生成为开启
~$ gedit .bashrc # 修改当前用户的.bashrc或.bash_profile
#在文件末尾添加: ulimit -c unlimited 
~$ source .bashrc
~$ ulimit -c #ulimit生成的core文件大小不受限制
```



##### Core dump+gdb调试

```shell
#gcc编译成可执行文件后，运行该文件
~$ gcc -g -o main main.c #一般会加-g用来添加调试信息
~$./main #出现错误，相应文件夹下会产生core文件
~$ gdb main core #进入gdb调试，可以看到出错的代码位置及原因
(gdb) bt #进入gdb后，输入bt查看进程结束的地方
```

<img src="linux.assets/image-20200818155709092.png" alt="image-20200818155709092" style="zoom:80%;" />



#### GDB调试

[GDB 调试指南](https://www.yanbinghu.com/2019/04/20/41283.html)

```shell
#在编译时增加-g参数，保留调试信息
gcc -g testGDB.c -o testGDB
g++ -g testGDB.cpp -o testGDB
g++ -g -std=c++11 testGDB testGDB.cpp 

#==== 调试未运行的程序 ====
#调试启动无参程序
gdb testGDB 

#调试启动带参程序
gdb testGDB
#方法1：run后面跟参数
run arg_hello arg_world 
#方法2：set args后面跟参数，再run
set args arg_hello arg_world
run

#==== 调试已运行的程序 ====
ps -ef|grep 进程名
```



> [Linux下GDB调试指令](https://zhuanlan.zhihu.com/p/71519244)

```shell
#运行指令
r/run：运行程序到断点
c/continue：继续执行到下一个断点
n/next：单步跳过，如果遇到函数调用，不进入函数
s/step：单步进入，如果遇到函数调用，则进入函数
finish：运行程序，知道当前函数完成返回，并打印函数返回时堆栈的地址、返回值、参数值等信息
q/quit：退出gdb
attch：用gdb调试一个正在运行的进程
#设置断点
b/break n：在第n行设置断点
b/break func：在func()函数入口处设置断点
b fn1 if a>b：条件断点
clear n：删除第n行的断点
info b/breakpoints：显示当前程序的所有断点
delete breakpoints：清除当前程序的所有断点
#查看源码
l/list
#打印表达式
p/print 表达式：显示程序中任何有效的表达式
p ptr：若ptr为指针，则打印指针地址
p *ptr：通过解引用打印指针指向的内容
p *ptr@10：打印指针所指数组的多个值
watch 表达式：设置一个监视点
info locals：显示当前堆栈页的所有变量
#查询运行信息
where/bt：当前运行的堆栈列表
bt backtrace：显示当前调用堆栈
info program：查看程序是否在运行，进程号，被暂停的原因等
```







>https://www.cnblogs.com/caoer/p/12669183.html#autoid-h3-4-2-0

**VMware运行Ubuntu无法跨系统复制粘贴**

重新安装VMwareTools

1. 打开虚拟机菜单栏中 选择 虚拟机-->重新安装VMware Tools 即可打开VMware Tools窗口
2. 复制VMwareTools-10.1.6-5214329.tar.gz压缩包到下载
3. 在文件夹下打开终端输入：tar zxvf VMwareTools-10.1.6-5214329.tar.gz，回车，即可解压tar.gz文件，可以看到解压的文件夹
4. 终端输入cd vmware-tools-distrib,即可进入文件夹vmware-tools-distrib目录下，再输入命令sudo ./vmware-install.pl,回车，需要输入用户密码，回车，即开始安装VMware Tools工具，下面一直选择yes或者回车即可。

**yum命令“无法启用仓库”**

`yum -y install xxx`

Ubuntu下不支持yum命令，应替换为`apt-get install xxx`

最新版git





### git

#### 1. 创建初版

```bash
#在要git的文件夹下打开git Bash
git init
git add .
git commit -m "===message===="
#要在github上创建相应的repository
git remote add origin https://github.com/yrrSelena/MedInfoSearch.git
git push origin master
```

#### 2. 修改版

```bash
git add .
git commit -m "===message===="
git push origin master
```

#### 3. 获取远程主机的最新版

```bash
git fetch origin master #fetch：将远程主机的最新内容拉到本地，不进行合并
git log -p master..origin/master #比较本地的master分支和origin/master分支的差别
git merge origin/master #合并内容到本地

#pull：将远程主机的master分支最新内容拉下来后与当前本地分支直接合并 fetch+merge
git pull origin master
```

#### 4. 分支管理

```shell
git branch #显示当前所有分支，‘*’指向当前分支
git branch -v #查看看每个分支最后一次提交的版本

#创建分支
git branch branch_name #创建分支
git checkout branch_name #切换到branch_name分支
git checkout -b branch_name #创建并切换到branch_name分支

#合并分支
git chechout master #先切换到master分支
git merge branch_name #再合并分支
#合并冲突
get branch --no-merged #查看未合并的分支

#删除分支
git branch -d branch_name

#远程分支
git push origin linux
```

#### 5. 远程仓库

```shell
git remote -v #查看远程仓库详细信息，可以看到仓库名称
git remote rm origin #删除origin仓库
git remote add origin https://github.com/yrrSelena/MedInfoSearch.git #重新添加远程仓库地址
git push -u origin master #提交到远程仓库的master主干
```



#### 常见问题

**Git 提示fatal: remote origin already exists** 

删除远程 Git 仓库

`git remote rm origin`