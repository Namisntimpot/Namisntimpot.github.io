---
title: Git报错·detached HEAD
published: 2023-05-25
description: Git detached Head报错.
tags: [tools, Git]
category: tools
draft: false
---

如果一个HEAD指针并没有指向完整的代码仓库，而只指向了最后一条*commit*，那么这个HEAD会是*detached* 状态。正常情况下HEAD应该指向完整的目录树，*detached* 状态时候HEAD只指向最后一次*commit*，两者不同。  后者你看不到其它任何提交。  
在分离状态下，你仍然可以像平时那样add与commit，但是这都只是*试验性*的，当你离开这个*detached*状态的分支时，这些提交就成了垃圾。  
  
一般情况下，这是因为你试图跳转到一个本地不存在、但在远程仓库中可能存在的分支。在没有拉取下载远程仓库中分支时你无法在本地操作它，因而会是一个detached态。  
比如，我遇到的情况是，我现在在本地 *origin* 仓库的 *make-exercise* 仓库下，拉取远程仓库获取了新分支 `origin/mips-exercise`，如果我`git checkout origin/mips-exercise` 就会出现 *detached* 状态，因为这样搜索分支搜的是“绝对路径”`origin/origin/mips-exercise`，本地不存在，但是远程仓库中恰好有一个 `origin/mips-exercise`。  
用 `git checkout mips-exercise` 就行了。  
  
此外，如果你想对这个东西做出*有效修改*，可以以当前状态为基建立一个新的*分支*，即在这个孤零零的 commit 上建立一个目录树，这个 commit 就是分支中的第一个 commit。用 `git switch -c name`，然后你就可以在这个新分支里对它进行有效操作了。  
