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
