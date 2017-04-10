### Git

**Git**:分布式版本控制系统

**SVN**:集中式版本控制系统，版本库集中存放在中央服务器的，中央服务器好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。

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

### 管理修改

