# 简析 Svn 与 Git

## 一、Git 是什么？

​	简单来说，Git 是一种分布式的版本管理系统，在保存和对待各种信息的时候与其它版本控制系统（如 Subversion 和 Perforce 等）有很大差异，尽管操作起来的命令形式非常相近。

### 1.1 文件管理

#### 1.1.1 直接记录快照，而非差异比较

​	Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方法。 概念上来区分，其它大部分系统以文件变更列表的方式存储信息。 这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。

![image-20190326162324618](/Users/tlp/Library/Application Support/typora-user-images/image-20190326162324618.png)

​	Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

![image-20190326162400778](/Users/tlp/Library/Application Support/typora-user-images/image-20190326162400778.png)

####  1.1.2 近乎所有操作都是本地执行

​	与 Svn 不同，在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。 如果你习惯于所有操作都有网络延时开销的集中式版本控制系统，Git 在这方面会让你感到速度之神赐给了 Git 超凡的能量。 因为你在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。

#### 1.1.3 Git 保证完整性

​	Git 中所有数据在存储前都计算校验和，然后以校验和来引用。 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 这个功能建构在 Git 底层，是构成 Git 哲学不可或缺的部分。 若你在传送过程中丢失信息或损坏文件，Git 就能发现。

​	Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成字符串，基于 Git 中文件的内容或目录结构计算出来。 SHA-1 哈希看起来是这样：

> 24b9da6552252987aa493b52f8696cd6d3b00373

​	在平常的使用中，我们可以使用 hash 值的前五位来简单标记版本。

#### 1.1.4 Git文件的三种状态

​	Git 的文件有三种状态，你的文件可能处于其中之一：已提交（committed）、已修改（modified）和已暂存（staged）。 已提交表示数据已经安全的保存在本地数据库中。 已修改表示修改了文件，但还没保存到数据库中。 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

​	由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。![image-20190326162811867](/Users/tlp/Library/Application Support/typora-user-images/image-20190326162811867.png)

### 1.2 仓库

#### 1.2.1 获取 Git 仓库

​	有两种取得 Git 项目仓库的方法。 第一种是在现有项目或目录下导入所有文件到 Git 中； 第二种是从一个服务器克隆一个现有的 Git 仓库。

##### 在现有目录中初始化仓库

​	如果你打算使用 Git 来对现有的项目进行管理，你只需要进入该项目目录并输入：

```console
$ git init
```

​	该命令将创建一个名为 `.git` 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干。 但是，在这个时候，我们仅仅是做了一个初始化的操作，你的项目里的文件还没有被跟踪。

​	如果你是在一个已经存在文件的文件夹（而不是空文件夹）中初始化 Git 仓库来进行版本控制的话，你应该开始跟踪这些文件并提交。 你可通过 `git add` 命令来实现对指定文件的跟踪，然后执行 `git commit`提交：

```console
$ git add *.c
$ git add LICENSE
$ git commit -m 'initial project version'
```

##### 克隆现有的仓库

​	如果你想获得一份已经存在了的 Git 仓库的拷贝，比如说，你想为某个开源项目贡献自己的一份力，这时就要用到 `git clone` 命令。 如果你对其它的 VCS 系统（比如说Subversion）很熟悉，请留心一下你所使用的命令是"clone"而不是"checkout"。 这是 Git 区别于其它版本控制系统的一个重要特性，Git 克隆的是该 Git 仓库服务器上的几乎所有数据，而不是仅仅复制完成你的工作所需要文件。 当你执行 `git clone` 命令的时候，默认配置下远程 Git 仓库中的每一个文件的每一个版本都将被拉取下来。 事实上，如果你的服务器的磁盘坏掉了，你通常可以使用任何一个克隆下来的用户端来重建服务器上的仓库（虽然可能会丢失某些服务器端的挂钩设置，但是所有版本的数据仍在）。

​	克隆仓库的命令格式是 `git clone [url]` 。 比如，要克隆本文档配套的 Demo，可以用下面的命令：

```console
$ git clone https://github.com/LePengTian/GitDemo.git
```

​	这会在当前目录下创建一个名为 “GitDemo” 的目录，并在这个目录下初始化一个 `.git` 文件夹，从远程仓库拉取下所有数据放入 `.git` 文件夹，然后从中读取最新版本的文件的拷贝。 如果你进入到这个新建的 `GitDemo` 文件夹，你会发现所有的项目文件已经在里面了，准备就绪等待后续的开发和使用。 如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以使用如下命令：

```console
$ git clone https://github.com/LePengTian/GitDemo.git myGitDemo
```

​	这将执行与上一个命令相同的操作，不过在本地创建的仓库名字变为 `myGitDemo`。

#### 1.2.2 记录每次更新到仓库

​	现在我们手上有了一个真实项目的 Git 仓库，并从这个仓库中取出了所有文件的工作拷贝。 接下来，对这些文件做些修改，在完成了一个阶段的目标之后，提交本次更新到仓库。

​	请记住，你工作目录下的每一个文件都不外乎这两种状态：已跟踪或未跟踪。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。

​	编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 我们逐步将这些修改过的文件放入暂存区，然后提交所有暂存了的修改，如此反复。所以使用 Git 时文件的生命周期如下：

### ![image-20190327103601762](/Users/tlp/Library/Application Support/typora-user-images/image-20190327103601762.png)



### 1.3 分支——“必杀技特性”

​	几乎所有的版本控制系统都以某种形式支持分支。 使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。 在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。本节我会用几个单的例子带大家认识分支的概念和用法，我会在**第三章**阐述如何使用 SourceTree 进行这些操作。

#### 1.3.1 Git 的分支功能的实现基于快照与指针

​	在进行提交操作时，Git 会保存一个提交对象（commit object）。知道了 Git 保存数据的方式，我们可以很自然的想到——该提交对象会包含一个指向暂存内容快照的指针。 但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象，

​	为了更加形象地说明，我们假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。 暂存操作会为每一个文件计算校验和（使用我们在 **1.1** 中提到的 SHA-1 哈希算法），然后会把当前版本的文件快照保存到 Git 仓库中（Git 使用 blob 对象来保存它们），最终将校验和加入到暂存区域等待提交：

```console
$ git add README test.rb LICENSE
$ git commit -m 'The initial commit of my project'
```

​	当使用 `git commit` 进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和，然后在 Git 仓库中这些校验和保存为树对象。 随后，Git 便会创建一个提交对象，它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。如此一来，Git 就可以在需要的时候重现此次保存的快照。

​	现在，Git 仓库中有五个对象：三个 blob 对象（保存着文件快照）、一个树对象（记录着目录结构和 blob 对象索引）以及一个提交对象（包含着指向前述树对象的指针和所有提交信息）。

![image-20190326164910154](/Users/tlp/Library/Application Support/typora-user-images/image-20190326164910154.png)

做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。![image-20190326165153583](/Users/tlp/Library/Application Support/typora-user-images/image-20190326165153583.png)

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 

#### 1.3.2 分支创建

​	Git 是怎么创建新分支的呢？ 很简单，它只是为你创建了一个可以移动的新的指针。 比如，创建一个 testing 分支， 你需要使用 `git branch` 命令：

```console
$ git branch testing
```

​	这会在当前所在的提交对象上创建一个指针。

![image-20190326165705355](/Users/tlp/Library/Application Support/typora-user-images/image-20190326165705355.png)

​	那么，Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 `HEAD` 的特殊指针。 请注意它和许多其它版本控制系统（如 Subversion 或 CVS）里的 `HEAD` 概念完全不同。 在 Git 中，它是一个指针，指向当前所在的本地分支（译注：将 `HEAD` 想象为当前分支的别名）。 在本例中，你仍然在 `master`分支上。 因为 `git branch` 命令仅仅 *创建* 一个新分支，并不会自动切换到新分支中去。

![image-20190326181135817](/Users/tlp/Library/Application Support/typora-user-images/image-20190326181135817.png)

#### 1.3.3 分支切换

​	要切换到一个已存在的分支，你需要使用 `git checkout` 命令。 我们现在切换到新创建的 `testing` 分支去：

```console
$ git checkout testing
```

这样 `HEAD` 就指向 `testing` 分支了。

![image-20190326181241674](/Users/tlp/Library/Application Support/typora-user-images/image-20190326181241674.png)

那么，这样的实现方式会给我们带来什么好处呢？ 现在不妨再提交一次：

```console
$ vim test.rb
$ git commit -a -m 'made a change'
```

![image-20190326181306126](/Users/tlp/Library/Application Support/typora-user-images/image-20190326181306126.png)

​	如图所示，你的 `testing` 分支向前移动了，但是 `master` 分支却没有，它仍然指向运行 `git checkout` 时所指的对象。 这就有意思了，现在我们切换回 `master` 分支看看：

```console
$ git checkout master
```

![image-20190326181401532](/Users/tlp/Library/Application Support/typora-user-images/image-20190326181401532.png)

​	这条命令做了两件事。 一是使 HEAD 指回 `master` 分支，二是将工作目录恢复成 `master` 分支所指向的快照内容。 也就是说，你现在做修改的话，项目将始于一个较旧的版本。 本质上来讲，这就是忽略 `testing` 分支所做的修改，以便于向另一个方向进行开发。

​	我们不妨再稍微做些修改并提交：

```console
$ vim test.rb
$ git commit -a -m 'made other changes'
```

​	现在，这个项目的提交历史已经产生了分叉。 因为刚才你创建了一个新分支，并切换过去进行了一些工作，随后又切换回 master 分支进行了另外一些工作。 上述两次改动针对的是不同分支：你可以在不同分支间不断地来回切换和工作，并在时机成熟时将它们合并起来。 而所有这些工作，你需要的命令只有 `branch`、`checkout` 和 `commit`。

![image-20190326181455607](/Users/tlp/Library/Application Support/typora-user-images/image-20190326181455607.png)

#### 1.3.4 分支合并——一个简单的例子

​	让我们来看一个简单的分支新建与分支合并的例子，实际工作中你可能会用到类似的工作流。 你将经历如下步骤：

1. 开发某个网站。

2. 为实现某个新的需求，创建一个分支。

3. 在这个分支上开展工作。

   正在此时，你突然接到一个电话说有个很严重的问题需要紧急修补。 你将按照如下方式来处理：

1. 切换到你的线上分支（production branch）。
2. 为这个紧急任务新建一个分支，并在其中修复它。
3. 在测试通过之后，切换回线上分支，然后合并这个修补分支，最后将改动推送到线上分支。
4. 切换回你最初工作的分支上，继续工作。

##### 新建分支

首先，我们假设你正在你的项目上工作，并且已经有一些提交。

![一个简单的提交历史。](https://git-scm.com/book/en/v2/images/basic-branching-1.png)

​	现在，你已经决定要解决你的公司使用的问题追踪系统中的 #53 问题。 想要新建一个分支并同时切换到那个分支上，你可以运行一个带有 `-b` 参数的 `git checkout` 命令：

```console
$ git checkout -b iss53
Switched to a new branch "iss53"
```

​	它是下面两条命令的简写：

```console
$ git branch iss53
$ git checkout iss53
```

![创建一个新分支指针。](https://git-scm.com/book/en/v2/images/basic-branching-2.png)

​	你继续在 #53 问题上工作，并且做了一些提交。 在此过程中，`iss53` 分支在不断的向前推进，因为你已经检出到该分支（也就是说，你的 `HEAD` 指针指向了 `iss53` 分支）

```console
$ vim index.html
$ git commit -a -m 'added a new footer [issue 53]'
```

![iss53 分支随着工作的进展向前推进。](https://git-scm.com/book/en/v2/images/basic-branching-3.png)

​	现在你接到那个电话，有个紧急问题等待你来解决。 有了 Git 的帮助，你不必把这个紧急问题和 `iss53`的修改混在一起，你也不需要花大力气来还原关于 53# 问题的修改，然后再添加关于这个紧急问题的修改，最后将这个修改提交到线上分支。 你所要做的仅仅是切换回 `master` 分支。

​	但是，在你这么做之前，要留意你的工作目录和暂存区里那些还没有被提交的修改，它可能会和你即将检出的分支产生冲突从而阻止 Git 切换到该分支。 最好的方法是，在你切换分支之前，保持好一个干净的状态。 有一些方法可以绕过这个问题（即，保存进度（stashing） 和 修补提交（commit amending））。 现在，我们假设你已经把你的修改全部提交了，这时你可以切换回 `master` 分支了：

```console
$ git checkout master
Switched to branch 'master'
```

​	这个时候，你的工作目录和你在开始 #53 问题之前一模一样，现在你可以专心修复紧急问题了。 请牢记：当你切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样。

​	接下来，你要修复这个紧急问题。 让我们建立一个针对该紧急问题的分支（hotfix branch），在该分支上工作直到问题解决：

```console
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
$ vim index.html
$ git commit -a -m 'fixed the broken email address'
[hotfix 1fb7853] fixed the broken email address
 1 file changed, 2 insertions(+)
```

![基于 `master` 分支的紧急问题分支（hotfix branch）。](https://git-scm.com/book/en/v2/images/basic-branching-4.png)

​	你可以运行你的测试，确保你的修改是正确的，然后将其合并回你的 `master` 分支来部署到线上。 你可以使用 `git merge` 命令来达到上述目的：

```console
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
```

在合并的时候，你应该注意到了"快进（fast-forward）"这个词。 由于当前 `master` 分支所指向的提交是你当前提交（有关 hotfix 的提交）的直接上游，所以 Git 只是简单的将指针向前移动。 换句话说，当你试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候，只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进（fast-forward）”。

现在，最新的修改已经在 `master` 分支所指向的提交快照中，你可以着手发布该修复了。

![`master` 被快进到 `hotfix`。](https://git-scm.com/book/en/v2/images/basic-branching-5.png)

​	关于这个紧急问题的解决方案发布之后，你准备回到被打断之前时的工作中。 然而，你应该先删除 `hotfix` 分支，因为你已经不再需要它了 —— `master` 分支已经指向了同一个位置。 你可以使用带 `-d`选项的 `git branch` 命令来删除分支：

```console
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
```

现在你可以切换回你正在工作的分支继续你的工作，也就是针对 #53 问题的那个分支（iss53 分支）。

```console
$ git checkout iss53
Switched to branch "iss53"
$ vim index.html
$ git commit -a -m 'finished the new footer [issue 53]'
[iss53 ad82d7a] finished the new footer [issue 53]
1 file changed, 1 insertion(+)
```

![继续在 `iss53` 分支上的工作。](https://git-scm.com/book/en/v2/images/basic-branching-6.png)

​	你在 `hotfix` 分支上所做的工作并没有包含到 `iss53` 分支中。 如果你需要拉取 `hotfix` 所做的修改，你可以使用 `git merge master` 命令将 `master` 分支合并入 `iss53` 分支，或者你也可以等到 `iss53`分支完成其使命，再将其合并回 `master` 分支。

##### 合并分支

​	假设你已经修正了 #53 问题，并且打算将你的工作合并入 `master` 分支。 为此，你需要合并 `iss53` 分支到 `master` 分支，这和之前你合并 `hotfix` 分支所做的工作差不多。 你只需要检出到你想合并入的分支，然后运行 `git merge` 命令：

```console
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```

​	这和你之前合并 `hotfix` 分支的时候看起来有一点不一样。 在这种情况下，你的开发历史从一个更早的地方开始分叉开来（diverged）。 因为，`master` 分支所在提交并不是 `iss53` 分支所在提交的直接祖先，Git 不得不做一些额外的工作。 出现这种情况的时候，Git 会使用两个分支的末端所指的快照（`C4` 和 `C5`）以及这两个分支的工作祖先（`C2`），做一个简单的三方合并。

![一次典型合并中所用到的三个快照。](https://git-scm.com/book/en/v2/images/basic-merging-1.png)

​	和之前将分支指针向前推进所不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。

![一个合并提交。](https://git-scm.com/book/en/v2/images/basic-merging-2.png)

​	需要指出的是，Git 会自行决定选取哪一个提交作为最优的共同祖先，并以此作为合并的基础；这和更加古老的 CVS 系统或者 Subversion （1.5 版本之前）不同，在这些古老的版本管理系统中，用户需要自己选择最佳的合并基础。 Git 的这个优势使其在合并操作上比其他系统要简单很多。

​	既然你的修改已经合并进来了，你已经不再需要 `iss53` 分支了。 现在你可以在任务追踪系统中关闭此项任务，并删除这个分支。

```console
$ git branch -d iss53
```

## 二、为什么要用 Git——与 Svn 的区别

​	通过**第一章**我们对 Git 有了基本的了解，Git 与 Svn 最主要的区别是他们分别属于分布式和集中式的版本管理系统。

​	集中化的版本控制系统（Centralized Version Control Systems，简称 CVCS）诸如 CVS、Subversion 以及 Perforce 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。 多年以来，这已成为版本控制系统的标准做法。

​	集中式模型：

![image-20190327172613717](/Users/tlp/Library/Application%20Support/typora-user-images/image-20190327172613717.png)

​	这种做法带来了许多好处，每个人都可以在一定程度上看到项目中的其他人正在做些什么。 而管理员也可以轻松掌控每个开发者的权限，并且管理一个 CVCS 要远比在各个客户端上维护本地数据库来得轻松容易。

​	事分两面，有好有坏。 这么做最显而易见的缺点是中央服务器的单点故障。 如果宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。 如果中心数据库所在的磁盘发生损坏，又没有做恰当备份，毫无疑问你将丢失所有数据——包括项目的整个变更历史，只剩下人们在各自机器上保留的单独快照。 

​	分布式版本控制系统（Distributed Version Control System，简称 DVCS）像 Git、Mercurial、Bazaar 以及 Darcs 等，客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

分布式模型：



![image-20190327172627016](/Users/tlp/Library/Application%20Support/typora-user-images/image-20190327172627016.png)

​	更进一步，许多这类系统都可以指定和若干不同的远端代码仓库进行交互。籍此，你就可以在同一个项目中，分别和不同工作小组的人相互协作。 你可以根据需要设定不同的协作流程，比如层次模型式的工作流，而这在以前的集中式系统中是无法实现的。

​	下表对 Svn 与 Git 进行了主要特点的对比：

|                  | 集中式（SVN）                                                | 分布式（Git）                                                |
| :--------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 是否有中央服务器 | 有。开发人员需要从中央服务器获得最新版本的项目然后在本地开发，开发完推送给中央服务器。因此脱离服务器开发者是几乎无法工作的。服务器如果出了故障数据很难找回 | 没有中央服务器，开发人员本地都有本地仓库（ Local Repository），适合分布式开发，强调个体，公共服务器压力和数据量都不会特别大 |
|     网络依赖     | 必须要联网才能工作，而且对网络的依赖性较强，如果推送的文件比较大而且网络状况欠佳，则提交文件的速度会受到很大的限制 | 分布式在没有网络的情况下也可以执行commit、查看版本提交记录、以及分支操作，在有网络的情况下执行 push 到 远程仓库（Remote Repository） |
|   文件存储格式   | 按照原始文件存储，体积较大                                   | 按照元数据方式存储，体积很小                                 |
|   是否有版本号   | 有连续的版本号                                               | 没有，有通过文件计算的哈希值作为版本代号                     |
|  分支操作的影响  | 合并分支的开销比较大，创建新的分支则所有的人都会拥有和你一样的分支 | 分支的使用非常简单，分支操作不会影响其他开发人员             |
|       提交       | 提交的文件会直接记录到中央版本库                             | 提交是本地操作，意味着更快的速度，需要执行push操作才会提交到主要版本库 |
|       检出       | 可以将整个库检出到工作区，也可以将某个目录检出到工作区       | Git 没有部分检出，但是可以通过子模块处理这个问题             |
|       权限       | 通过对文件目录授权来实现权限管理，子目录默认继承父目录的权限。但是权限不能在分支中继承，不能对单个文件授权。 | Git的授权模型只能实现非零即壹式的授权，要么拥有全部的写权限，要么没有写权限，要么拥有整个版本库的读权限，要么禁用 |
|     学习成本     | 相对容易，符合一般人的思维习惯                               | 学习周期相对比较长，学会用命令操作 Git 需要一定的熟练度      |

## 三、如何使用 Git——SourceTree

​	Git 对命令操作有非常友好的支持，建议有时间的同学尽可能学习使用命令去操作 Git（[Git 命令列表](<https://git-scm.com/book/zh/v2/%E9%99%84%E5%BD%95-C%3A-Git-%E5%91%BD%E4%BB%A4-%E8%AE%BE%E7%BD%AE%E4%B8%8E%E9%85%8D%E7%BD%AE>)）。不想使用命令的同学也没关系，本章将借用一个 demo 为大家介绍 Mac 上的可视化 Git 应用程序 SourceTree 的使用方法。

### 3.1 SourceTree简介

​	SourceTree是一款基于界面的git版本控制软件，支持add、commit、clone、push、pull 和merge等操作，操作也比较简单。 
![image-20190328170935145](/Users/tlp/Library/Application Support/typora-user-images/image-20190328170935145.png)

### 3.2 SourceTree的安装与注册

​	首先去官网[(链接)](https://www.sourcetreeapp.com/)下载最新版SourceTree，也可以直接使用附件中的安装包: 
![image-20190328170606671](/Users/tlp/Library/Application Support/typora-user-images/image-20190328170606671.png)

接着注册

配置完之后，就到了SourceTree的仓库管理界面： 

![image-20190328172434977](/Users/tlp/Library/Application Support/typora-user-images/image-20190328172434977.png)

另外可以将你的账号与 GitHub 账号绑定，偏好设置--账户--添加--链接账号：

![image-20190328172237435](/Users/tlp/Library/Application Support/typora-user-images/image-20190328172237435.png)

​	在弹出的 GItHub 页面登陆你的账号，确认绑定即可。

### 3.3 SourceTree 配置git仓库

#### 3.3.1 从URL克隆

​	即将一个远程代码库克隆到本地： 

![image-20190328181830524](/Users/tlp/Library/Application Support/typora-user-images/image-20190328181830524.png)

添加git仓库的URL链接，选择本地存放位置以及项目名称，可以直接用 GitDemo 进行测试（地址：https://github.com/LePengTian/GitDemo.git）： 
![image-20190328182045842](/Users/tlp/Library/Application Support/typora-user-images/image-20190328182045842.png)

#### 3.3.2 打开本地代码库

​	直接打开本地已经存在的git仓库： 

![image-20190328195415679](/Users/tlp/Library/Application Support/typora-user-images/image-20190328195415679.png)

​	选择本地仓库(项目工程)的文件夹，打开即可。

### 4.SourceTree git界面介绍

![Snip20160927_42.png-252.6kB](http://static.zybuluo.com/Sweetfish/1msghy4b77k6c816tfh0kkxv/Snip20160927_42.png)
如上图，SourceTreegit支持中文界面，基本命令对比如下：

- 提交：git commit
- 拉取：git pull
- 推送：git push
- 抓取：git fetch

主界面中还有一个`未暂存文件`和`已暂存文件`，勾选`未暂存文件`会自动将`未暂存文件`添加到`已暂存文件`区域，就是一次`git add`命令。

左边栏可以看到本地和远程的分支，还有本地文件的工作状态。

### 5.代码提交

1.勾选代码，添加(add)到暂存区域 
![Snip20160927_21.png-290kB](http://static.zybuluo.com/Sweetfish/mma1e7bsaen8dht3q8va05uc/Snip20160927_21.png)

代码查看： 
![Snip20160927_20.png-277.9kB](http://static.zybuluo.com/Sweetfish/81y289wocsrtx7lkh3d4674m/Snip20160927_20.png)

2.提交(commit)代码到本地仓库

- 点击左上提交按钮
- 写提交日志
- 点击右下提交按钮
- 之后再点击“推送”按钮推送到远程

![Snip20160927_24.png-283.7kB](http://static.zybuluo.com/Sweetfish/yyxi60bcz7pvjszv4bsqtb8j/Snip20160927_24.png)
![Snip20160927_15.png-128.4kB](http://static.zybuluo.com/Sweetfish/0ljnbqrfmw99f1994hwx1tdl/Snip20160927_15.png)
![Snip20160930_9.png-125.5kB](http://static.zybuluo.com/Sweetfish/4c4ue71n74wupc6581i9v9cf/Snip20160930_9.png)

3.撤销添加 
即从暂存区到未暂存区，直接取消文件勾选即可： 
![Snip20160927_22.png-150.6kB](http://static.zybuluo.com/Sweetfish/6vk3pf1ntuen5xsq2s3ww917/Snip20160927_22.png)

4.放弃修改 
即放弃该文件的修改，恢复到修改前： 
![Snip20160927_26.png-255.8kB](http://static.zybuluo.com/Sweetfish/1t2zdcip4p9ht1t5mpj5jjb2/Snip20160927_26.png)

### 6.代码拉取和冲突解决

#### 6.1 拉取代码

点击“拉取”按钮，拉取远程仓库代码，确定拉取的远程`分支`： 
![Snip20160930_11.png-228.7kB](http://static.zybuluo.com/Sweetfish/n23pe5t1nkj8c6a6n6ajylei/Snip20160930_11.png)

#### 6.2 冲突解决

1.冲突出现 
如果拉取出现冲突，会弹出“合并冲突”的提醒弹窗，说明代码有冲突，需要人工合并代码。 
![Snip20160927_29.png-271.9kB](http://static.zybuluo.com/Sweetfish/mw0qda5orkryq0jaiou8179f/Snip20160927_29.png)

冲突文件的标记前面会出现"`感叹号❗`️"冲突标记： 
![Snip20160927_30.png-294.3kB](http://static.zybuluo.com/Sweetfish/udw61m1vhm36rfvrhe5iup4p/Snip20160927_30.png)

2.合并冲突 
`右键点击文件-->解决冲突-->启动外部合并工具`，会调用FileMerge工具对代码进行合并。(**此外也可以在Xcode或其他第三方工具如Beyond Compare中手动修改后保存**) 
![Snip20160927_32.png-314kB](http://static.zybuluo.com/Sweetfish/iwqket06ijas4ba77maa0wxy/Snip20160927_32.png)

FileMerge默认使用远程替换本地的代码(即打开FileMerge后，代码默认已经被它合并了，但还需要人工观察合并是否正确)：

- FileMerge中用`箭头→`指向的代码替换原代码，作为最新代码；
- `红色`的箭头表明是冲突代码；
- 最下面的区域显示的是最后合并的代码效果；

![Snip20160927_36.png-309.6kB](http://static.zybuluo.com/Sweetfish/oyy47bib8yivobaek574wyo8/Snip20160927_36.png)
![Snip20160927_33.png-298.6kB](http://static.zybuluo.com/Sweetfish/4b2zl6458iq86am9ekoijqg9/Snip20160927_33.png)

手动修改方式：

- 选中箭头

- 在右下角选择操作方式

   

  - Choose left：选择左边的代码
  - Choose right：选择右边的代码
  - Choose both(left first)：全部选择左边的代码
  - Choose both(right first)：全部选择右边的代码
  - Choose neither：都不选，保留冲突前的代码

- 合并完成之后，点击软件`File菜单-->save merge`保存合并代码。

![Snip20160927_35.png-213.9kB](http://static.zybuluo.com/Sweetfish/25xyjnomw6g7po9xphm81i8u/Snip20160927_35.png)
![Snip20160927_37.png-95.5kB](http://static.zybuluo.com/Sweetfish/c66es0wxzz0sa3b7r9k7yqta/Snip20160927_37.png)

提交冲突文件就和`代码提交`一样了。 
![Snip20160927_38.png-278.7kB](http://static.zybuluo.com/Sweetfish/4ur8ibzo7gwv0wyocv9tb7bk/Snip20160927_38.png)

此外，代码冲突会产生一些`备份文件`需要**手动删除**，或者不勾选提交。可以通过git命令不产生备份文件 
![Snip20160927_39.png-262.2kB](http://static.zybuluo.com/Sweetfish/d0ghdph1am5mg0gty2ku8b5h/Snip20160927_39.png)

**多余的.orig文件删除：** 
这个是由于git自身造成的，它会在解决冲突后生成一个原来冲突的备份，通过命令可以去掉

```
git config --global mergetool.keepBackup false
```





## 参考链接

【1】[Git官方文档中文版](<https://git-scm.com/book/zh/v2>)

【2】[Svn 与 Git 比较](<http://www.360doc.com/content/12/1228/20/11220452_256857021.shtml>)

【3】[SourceTree官方文档](<https://confluence.atlassian.com/get-started-with-sourcetree?_ga=2.238387269.450685589.1553763967-1291292951.1553763967>)

【4】[MAC上git版本管理软件SourceTree的使用](<https://www.zybuluo.com/Sweetfish/note/516129>)

【5】[SVN 教程-菜鸟教程](<http://www.runoob.com/svn/svn-tutorial.html>)

【6】[Git 教程-菜鸟教程](<http://www.runoob.com/git/git-tutorial.html>)

