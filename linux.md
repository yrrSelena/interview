### linux

#### 基本命令

```shell
# 用户权限设置
sudo -i #切换到root权限
exit #回到用户权限

# 文件操作
rm -f filename #删除文件 (rm: remove)
rm -rf filepath #删除目录及其以下的所有文件/文件夹 (-r:recursive 向下递归)
```



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



> https://blog.csdn.net/MOU_IT/article/details/88903668

产生core dump核心转储文件

```shell
~$ ulimit -c #若显示0，表明系统的core文件生成为开启
~$ gedit .bashrc # 修改当前用户的.bashrc或.bash_profile
#在文件末尾添加: ulimit -c unlimited 
~$ source .bashrc
~$ ulimit -c #ulimit生成的core文件大小不受限制
```



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

1. 初版

```bash
#在要git的文件夹下打开git Bash
git init
git add .
git commit -m "===message===="
#要在github上创建相应的repository
git remote add origin https://github.com/yrrSelena/MedInfoSearch.git
git push origin master
```

2. 修改版

```bash
git add .
git commit -m "===message===="
git push origin master
```

3. 获取远程主机的最新版

```bash
#fetch：将远程主机的最新内容拉到本地，不进行合并
git fetch origin master

#pull：将远程主机的master分支最新内容拉下来后与当前本地分支直接合并 fetch+merge
git pull origin master
```





#### 常见问题

**Git 提示fatal: remote origin already exists** 

删除远程 Git 仓库

`git remote rm origin`