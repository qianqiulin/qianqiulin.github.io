---
layout:     post
title:      "Fork别人的代码 原作者更新后如何同步"
subtitle:   "github fork sync"
date:       2017-09-12 15:26:30
author:     "TyurinTsien"
header-img: "img/post-bg-github.jpg"
catalog: true
tags:
    - Github
---

## Fork别人的代码 原作者更新后如何同步
    在github上fork了别人的代码，当需要同步原作者的代码的时候，可以用以下方法

### 给主题的fork加一个remote
+ 使用 git remote -v 查看远程状态
```
bogon:lanproxy TyurinTsien$ git remote -v
origin	https://github.com/qianqiulin/lanproxy.git (fetch)
origin	https://github.com/qianqiulin/lanproxy.git (push)
```

+ 把原作者的远程仓库添加到remote
```
git remote add upstream https://github.com/ffay/lanproxy.git
```

+ 再次查看是否添加成功
```
bogon:lanproxy TyurinTsien$ git remote -v
origin	https://github.com/qianqiulin/lanproxy.git (fetch)
origin	https://github.com/qianqiulin/lanproxy.git (push)
upstream	https://github.com/ffay/lanproxy.git (fetch)
upstream	https://github.com/ffay/lanproxy.git (push)
```

### 同步Fork
+ 从上游仓库 fetch 分支和提交点，传送到本地，并会被存储在一个本地分支 upstream/master
```
git fetch upstream
```

+ 切换到本地主分支(防止出错)
```
bogon:lanproxy TyurinTsien$ git checkout master
Already on 'master'
Your branch is up-to-date with 'origin/master'.
```
+ 把 upstream/master 分支合并到本地 master 上，这样就完成了同步，并且不会丢掉本地修改的内容。
```
git merge upstream/master
```

### 如果没有需要手动合并的冲突就，直接 git push origin master。直接更新到github上面的fork即可，如果出现需要手动合并的冲突