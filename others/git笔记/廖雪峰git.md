### git对象

每个对象(object) 包括三个部分：**类型**，**大小**和**内容**

- commit：指向一个"tree对象", 并且带有相关的描述信息

  `git show -s --pretty=raw commit对象id`

  `git log --pretty=raw commit对象id`

- tree：有一串(bunch)指向blob对象或是其它tree对象的指针，它一般用来表示内容之间的目录层次关系

  `git show tree对象id`

  `git ls-tree tree对象id`

- blob：一个"blob对象"就是一块二进制数据，它没有指向任何东西或有任何其它属性，甚至连文件名都没有

  `git show blob对象id`

- tag：一个标签对象包括一个对象名(译者注:就是SHA1签名), 对象类型, 标签名, 标签创建人的名字("tagger"), 还有一条可能包含有签名(signature)的消息

  `git cat-file tag 标签名`

### 创建版本库

```python
mkdir learngit
cd learngit
git init
```

### 添加文件

```python
git add 文件名
git commit -m ""
git commit -a：自动把所有内容被修改的文件(不包括新创建的文件)都添加到索引中，并且同时把它们提交
```

### 查看工作区状态

```python
git status
git diff 文件名
```

### 查看提交日志

```python
git log
git log --pretty=oneline
git log --oneline

git log -p：日志中包含补丁(patchs)
git log --stat：显示在每个提交(commit)中哪些文件被修改了, 这些文件分别添加或删除了多少行内容

git log --graph
```

### 版本回退

```python
# git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向 1 改为指向 2，然后顺便把工作区的文件更新了
git reset --hard HEAD  # 当前版本
git reset --hard HEAD^^  # 当前版本的父父版本
git reset --hard HEAD~5
```

### 后悔药

```python
# 回退到旧版本，并关闭终端，找不到新版的commit_id
git reflog  # 查看命令历史，找到新版commit_id
git reset --hard 新版commit_id
```

### 工作区和暂存区

```python
工作区（Working Directory）：在电脑里能看到的目录，比如我的learngit文件夹就是一个工作区
版本库（Repository）：工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库

版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD
```

![image-20200510222600672](/Users/yanglu/Library/Application Support/typora-user-images/image-20200510222600672.png)

### 查看工作区和版本库最新版的区别

```python
git diff HEAD --文件名
```

### 查看差异

- 工作区与暂存区：`git diff`

- 暂存区与版本库：`git diff --cached`

- 工作区与版本库：`git diff HEAD`

- 工作区与另一个分支：`git diff 分支名`

- 两个分支之间：`git diff 分支1..分支2`，如`git diff master..test`

- 找出‘master’,‘test’的共有 父分支和'test'分支之间的差异：`git diff 分支1...分支2`，

  如`git diff master...test`

### 区间

```python
# 可以用".."来指两个提交(commit)之间的区间
7b593b5..51bea1  # 给出你在"7b593b5" 和"51bea1"之间除了"7b593b5外"的所有提交(commit)(注意:51bea1是最近的提交).
7b593b..  # 从 7b593b开始的提交(commit). 相当于 7b593b..HEAD
```

### 撤销工作区的修改

```python
git checkout --文件名
git checkout其实是用版本库里的版本替换工作区的版本
# 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
# 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
注意：git checkout 分支名。是切换分支
```

### 撤销缓存区的修改

```python
git reset HEAD 文件名  # 把暂存区的修改撤销掉（unstage），重新放回工作区
此时，工作区有修改
要撤销工作区的修改：git checkout --文件名
```

### 撤销修改小结

```python
撤销工作区的修改：git checkout --文件名
撤销暂存区的修改：git reset HEAD 文件名  然后git checkout --文件名
撤销本地版本库的修改：git reset --hard HEAD^（版本回退）
撤销远程库的修改：暂未办法
```

### 删除文件

```python
rm 文件名
git add/rm 文件名
git commit -m''
```

### 恢复工作区文件

```python
# 从版本库中恢复
git checkout --文件名
```

### 配置远程仓库

```python
# 本地生成ssh密钥
ssh-keygen -t rsa -C "youremail@example.com"  # 在主目录下生成.ssh目录，里面有id_rsa和id_rsa.pub两个文件
# 登陆GitHub，打开SSH Keys页面，添加id_rsa.pub的内容
GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送
```

### 本地关联远程仓库

```python
# 登录github，新建仓库
# 关联远程库
git remote add origin https://github.com/louisyanglu/git_learning.git  # 远程库的名字叫origin，可更改
# 第一次推送，将master分支关联起来
git push -u origin master
# 后续推送
git push origin master
```

### 从远程库克隆

```python
git clone https://github.com/louisyanglu/gitskills.git
# Git支持多种协议，包括https，但ssh协议速度最快。
```

### 创建、合并分支

- 一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点

  ![image-20200511103233771](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511103233771.png)

- 创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上

  ![image-20200511103326513](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511103326513.png)

  ```python
  # 创建分支
  git branch dev
  # 切换分支
  git checkout dev  or  git switch dev
  
  # 创建、并切换分支
  git checkout -b dev  or git switch -c dev
  ```

  从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变

  ![image-20200511103417174](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511103417174.png)

  ```python
  # 查看当前分支
  git branch
  ```

- 假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。直接把`master`指向`dev`的当前提交，就完成了合并：

  ![image-20200511121813443](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511121813443.png)

  ```python
  # 切换分支
  git checkout master  or  git switch master
  # 合并分支
  git merge dev  # git merge命令用于合并指定分支到当前分支
  # 后悔了，撤销合并
  git reset --hard HEAD
  ```

- 合并完分支后，可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

  ![image-20200511103534647](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511103534647.png)

  ```python
  # 删除分支
  git branch -d dev
  ```

### 解决冲突

​	当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

​	解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

### 不使用fast forward模式的merge

![image-20200511121709575](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511121709575.png)

```python
# 合并分支使用fast forward模式，删除分支后，分支信息不可追溯
# 合并分支使用--no-ff禁用fast forward模式，合并时会生成一个commit，方便追溯分支信息
git merge --no-ff -m "merge with no-ff" dev
```

### 分支的策略

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

![image-20200511122349377](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511122349377.png)

### 临时任务、bug分支

- 把当前工作现场“储藏”起来，等以后恢复现场后继续工作

  `git stash`

- 切换分支，创建修复bug的分支

  `git checkout master`

  `git checkout -b issue-101`

- 修复bug，提交

- bug分支合并到master

  `git checkout master`

  `git merge --no-ff -m'bug fixed' issue-101`

- 回到工作分支，继续干活

  `git checkout dev`

  `git stash list`

  恢复工作现场有两种方法：

  `git stash apply`：恢复后，stash内容并不删除，你需要用`git stash drop`来删除

  `git stash pop`：恢复的同时把stash内容也删了

  指定stash内容：`git stash apply stash@{0}`

- 工作分支的bug也要修复

  使用`cherry-pick`命令，让我们能复制一个特定的提交到当前分支，避免重复劳动

  `git cherry-pick bug修复的commit对象`

### 新功能feature分支

每添加一个新功能，最好在dev分支上新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

如果要丢弃一个**没有被合并过**的分支，可以通过`git branch -D 分支名 `强行删除

### 多人协作模式

```python
# 从远程仓库克隆时，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且，远程仓库的默认名称是origin
# 查看远程库信息
git remote -v  # 上面显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址
# 推送本地分支
git push origin 本地分支名
# 从远程库clone时，只能看到master分支，开发需创建远程origin的dev分支到本地，在dev分支上进行开发
git checkout -b dev origin/dev
# 若两人同时对dev下统一文件进行修改，会产生冲突
# 先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送
git branch --set-upstream-to=origin/dev dev  # 指定本地dev分支与远程origin/dev分支的链接
git pull
手动解决冲突后，commit提交
git push origin dev
```

### 变基

![image-20200511134223974](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511134223974.png)

![image-20200511134336303](/Users/yanglu/Library/Application Support/typora-user-images/image-20200511134336303.png)

​		`git checkout mywork`

​		`git rebase origin`

​	会把你的"mywork"分支里的每个提交(commit)取消掉，并且把它们临时 保存为补丁(patch)(这些补丁放到".git/rebase"目录中),然后把"mywork"分支更新 到最新的"origin"分支，最后把保存的这些补丁应用到"mywork"分支上。

### 标签

tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起

- 创建标签

  `git tag 标签名`

  `git tag 标签名 commit_id`

  `git tag -a 标签名 -m "标签说明" commit_id`

- 查看标签

  `git tag`

- 查看标签信息

  `git show 标签名`

- 删除本地标签

  `git tag -d 标签名`

- 删除远程标签

  先删除本地：`git tag -d 标签名`

  再删除远程：`git push origin :refs/tags/标签名`

- 推送标签到远程

  `git push origin 标签名`

- 推送所有标签到远程

  `git push origin --tags`

### 码云

- 登录、上传SSH公钥

- 已有本地库，关联远程库，`git remote add`

  如果本地库关联了别的远程库，查看remote信息：`git remote -v`

  删除已有的远程库，`git remote rm origin`

  再关联码云远程库：`git remote add origin git@gitee.com:liaoxuefeng/learngit.git`

- 一个本地库，关联多个远程库

  `git remote add 远程库名1 ...`，推送使用：`git push 远程库名1 master`

  `git remote add 远程库名2 ...`，推送使用：`git push 远程库名2 master`

### git config

- 界面显示颜色

  `git config --global color.ui true`

- 忽略文件

  ```python
  # 在git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去
  # 文件名参考：https://github.com/github/gitignore
  
  提交.gitignore文件，对.gitignore做版本管理
  ```

  如果某个文件被忽略，但确实需要添加，使用强制添加

  `git add -f 文件名`

- 配置别名

  ```python
  git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
  ```

  仓库（--local）的配置文件在`./git/config`文件中

  用户（--global）的配置文件在用户主目录下的`.gitconfig`

  