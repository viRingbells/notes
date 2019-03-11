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

## Rebase黄金法则

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

（待续……）
