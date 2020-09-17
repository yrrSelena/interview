[TOC]



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

mac系统安装最新版git

```shell
brew install git #安装最新版git
which git #查看git指向
git --version #显示版本信息

#如果一直显示旧的版本，需要进行路径配置
echo $PATH #查看路径
vim ~/.bash_profile #在用户级编辑环境变量
export PATH=/usr/local/Cellar/git/2.28.0/bin:$PATH  #添加环境变量
source ~/.bash_profile #执行source使环境变量生效

git --version #显示最新版本
```





Q: Git 提示fatal: remote origin already exists

删除远程 Git 仓库

`git remote rm origin`



Q: Git 冲突 Your local changes to the following files would be overwritten by merge:

方法一、合并，stash 隐藏

```shell
git stash #备份当前工作区的内容到git栈中
git pull #获取远程仓库的内容
git stash pop #从git栈读取最近一次保存的内容，恢复工作区的相关内容
git diff #查看代码自动合并情况
git diff -w filename #查看文件filename的合并情况
```

方法二、用远程仓库的文件完全覆盖本地版本

```shell
git fetch --all
git reset --hard
git pull
```

方法三、直接commit自己的代码