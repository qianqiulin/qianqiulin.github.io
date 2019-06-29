---
layout:     post
title:      "OS X修改环境变量"
subtitle:   " mac os 环境变量"
date:       2017-07-10 14:21:28
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - MAC
---


## [Mac] OS X修改环境变量

+ TyurinTsien
+ 2017年07月10日14:21:28

### OS X系统的环境变量，加载顺序为：

```markdown
/etc/profile
/etc/paths 
~/.bash_profile 
~/.bash_login 
~/.profile 
~/.bashrc
```

/etc/profile和/etc/paths是系统级别的，系统启动就会加载，
后面几个是当前用户级的环境变量。

~/.bash_profile，~/.bash_login，~/.profile按照从前往后的顺序读取，
如果~/.bash_profile文件存在，则后面的几个文件就会被忽略不读了，
如果~/.bash_profile文件不存在，才会以此类推读取后面的文件。

~/.bashrc没有上述规则，它是bash shell打开的时候载入的。

### 设置PATH的语法为：

```markdown
export PATH="$PATH:<PATH 1>:<PATH 2>:<PATH 3>:...:<PATH N>"
```

注：

1. 一般环境变量更改后，重启后才可生效。如果想立刻生效，则可执行下面的语句：

    ```markdown
    $ source 相应的文件
    ```

2. 如果默认shell是bash，那么shell启动时会触发.bashrc，如果默认shell是zsh，那么shell启动时会触发.zshrc

3. 环境变量既可以加到$PATH头部，也可以加到$PATH尾部。
例如mac中自带emacs的位置在/usr/local/emacs

    ```markdown
    $ type emacs
    emacs is /usr/local/emacs
    ```

如果在.zshrc中添加export PATH="$PATH:/Applications/Emacs.app/Contents/MacOS"
且 source .zshrc之后，type emacs还是emacs is /usr/local/emacs

解决方案是，把路径加在$PATH头部，

```markdown
export PATH="/Applications/Emacs.app/Contents/MacOS:$PATH"
```

或者增加alias，

```markdown
alias emacs="/Applications/Emacs.app/Contents/MacOS/Emacs"
```