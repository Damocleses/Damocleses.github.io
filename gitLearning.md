**前言**：
本文章主要介绍git的原理、使用和一些技巧，目的在于使读者对git的了解不仅仅局限于简单的使用push、pull命令，而要做到知其然且知其所以然。当然，本文并不会深入去探讨诸如git的实现原理之类的深层次东西，毕竟它只是一个代码管理工具罢了，作为使用者，我们只要达到真正熟练使用的地步就够了，至于更深层次的东西，诸位有兴趣的可以自行学习研究。

另外，本文分支相关图片取自[learngitbranching](https://learngitbranching.js.org/?locale=zh_CN),这是个用游戏的方式，图文结合学习git分支的网站，相当nice，推荐大家去完整过一遍，相信对于git的理解会更上一层楼
# 起源
git是由Linux的作者Linus花两周时间写出的分布式版本控制软件。在这之前，Linux社区使用BitKeeper作为版本控制系统，但是由于社区中有人试图破解BitKeeper的协议，这惹恼了BitKeeper的东家BitMover公司，于是BitMover决定收回linux社区的免费使用权。

在这样的背景下，Linus花了两周时间写出了git，在一个月内替换了BitKeeper，作为Linux的版本控制工具，并在后面不断完善，最终成为了现在代码版本控制的首选

# 基本概念
## 三种工作域
- **git目录(git direcdtory)**：即仓库(Repository)，保存项目中所有版本和相关信息，是git存放数据和信息的地方
- **工作目录(work directory)**: 是对应项目的某个版本的文件集合，对应从 git 目录中解压出来的供用户进行操作和修改的数据和信息
- **暂存目录(staging area)**:用于记录下次commit时需要保存的文件列表
## 三种状态
git管理文件存在以下三种状态：
- committed：已提交状态，表示数据文件已经被保存至本地数据仓库中。
- modified：修改状态，表示文件已被修改，但是尚未被提交(保存)。
- staged：暂存状态，表示是被标记了的被修改文件，在下次提交时会将所有标记过的修改保存。

另外新增的文件为untracked file，未在git管理范围内，需要先通过git add添加到暂存目录，然后其状态会变为staged

![原理图](https://img-blog.csdn.net/20140417104150109)

如图所示，git分为远端和本地。远程远程服务器存储了仓库信息，而本地则是三种工作域都有。
## Branch、HEAD和Commit tree
本地提交代码到远端的一般流程：
1. git add，将修改保存到暂存区(stage area)
2. git commit，将暂存区中的文件推送到本地分支，本地仓库更新
3. git push，将本地仓库的更改推送到远程仓库，远程仓库更新

可以看到，想要更新代码，commit是必不可少的。每次commit都会生成一个工作目录的快照(前提是有修改)，在git中，这些commit的快照数据使用树(tree)结构来管理，称为**提交树**(Commit Tree)或者**工作树**(Work Tree)。

Git 的**分支**(Branch)，其实本质上仅仅是指向提交对象的可变指针。分支是git的核心所在，因为分支的存在，工作树才是工作树而不是工作"线"。可以将每个分支看作工作树的分叉，项目可以在不同分支上并行开发，然后在合适的时机又可以合并在一起，这都是分支的作用。

**HEAD**表示当前所处提交位置。通常来说，HEAD是指向某个分支的，当然也可以手动切换将HEAD指向工作树中的任意commit(这种情况称为HEAD分离)。

光是看这些概念可能有点枯燥晦涩，让我们看看下面的图：
![branch](http://note.youdao.com/noteshare?id=6091bc11563a98bd1b2e82d866a05580&sub=5EC7199DE5994345AA0B54BC7EFADDE5)

图中一共有c0-c4四个提交，main、bugFix和feature三个分支，三个分支分别指向C1、C3、C4三个提交，HEAD处于分离状态，指向C2

了解了以上的基础概念以后，让我们来探讨一下git分支相关内容。

# Git分支
之前说了，分支的存在是为了并行开发，每一个分支都会指向一个具体的提交。需要多人协作的项目离不开对分支的操作。

通常来说，新建一个项目时默认分支为master，可以根据需要新建develop、release等分支。

下面介绍一些常见的git分支相关命令
- **git commit**。以当前提交为父节点生成新的子提交节点，并且当前HEAD/Branch会指向新生成的节点
- **git branch**。单独使用时查看所有分支，后面加一个branchName表示在当前节点创建名为branchName的新分支，git checkout -b branchName也能达到相同的效果，区别在于后者会将HEAD指向新创建的分支
- **git checkout** BRANCH/COMMIT。切换到对应的分支或者提交节点，之前有提过，直接切换到提交节点就是HEAD分离状态
- **git merge** BRANCH/COMMIT。将当前提交节点和指定提交节点合并并生成新的提交节点，新生成的节点有**两个父节点**。
- **git rebase** BRANCH/COMMIT。将当前分支所在节点及之前的节点中有且在另一分支/节点中所没有的节点全部按序复制一份到目标分支，然后将当前BRANCH/HEAD移到目标位置。
- **git reset** COMMIT.撤销commit到指定提交，与git branch -f CURRENT_COMMIT COMMIT效果是一样的。但是这种更改只能在本地体现，是不能同步到远程的
- **git revert** COMMIT。要想撤销能同步到远程，就得用到git revert COMMIT。此命令会通过新生成提交节点的方式撤销指定的提交节点
- **git cherry-pick** COMMIT_1 COMMIT_2 ... 。选取指定的一些节点按序rebase到当前分支
### git merge和git rebase的区别和抉择
git merge会生成一个以合并的两个节点为父节点的子节点，而git rebase则是复制变更节点到指定分支，是线性变化。

使用git rebase会使提交记录看起来很干净简洁，但缺点是会打乱提交顺序，溯源分析时不方便。

使用git merge会使提交记录变复杂，但是溯源比较方便、清晰。