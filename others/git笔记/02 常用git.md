### 删除不需要的分支

`git branch -d 分支名`

`git branch -D 分支名`

使用-d 在删除前Git会判断在该分支上开发的功能是否被merge到其它分支。如果没有，不能删除。

如果merge到其它分支，但之后又在其上做了开发，使用-d还是不能删除。-D会强制删除

### 修改上一次的提交

`git commit --amend` 修改上一次提交，不只是修改message。
比如上一次提交时有几个文件没有add以及commit，可以重新进行add之后再commit --amend提交。
但这次提交之后不会增加一次新的commit，而是相当于在上一次commit的基础上进行修改。

### rebase变基修改老旧的commit message

git rebase工作的过程中，就是用了分离头指针。rebase意味着基于新base的commit来变更部分commits。它处理的时候，把HEAD指向base的commit，此时如果该commit没有对应branch，就处于分离头指针的状态。git接下来会在我们修改message那个commit往后依次对后面的commit修改hash值但不更改commit的文件内容,让后续的commit的hash值都是基于修改mseeage之前的那个commit依次更新！当rebase创建完最后一个commit后，结束分离头状态，Git让变完基的分支名指向HEAD。

注意git rebase -i 要变更的节点的父节点名

如果要更改的是第一次提交，git rebase -i --root

### 合并连续的commit为1个

git rebase -i 开始commit [结束commit], 在执行这个命令时，
如果没有指定 结束commit,那么结束commit 默认为当前分支最新的 commit，那么rebase 结束后会自动更新当前分支指向的 commit,
如果指定了结束 commit，而且结束 commit不是当前分支最新的 commit，那么rebase 后会有生成一个 游离的 head,，而且当前分支指向的commit 不会更新

git rebase -i 要变更的几个commit的父节点

pick一个子节点，其余节点选择模式，如s

撤销此次合并：git reset --hard HEAD^

### 合并间隔的commit为1个

git rebase -i --root

将要合并的commit放在一起

如果出现冲突，1、解决冲突，2、 git add，3、git rebase --continue

### 比较暂存区与HEAD（指向最近一次commit）的区别

`git diff --cached [--具体的多个文件名]`

`git diff --staged [--具体的文件名]`

比较工作区和暂存区的区别

`git diff [--具体的文件名]`

比较工作区和HEAD的区别

`git diff HEAD [--具体的文件名]`

### 恢复暂存区到HEAD状态

变更暂存区用reset

`git reset HEAD [--恢复的文件]`

git reset 有三个参数
--soft 这个只是把 HEAD 指向的 commit 恢复到你指定的 commit，暂存区 工作区不变
--hard 这个是 把 HEAD， 暂存区， 工作区 都修改为 你指定的 commit 的时候的文件状态
--mixed 这个是不加时候的默认参数，把 HEAD，暂存区 修改为 你指定的 commit 的时候的文件状态，工作区保持不变

### 恢复工作区到暂存区

变更工作区用checkout

`git checkout --恢复的文件`

### 消除最近的几次提交

git reset --hard commit对象

使用--hard会删除工作区的内容，慎用

git rebase -i commit对象  ——> 设置为d

### 比较两个分支最近一次提交的差异

git diff 分支1 分支2 --文件名

git diff commit1 commit2 --文件名

### 删除文件

git rm 文件名

### 临时任务

git stash list

git stash 把当前工作区的内容放入暂存区
git stash pop 把暂存区的内容恢复到工作区，且删除
git stash apply把暂存区的内容恢复到工作区，且保留

git stash pop stash@{序号}

### 制定不需要git管理的文件

把想忽略的文件添加到 .gitignore文件

然后通过 git rm -- cached name_of_file 的方式删除掉git仓库里面无需跟踪的文件。

### git备份

![image-20200510175826082](/Users/yanglu/Library/Application Support/typora-user-images/image-20200510175826082.png)

智能协议传输进度可见，且速度快