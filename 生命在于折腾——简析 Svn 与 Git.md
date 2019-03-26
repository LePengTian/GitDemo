# 简析 Svn 与 Git

## 一、Git 是什么？

​	 简单来说，Git 是一种分布式的版本管理系统，在保存和对待各种信息的时候与其它版本控制系统（如 Subversion 和 Perforce 等）有很大差异，尽管操作起来的命令形式非常相近。

### 1.1 文件管理

#### 直接记录快照，而非差异比较

​	Git 和其它版本控制系统（包括 Subversion 和近似工具）的主要差别在于 Git 对待数据的方法。 概念上来区分，其它大部分系统以文件变更列表的方式存储信息。 这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。

![image-20190326162324618](/Users/tlp/Library/Application Support/typora-user-images/image-20190326162324618.png)

​	Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

![image-20190326162400778](/Users/tlp/Library/Application Support/typora-user-images/image-20190326162400778.png)

####  近乎所有操作都是本地执行

​	与 Svn 不同，在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。 如果你习惯于所有操作都有网络延时开销的集中式版本控制系统，Git 在这方面会让你感到速度之神赐给了 Git 超凡的能量。 因为你在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。

#### Git 保证完整性

​	Git 中所有数据在存储前都计算校验和，然后以校验和来引用。 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 这个功能建构在 Git 底层，是构成 Git 哲学不可或缺的部分。 若你在传送过程中丢失信息或损坏文件，Git 就能发现。

​	Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成字符串，基于 Git 中文件的内容或目录结构计算出来。 SHA-1 哈希看起来是这样：

> 24b9da6552252987aa493b52f8696cd6d3b00373

#### 三种状态

​	Git 有三种状态，你的文件可能处于其中之一：已提交（committed）、已修改（modified）和已暂存（staged）。 已提交表示数据已经安全的保存在本地数据库中。 已修改表示修改了文件，但还没保存到数据库中。 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

​	由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。![image-20190326162811867](/Users/tlp/Library/Application Support/typora-user-images/image-20190326162811867.png)

### 1.2 分支——“必杀技特性”

​	几乎所有的版本控制系统都以某种形式支持分支。 使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。 在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。

#### Git 的分支功能的实现基于快照与指针

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

#### 分支创建

​	Git 是怎么创建新分支的呢？ 很简单，它只是为你创建了一个可以移动的新的指针。 比如，创建一个 testing 分支， 你需要使用 `git branch` 命令：

```console
$ git branch testing
```

​	这会在当前所在的提交对象上创建一个指针。

![image-20190326165705355](/Users/tlp/Library/Application Support/typora-user-images/image-20190326165705355.png)

​	那么，Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 `HEAD` 的特殊指针。 请注意它和许多其它版本控制系统（如 Subversion 或 CVS）里的 `HEAD` 概念完全不同。 在 Git 中，它是一个指针，指向当前所在的本地分支（译注：将 `HEAD` 想象为当前分支的别名）。 在本例中，你仍然在 `master`分支上。 因为 `git branch` 命令仅仅 *创建* 一个新分支，并不会自动切换到新分支中去。

![image-20190326181135817](/Users/tlp/Library/Application Support/typora-user-images/image-20190326181135817.png)

#### 分支切换

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

稍后我会在**第三章**教大家如何使用 SourceTree 进行这些操作。

