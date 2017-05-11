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

`git diff HEAD -- readme.txt`命令可以查看工作区和版本库里面最新版本的区别

#### 版本回退

`git log`显示从最近到最远的提交日志

在Git中，用`HEAD`表示当前版本，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，往上100个版本可以写成`HEAD~100`。因此我们可以使用git reset命令回退到上一个版本：`git reset --hard HEAD^`

> 用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。回退到某个版本`git reset --hard commit_id`

### 工作区和暂存区

__工作区（Working Directory）:__ 我们自己建立的项目文件夹即工作区，比如之前建立的learngit文件夹就是一个工作区.

__版本库（Repository）:__ 在初始化git版本库之后会生成一个隐藏的目录.git,这个就是Git的版本库.

在.git目录里面还很多文件，其中有一个index目录，就是**暂存区(stage)**，暂存区可以理解为一个虚拟工作区，这个虚拟工作区会跟踪工作区的文件变化（增删改等操作），另外Git还为我们自动生成了一个分支master以及指向该分支的指针head.

![stage](../img/other/git_stage.jpg)



__说明:__ 平时我们使用的命令`git add readme.txt`实际上是把所有的修改从工作区提交到暂存区，`git commit -m "add readme"`是一次性把暂存区的所有修改提交到分支，因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以commit就提交到了master上了。

### 撤销修改

__管理修改:__ Git跟踪并管理的是修改，而非文件，每次修改，如果不add到暂存区，那就不会加入到commit中。

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两 步，第一步用命令`git reset HEAD file`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，可以使用版本回退命令`git reset --hard commit_id`，不过前提是没有推送到远程库。

### 删除文件

当我们直接将本地的文件删除了或者使用`rm`命令删除了：

`rm readme.txt`

这个时候，工作区和版本库就不一致了，`git status`命令就能查看出那些文件被删除了。

+ 假如我们确实要从版本库中删除文件，那么使用命令`git rm file`以及`commit`命令可以删除版本库中的文件。
+ 另外就是误删了工作区的文件，我们可以使用`git checkout -- file`命令将误删的文件恢复到最新版本.__（该操作将会丢失最近一次提交后你修改的内容）__


### 远程仓库

[Github](https://github.com)提供免费的Git远程仓库（在[Github](https://github.com)上免费托管的Git仓库是所有人可见，但是只有自己能修改），首先检查SSH Key是否存在

```
ls -al ~/.ssh
```

不存在则创建SSH Key:

```
ssh-keygen -t rsa -C "youemail@email.com"
```
现在你的私钥被放在了~/.ssh/id_rsa 这个文件里，而公钥被放在了 ~/.ssh/id_rsa.pub 这个文件里。

> SSH key提供了一种与GitHub通信的方式，通过这种方式，能够在不输入密码的情况下，将GitHub作为自己的remote端服务器，进行版本控制.

> git可使用rsa，rsa要解决的一个核心问题是，如何使用一对特定的数字，使其中一个数字可以用来加密，而另外一个数字可以用来解密。这两个数字就是你在使用git和github的时候所遇到的public key也就是公钥以及private key私钥。
>
> 其中，公钥就是那个用来加密的数字，这也就是为什么你在本机生成了公钥之后，要上传到github的原因。从github发回来的，用那公钥加密过的数据，可以用你本地的私钥来还原。

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

另外使用`git clone`克隆远程仓库到本地。

```
git clone git@github.com:flwcy/knowledge.git
```

### 分支管理

__分支__就是从主线上分离出来进行另外的操作，而又不影响主线，主线又可以继续干其他的事。在最开始的时候，`master`分支是一条线，git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点：

![git_branch_01](../img/other/git_branch_01.png)

每次提交，`master`分支都会向前移动一步。当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

![git_branch_02](../img/other/git_branch_02.png)

Git创建一个分支是很快的，因为除了增加一个`dev`指针，改变`HEAD`的指向，工作区的文件都没有任何变化！不过，从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

![git_branch_03](../img/other/git_branch_03.png)

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并：

![git_branch_04](../img/other/git_branch_04.png)

合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

![git_branch_05](../img/other/git_branch_05.png)

查看分支：`git branch`

> 当前分支前面会标一个`*`号。

创建分支：`git branch <name>`

切换分支：`git checkout <name>`

创建+切换分支：`git checkout -b <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

#### 解决冲突

`master`分支和`feature1`分支各自都分别有新的提交，变成了这样：

![git_branch_06](../img/other/git_branch_06.png)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，__但这种合并就可能会有冲突，必须手动解决冲突后再提交。__`git status`也可以告诉我们冲突的文件。Git用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容。`master`分支和`feature1`分支变成了下图所示：
![git_branch_07](../img/other/git_branch_07.png)
用`git log --graph`命令可以看到分支的合并情况。