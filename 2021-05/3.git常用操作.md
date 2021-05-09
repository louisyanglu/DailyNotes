## 集中式 vs 分布式

### 集中式

1.  集中式版本控制系统，版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。
2.  集中式版本控制系统最大的毛病就是必须联网才能工作，遇到网速慢的话，可能提交一个10M的文件就需要5分钟
3.  集中式版本控制系统的中央服务器要是出了问题，所有人都没法干活了

### 分布式

1.  分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上
2.  和集中式版本控制系统相比，分布式版本控制系统的安全性要高很多，因为每个人电脑里都有完整的版本库，某一个人的电脑坏掉了不要紧，随便从其他人那里复制一个就可以了

### 版本控制系统

1.  只能跟踪文本文件的改动，比如TXT文件，网页，所有的程序代码等等
2.  没法跟踪图片、视频这些二进制文件的变化
3.  Microsoft的Word格式是二进制格式，因此，版本控制系统是没法跟踪Word文件的改动的

------

## 本地仓库

### 创建

1.  cd到目标目录，使用`git init`
2.  当前目录下多了一个`.git`的目录，这个目录是Git来跟踪管理版本库的

### 提交

1.  `git add`
2.  `git commit -m ""`

### 对比

1.  `git status`
2.  `git diff`

### 版本回退

1.  `git log` / `git log --pretty=oneline`
2.  在Git中，用`HEAD`表示当前版本。上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`
3.  `git reset --hard HEAD^`
4.  最新的那个版本已经看不到了！好比你从21世纪坐时光穿梭机来到了19世纪，想再回去已经回不去了，肿么办？
    -   只要上面的命令行窗口还没有被关掉，你就可以顺着往上找啊找啊，找到那个最新版本的`commit id`
        -   `git reset --hard 031d`
    -   找不到``commit id`了，Git提供了一个命令`git reflog`用来记录你的每一次命令
        -   `git reflog`

### 撤销修改

1.  工作区文件修改了，但没add到stage缓存区，回到版本库状态：`git restore 文件`
2.  工作区文件修改且add到stage缓存区，想回到版本库状态：
    1.  恢复缓存区：`git restore --staged 文件`
    2.  恢复工作区：`git restore 文件`

### 删除文件

1.  工作区文件误删想恢复，`git restore 文件`
2.  工作区和版本库都想删
    -   `git rm test.txt`
    -   `git commit -m XXX`

------

## 工作区和版本库

### 工作区

1.  电脑里能看到的目录

### 版本库

1.  工作区有一个隐藏目录`.git`就是其版本库

2.  版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`

    ![git-repo](https://www.liaoxuefeng.com/files/attachments/919020037470528/0)

3.  第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

    第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

4.  第一次修改 -> `git add` -> 第二次修改 -> `git commit`

    Git管理的是修改，当你用`git add`命令后，在工作区的第一次修改被放入暂存区，准备提交，但是，在工作区的第二次修改并没有放入暂存区，所以，`git commit`只负责把暂存区的修改提交了，也就是第一次的修改被提交了，第二次的修改不会被提交

------

## 远程仓库

1.  在github上新建仓库

2.  本地库关联远程库`git remote add origin https://github.com/louisyanglu/gitlearn.git`

    远程库的名字就是`origin`，这是Git默认的叫法

3.  把本地库的所有内容推送到远程库上`git push -u origin master`

    由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。
    
4.  查看远程库`git remote -v`

5.  推送分支。把该分支上的所有本地提交推送到远程库。`git push origin 本地分支`

6.  如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；

    ![image-20210509152402820](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509152408.png)

    -   直接使用`git pull`：![](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509153216.png)
    -   将本地分支与远程分支关联起来：`git branch --set-upstream-to=origin/dev dev`
    -   再`git pull`
    -   再解决冲突，提交

7.  如果合并有冲突，则解决冲突，并在本地提交；

------

## 分支

### 分支指针

1.  `HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支

    ![](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509123208.png)

2.  每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长

### 创建分支

1.  当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交

    ```python
    git branch dev  # 创建分支
    ```

2.  把`HEAD`指向`dev`，就表示当前分支在`dev`上

    ![git-br-create](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509123347.png)

    ```python
    git checkout dev  # 切换分支
    git switch dev
    
    git checkout -b dev  # 创建、切换分支
    git switch -c dev
    git switch -c dev origin/dev  # 创建分支，对应于远程库中的dev分支
    
    git branch  # 查看分支，当前分支前面会标一个*号
    ```

3.  从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变

    ![git-br-dev-fd](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509123422.png)

### 合并分支

1.  假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并

    ![git-br-ff-merge](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509123556.png)

    ```python
    # 1、在dev分支做完工作
    # 2、切换回主分支
    git checkout master
    # 3、master合并dev分支
    git merge dev  # 合并指定分支(dev)到当前分支(master)
    ```

2.  默认使用`Fast forward`模式，这种模式下，删除分支后，会丢掉分支信息

3.  强制禁用`Fast forward`模式，Git就会在merge时**生成一个新的commit**，这样，从分支历史上就可以看出分支信息

    ```python
    git merge --no-ff -m "merge with no-ff" dev  # 禁用Fast forward模式
    # 使用git log --graph 查看分支历史图
    git log --graph --pretty=oneline --abbrev-commit
    ```

    ![git-no-ff-mode](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509131735.png)

### 删除分支

1.  合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支

    ![git-br-rm](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509123633.png)

    ```python
    git branch -d dev  # 删除分支

### 解决合并冲突

1.  dev分支和master同时修改文件，然后将dev合并到master会出现冲突

    ![git-br-feature1](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509130822.png)

2.  必须手动解决冲突，然后提交

    ![git-br-conflict-merged](https://raw.githubusercontent.com/louisyanglu/TyporaImages/master/20210509130902.png)

### 暂存任务

1.  Git还提供了一个`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作

    ```python
    git stash  # 暂存任务
    git stash list  # 查看暂存任务列表
    
    git stash apply stash@{0}  # 恢复任务
    git stash drop stash@{0}  # 删除任务列表中的任务
    git stash pop  # 恢复任务，并在列表删除
    ```

### 复制修改

1.  master分支上有个bug需要修改文件，dev分支是早期从master分支分出来的，所以，这个bug其实在当前dev分支上也存在

2.  可以直接把master分支上所做的修改复制到dev分支。注意：我们只想复制`4c805e2 fix bug 101`这个提交所做的修改，并不是把整个master分支merge过来

    ```python
    git cherry-pick 4c805e2  # git cherry-pick 复制特定提交到当前分支，避免重复劳动
    ```

------

## 标签

### 创建标签

1.  `git tag <name>`就可以打一个新标签
2.  `git tag v0.9 f52c633`对历史提交打标签
3.  `git tag -a v0.1 -m "version 0.1 released" 1094adb`对历史提交创建有说明的标签
4.  `git tag`查看标签
5.  `git show <tagname>`查看标签信息

### 操作标签

1.  `git tag -d v0.1`删除本地标签
2.  标签都只存储在本地，不会自动推送到远程。要推送某个标签到远程，使用命令`git push origin <tagname>`
3.  一次性推送全部尚未推送到远程的本地标签`git push origin --tags`
4.  `git push origin :refs/tags/<tagname>`可以删除一个远程标签

