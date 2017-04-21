### Git

**Git**:分布式版本控制系统

**SVN**:集中式版本控制系统，版本库集中存放在中央服务器的。

创建版本库：

```
mkdir learngit
cd learngit
pwd
```

```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

`pwd`命令用于显示当前目录

`git init`初始化可管理的仓库

`git add readme.txt`把文件添加到仓库

`git commit -m "test commit"`把文件提交到仓库[-m后面输入本次提交的说明]

`git status`查看当前仓库状态

`git diff`查看difference

#### 版本回退

`git log`显示从最近到最远的提交日志

在Git中，用`HEAD`表示当前版本，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，往上100个版本可以写成`HEAD~100`。因此我们可以使用git reset命令回退到上一个版本：`git reset --hard HEAD^`

> 用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。回退到某个版本`git reset --hard commit_id`

### 工作区和暂存区

**工作区（Working Directory）:**我们自己建立的项目文件夹即工作区，比如之前建立的learngit文件夹就是一个工作区.

**版本库（Repository）:**在初始化git版本库之后会生成一个隐藏的目录.git,这个就是Git的版本库.

在.git目录里面还很多文件，其中有一个index目录，就是**暂存区(stage)**，暂存区可以理解为一个虚拟工作区，这个虚拟工作区会跟踪工作区的文件变化（增删改等操作），另外Git还为我们自动生成了一个分支master以及指向该分支的指针head.

![stage](http://img.blog.csdn.net/20160426185910609)



**说明:**平时我们使用的命令`git add readme.txt`实际上是把所有的修改从工作区提交到暂存区，`git commit -m "add readme"`是一次性把暂存区的所有修改提交到分支，因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以commit就提交到了master上了。

### 撤销修改

**管理修改:**Git跟踪并管理的是修改，而非文件，每次修改，如果不add到暂存区，那就不会加入到commit中。

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两 步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，可以使用版本回退命令`git reset --hard commit_id`，不过前提是没有推送到远程库。

### 删除文件

当我们直接将本地的文件删除了或者使用`rm`命令删除了：

`rm readme.txt`

这个时候，工作区和版本库就不一致了，`git status`命令就能查看出那些文件被删除了。

+ 假如我们确实要从版本库中删除文件，那么使用命令`git rm file`以及`commit`命令可以删除版本库中的文件。
+ 另外就是误删了工作区的文件，我们可以使用`git checkout -- file`命令将误删的文件恢复到最新版本.**（该操作将会丢失最近一次提交后你修改的内容）**


### 远程仓库

[Github](https://github.com)提供免费的Git远程仓库

```
ssh-keygen -t rsa -C "youemail@email.com"
```

通过命令行将本地的仓库与远程库关联

```
git remote add origin git@github.com:flwcy/knowledge.git
```

把本地库的内容推送到远程仓库，用`git push`命令，实际上是把当前分支master推送到远程。

```
git push -u origin master
```

>由于是第一次推送master分支，加上了-u参数，Git不但会把本地的master分支内容推送到远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令` git push origin master`把本地master分支的最新修改推送到GitHub。