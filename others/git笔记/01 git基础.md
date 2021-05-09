### 1、配置user、email

```python
git config --global user.name 'yanglu'
git config --global user.email 'louisyanglu@163.com'

参数区别:
git config --local  # 只对某个仓库有效,切换到另外一个仓库失效
git config --global  # 当前用户的所有仓库有效,工作当中最常用
git config --system  # 系统的所有用户,几乎不用

查看配置:

git config --list --local  # 只能在仓库里面起作用, 普通路径git不管理
git config --list --global
git config --list --system

local的在.git/config里面；global的在个人home目录下的.gitconfig里面；system应该在git安装目录下
```

### 2、创建local仓库

```python
1、已有项目纳入git管理
cd 项目文件夹
git init

2、没有项目，新建
cd 文件夹
git init 项目名  # 会新建项目文件夹
cd 项目名
git add 文件名
git status
git commit -m'...'
git log
```

### 3、工作目录的**多个文件**放在暂存区

```python
git add -u：将文件的修改、文件的删除，添加到暂存区。
git add .：将文件的修改，文件的删除，文件的新建，添加到暂存区。只能够提交当前目录或者它子目录下相应文件
git add -A：将文件的修改，文件的删除，文件的新建，添加到暂存区。可提交整个仓库文件。git add --all
```

### 4、重命名文件

```python
mv readme readme.md
git add readme.md
git rm readme

git mv readme readme.md
```

### 5、git log参数

```python
git log --all 查看所有分支的历史
git log --all --graph 查看图形化的 log 地址
git log --oneline 查看单行的简洁历史。
git log --oneline -n4 查看最近的四条简洁历史。
git log --oneline --all -n4 --graph 查看所有分支最近 4 条单行的图形化历史。
git help --web log 跳转到git log 的帮助文档网页

git branch -v  ——> 查看有几个分支
git checkout -b 分支名 4ec3ec22fd7 ——> 根据以前的版本，建立分支，并跳转
git checkout 分支名	——> 切换分支
```

### 6、.git文件夹

```python
# HEAD文件夹
cat HEAD 查看HEAD文件的内容
HEAD里面存放的内容ref：refs/heads/指示目前正在使用的分支
最终落脚于commit对象

# config文件
config：存放本地仓库（local）相关的配置信息。

# refs文件夹
refs/heads文件夹:存放分支
refs/tags文件夹:存放tag，又叫里程牌 （当这次commit是具有里程碑意义的 比如项目1.0的时候 就可以打tag）

git cat-file 命令 显示版本库对象的内容、类型及大小信息。
git cat-file -t b44dd71d62a5a8ed3 显示版本库对象的类型
git cat-file -p b44dd71d62a5a8ed3 显示版本库对象的内容
git cat-file -s b44dd71d62a5a8ed3 显示版本库对象的大小
# objects文件夹
objects：存放对象 .git/objects/ 文件夹中的子文件夹都是以哈希值的前两位字符命名 每个object由40位字符组成，前两位字符用来当文件夹，后38位做文件。
```

### 7、commit、tree、blob三个对象之间的关系

当我们添加或者修改了文件并且add到Stage Area之后，首先会根据文件内容创建不同的blob，当进行提交之后马上创建一个tree组件把需要的blob组件添加进去，之后再封装到一个commit组件中完成本次提交。在将来进行reset的时候可以直接使用`git reset --hard xxxxx`可以恢复到某个特定的版本，在reset之后，git会根据这个commit组件的id快速的找到tree组件，然后根据tree找到blob组件，之后对仓库进行还原，整个过程都是以hash和二进制进行操作，所以git执行效率非常之高。

![image-20200509212757901](/Users/yanglu/Library/Application Support/typora-user-images/image-20200509212757901.png)

### 8、分离头指针

git checkout commit对象名  导致  工作在没有branch的环境下

如果切换到别的分支上，做的更改会无效。

git branch 新建分支名 commit对象名  ——> 新建一个分支，使更改与branch绑在一起

### 9、HEAD和branch

git checkout -b 新的分支 基于的分支

git diff HEAD HEAD^

一个节点，可以包含多个子节点（checkout 出多个分支）
2 一个节点可以有多个父节点（多个分支合并）
3 ^是~都是父节点，区别是跟随数字时候，^2 是第二个父节点，而~2是父节点的父节点
4 ^和~可以组合使用,例如 HEAD~2^2

"^"这个操作符代表父commit。
当一个commit有多个父commit时，可以通过在符号“^”后面跟上一个数字来表示第几个父commit。
比如，"A^" 等于 “A^1”(表示A这个commit的第1个父commit)。

连续的“^”符号依次沿着父commit进行定位，直到某个祖先commit。

~<n> 相当于连续n个符合“^”。

所以，HEAD^^ 等同于 HEAD~2 是对的

