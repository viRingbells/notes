---
title: Git Merge 与 Rebase 比较
tags:
  - 学习
categories:
  - 工程
date: 2019-03-07 21:08:17
---


> 原文：[Merging vs. Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

# 概念概述

关于`rebase`和`merge`，需要了解的第一件事就是，它们解决的是同一个问题：将一个分支的改动合并到另外一个分支里。不过它们的工作方式却全然不同。

考虑一下你正在分支`feature`下进行开发，团队中其他成员向主干提交代码的情况。这会造成`git`使用者常见的历史分叉的情况。

![分叉图片](https://wac-cdn.atlassian.com/dam/jcr:01b0b04e-64f3-4659-af21-c4d86bc7cb0b/01.svg?cdnVersion=lc)

假设你正在开发的内容，与主干中发生变更的部分有关联，那么你就需要使用`rebase`或者`merge`将主干的变更合并到自己的分支里。

## Merge

最简单的方法就是用下面这组命令将主干代码合并到分支里：

```
$ git checkout feature
$ git merge master
```

或者将它们简化到一个命令里：

```
$ git merge feature master
```

然后会产生一个新的将两边提交历史关联在一起的新的提交版本：『合并提交』，然后时整个提交历史结构变成下面这个样子：

![Merge提交历史图片](https://wac-cdn.atlassian.com/dam/jcr:e229fef6-2c2f-4a4f-b270-e1e1baa94055/02.svg?cdnVersion=lc)

`merge`操作不会对分支产生任何影响，因此这个操作也是『无害』的操作。相比`rebase`能减少许多可能的风险。（后面会说到`rebase`给分支带来的影响和隐患。）

这也意味着，每当你需要把主干代码合并到分支里时，你就需要一次额外的『合并提交』。尤其是主干变更非常频繁时，这些『合并提交』就会对你分支的提交历史形成干扰。这个问题虽然可以通过`git log`的高级特性来缓解，但是仍然会给其他维护者理解此分支的提交历史带来不小的障碍。

## Rebase

除了`merge`，你也可以使用`rebase`来将分支的基准重新定位到主干变更后的版本上。

```
$ git checkout feature
$ git rebase master
```

这个操作会高效的将主干代码合并到分支中，并且将分支的基准变更到主干的最新版本上。但是与『合并提交』不同，`rebase`会为分支上每一个的历史提交都创建一个新的提交，并重写到分支的历史上。

![Rebase提交历史图片](https://wac-cdn.atlassian.com/dam/jcr:5b153a22-38be-40d0-aec8-5f2fffc771e5/03.svg?cdnVersion=ld)

`Rebase`的主要好处就是，它使分支的提交历史变得干净。一方面，它不会产生像`merge`操作那样带来的额外的提交信息；另一方面，从上图也可以看出来，它使提交历史变成了完美的线性模式——你可以从分支的最新版本一直追溯到最初版本，中间不会有任何的分叉。这也使得用诸如`git log`、`git bisect`和`gitk`之类的命令来处理项目时更加容易。

当然，这个全新的提交历史也在安全性和可追踪性上存在一些折衷。如果你不遵守[Rebase的黄金法则](#Rebase黄金法则)，那么对于团队合作的工作流来说，重写提交历史有着灾难性的隐患。另外一方面，`rebase`会丢失合并的信息——你无法知道是什么时候把上游的变动引入到当前分支的。

## 互动式Rebase

互动式Rebase允许你在进行`Rebase`时更改提交信息。这种方式允许用户全面控制分支的提交历史，因此也比自动`rebase`更加强大。一般来说，在把一个分支合入主干时，会使用这个方式来整理凌乱的提交历史。

在使用`git rebase`命令时添加`-i`命令来启用互动式`rebase`。

```
git checkout feature
git rebase -i master
```

这个命令会打开一个编辑器，并在里面列举出会被合并的所有提交历史：

```
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

这个列表就是`rebase`完成后分支的提交历史最终的样子。你可以更改`pick`命令、调整先后顺序来按照自己的意愿设置提交历史。例如，如果第二个提交只是修复了第一个提交带入的小bug，那么你可以把它们合并成一个命令是`fixup`的提交记录：

```
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

当关闭文件后，Git工具就会按照文本中的指令来执行`rebase`，将提交历史修改为下图中的样子：

![交互式rebase图](https://wac-cdn.atlassian.com/dam/jcr:fe6942b4-7a60-4464-9181-b67e59e50788/04.svg?cdnVersion=ld)

隐藏不重要的提交信息可以使提交历史更加的容易理解，这也是简单的`git merge`无法做到的。

# Rebase黄金法则

对于`rebase`，最重要的事情是知道什么情况下不要去使用它。黄金法则就是，不要在_公共_的分支上使用`git rebase`功能。

比如，试想一下在主干上执行对某分支的`rebase`操作：

![主干上执行rebase](https://wac-cdn.atlassian.com/dam/jcr:1d22f018-b2c7-4096-9db1-c54940cf4f4e/05.svg?cdnVersion=ld)

那么Git会把主干的提交历史修改到分支`feature`的末端之后。问题是这只会在你的代码库中发生。其他的开发者还在原先的主干上进行开发。由于`rebase`会产生全新的提交历史，所以Git会认为你的主干代码与其他人的主干代码都不一致。

要使两边的主干回到一致，唯一的办法就是将它们在合并到一起。这会导致一个新的合并提交记录，和两堆完全一样的提交集合（原始主干的提交历史，和你的代码库主干提交历史）。毫无疑问，这会让人感到非常困扰。

所以，在你执行`git rebase`之前，一定要问自己一句，“还有别人在使用这个分支吗？”如果答案是肯定的，那么立刻双手离开键盘，再想一个没有破坏性的方法来提交你的修改。如果不是，那么就可以安全的重写提交历史了。

## 强行Push

如果你在`rebase`后使用`push`来上传主干代码到远程代码库，那么Git会因为存在冲突而拒绝上传。但是你也可以通过`--force`参数来强行上传:

```
# 这个命令一定要谨慎使用
git push --force
```

这个命令会按照你本地代码库`rebase`后的情况来重写远程代码库的主干。这会使团队的其他成员非常困惑。所以，除非你清楚的了解后果，否则对于这个命令一定要谨慎。

只有少数几种情况应当使用强行`push`，其中一种就是你在将本地分支`push`到远程分支（比如，想要备份代码）后，有对本地分支进行了整理。就好像跟Git说：『噢，其实我并不想`push`之前的那个版本，请使用这个版本代替它吧』。当然，一定要确保没有人正在基于你之前提交的版本在进行开发。

# 工作流的工作模式

根据团队的需要，`rebase`可以适当程度的介入到你们已有的工作流中。这一节就会介绍`rebase`在开发一个新特性的不同环节中提供的各种便利。

首先，必须创建一个用于开发新特性的分支。这样就有了如下的分支结构，才能开始安全的使用`rebase`操作。

![普通分支](https://wac-cdn.atlassian.com/dam/jcr:6af9de07-088b-4f8b-97a7-b66569a9e4ac/06.svg?cdnVersion=ld)

## 本地整理

在工作流中引入`rebase`最佳的方式之一就是整理正在进行中的本地分支。定期进行`rebase`，能使你的分支提交历史看起来更加的专注和清晰。这也让你在写代码时不必顾虑分多次提交导致提交历史破碎的问题——你可以稍后在整理它们。

当你执行`git rebase`的时候，你可以选择要基于的版本：当前分支的父级分支（一般来说是主干）；或者当前分支更早的某个版本。在_交互式`rebase` _中，已经有一个基于主干`rebase`的例子。不过当你只需要修复最近的几次提交历史时，就应该使用第二种方式了。下面就是只对最近3此提交做整理的`rebase`方法：

```
git checkout feature
git rebase -i HEAD~3
```

指定`HEAD~3`作为新的基准版本，实际上并不会造成分支基准版本的移动，它只会重写最近3此提交历史而已。如图所示，这个命令不会导致分支的基准发生变化。

![rebase HEAD](https://wac-cdn.atlassian.com/dam/jcr:079532c4-2594-40ed-a5c4-0e3621b9edff/07.svg?cdnVersion=ld)

如果你想要重写整个分支的提交历史，那么你可以使用`merge-base`命令来找到最初的基准版本，然后将它和`git rebase`一起使用实现重写全部提交历史的目的。

```
git merge-base feature master
```

由于这种方式只会影响到本地分支，因此也是在工作流中引入`rebase`的一种好的方式。对于其他开发者来说，他们能看到的只是一个完整的，带有干净、容易理解的提交历史的产品。

再一次说明，这种方式只适合本地私有分支。如果有其他人在同一分支上进行开发，那这个分支就是公共分支，你不应该重写公共分支的提交历史。

## 将上游变更引入到分支

在_概念概述_中，我们已经看到使用`git merge`和`git rebase`来把主干变更引入到分支的例子。`merge`保留了分支的全部历史，因此更加安全；而`rebase`通过把分支基准移动到主干末端来保持分支提交历史的线性。

此节`git rebase`和本地整理原理相同（并且可以同时进行），只是这里是将上游（主干）的提交引入到分支来。

将非主干分支的变更引入到本分支也是完全允许的。当你和别人在同一个分支上开发，并且你需要引入他提交的变更时就可以使用。

例如，你和你的同事John都在分支`feature`上开发。在John提交一些变更后，你再拉取分支，就会使提交历史变成下图的样子：

![同分支协同开发](https://wac-cdn.atlassian.com/dam/jcr:0bb661aa-361d-47ba-8c7b-00b3be0546cb/08.svg?cdnVersion=ld)

你可以同合并主干同样的办法来处理这个分支：要么使用`merge`来合并，要么使用`rebase`来把基准移到John的提交版本之后。

![同分支协同开发-merge](https://wac-cdn.atlassian.com/dam/jcr:1896adb1-5d49-419a-9b50-3a36adac186c/09.svg?cdnVersion=ld)

![同分支协同开发-rebase](https://wac-cdn.atlassian.com/dam/jcr:1896adb1-5d49-419a-9b50-3a36adac186c/09.svg?cdnVersion=ld)

注意，这里`rebase`方法并没有违反_rebase的黄金法则_，你只是对本地分支的历史做了修改，其他的都没有任何变化。这就像对Git说：『把我的更改放到John完成的工作后面』一样。在大多数情况下，这都要比合并提交要更加的直观。

默认情况下，使用`git pull`会自动使用`merge`来合并远程分支代码，但是你也可以使用`--rebase`参数来强制使用`rebase`方法合并。

## 使用PullRequest(RP)来检查分支

如果你在工作流中使用RP来检查分支，那么在提交PR后就不可以再使用`rebase`改写提交历史。一旦提交RP后，别人就会来检查分支上的提交历史，那么这个分支就变成了『公共分支』。此时重写提交历史会导致Git和团队伙伴无法对接到后续的提交。

其他开发者的变更也必须使用`git merge`而不是`git rebase`合入进来。

因此，最好在提交RP之前就使用`rebase`整理好分支的提交历史，而不是在提交RP之后。

## 合并一个过审的分支

在一个分支通过了团队评审后，你就可以使用`git rebase`将它的基准移到主干末端，然后使用`git merge`来把分支合并到主干中。

这本质上与引入分支变更到主干相同，但是主干是不允许重写提交历史的，所以最终只能用`git merge`来将分支代码合并到主干中。然而，只要在分支上执行`rebase`后，能尽快的在主干上执行`merge`，那么产生的提交历史仍然是线性的。这样你也可以把PR期间的提交塞到提交历史中。

![使用/不使用Rebase将分支合入主干的情况](https://wac-cdn.atlassian.com/dam/jcr:df39b1f1-2686-4ee5-90bf-9836783342ce/10.svg?cdnVersion=ld)

如果对于`git rebase`还不那么熟练，你也可以在一个临时分支里来尝试这个操作。这样，即使你不小心把提交历史弄乱了，你也还是可以恢复到最初的分支状态。比如：

```
git checkout feature
git checkout -b temporary-branch
git rebase -i master
# [Clean up the history]
git checkout master
git merge temporary-branch
```

# 总结

以上就是`rebase`所需要的基本知识了。如果想要一个干净、线性的、没有合并提交记录的提交历史，那么就应该使用`git rebase`代替`git merge`来处理分支之间代码合并的工作。

另外一方面，如果要保留全部的提交历史，规避重写公共提交历史的风险，那么仍然需要使用`git merge`。两种方法都是有效的，不过现在至少多了一个`git rebase`多选择。
