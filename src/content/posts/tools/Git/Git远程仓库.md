---
title: Git(4)·Git远程仓库
published: 2023-05-25
description: Git基本用法，关联远程仓库(github, gitee).
tags: [tools, Git]
category: tools
draft: false
---

让Git突破本地版本管理器，链接上远程仓库，比如GitHub，Gitlab等。  
  
## 获取ssh密钥  
```  
ssh-keygen -t rsa -C "youremail@example.com"  
```  
其中，邮箱是后面注册GitHub等一直要用的。  
创建成功后在用户目录下出现.ssh/，里面的 *id_rsa* 是密钥，*id_rsa.pub* 是公钥，这个公钥将用来绑定到github等网站。  
里面还有一个 *known_hosts* ，大概是已知的访问，是别人的公钥（大概），clone了别人的仓库后会也许会出现。  
  
有了公钥后再GitHub等的账户配置里绑定ssh公钥。  
  
## 添加远程仓库  
```  
git remote add (shortname) [url]  
```  
shortname 将是你远程仓库的识别名(一个别名，它和你在GitHub等看到的仓库名不一样）。GitHub的url 似乎是git@github.com:xxxxxxxx...  
```  
好像一个ssh绑定多个repository,一个repository中可以有很多“项目”，是这样吗...就像一个本地仓库可以有很多.git项目...  
这个项目add到哪个repository里是在url链接后面控制的。  
```  
可以在GitHub上新建 new repository，新建的new repository似乎有默认的识别名，会在建立成功后给出，比如origin。  
*此后push, pull, fetch等操作都和这个识别名有关*，只通过这个识别名识别到你的远程仓库。  
  
**查看远程仓库**：`git remove (-v)， -v显示详细仓库的链接`  
  
## push, fetch  
```  
git fetch origin master  
```  
从识别名为 origin 的仓库上获取 master 分支到本地分支*origin/master* 中。可以用来获取远程仓库中的更新。此后需要用`git merge origin/master`将获得的分支合并到自己的本地分支中。  
可以有多个分支名（master后面，为origin的分支），一次获取多个分支.  
  
```  
git push origin master  
```  
将本地的 master 分支推送到远程识别名为 origin 的仓库中。往往在本地做完所有修改后，统一推送。  
  
## 删除远程仓库  
```  
git remote rm (shortname)  
```  
