---
title: Github 删除某个 commit 记录
date: 2023-09-07 18:19:18
tags:
 - Github

categories:
 - 日常学习
---
## 前言
在提交 git 的时候，有时候会手快打错备注信息，或者有的时候不想被人发现自己提交了什么奇奇怪怪的东西，但是又不能影响到之后提交的代码，那么可以用下面的这种方法。

## 具体操作
假设现在有三个提交记录：commit_A，commit_B，commit_C，你现在想删掉中间的提交记录 commit_B，那么可以用下面的命令：

```
git rebase -i (commit-id) 
```

`commit-id` 是想要删除的那个 commit 记录之前的那一条，而这个 ID 要取提交记录的哈希字符串。

假设三个提交记录分别如下：

```
commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:31 2008 -0700

    第三次提交：commit_C

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:21 2008 -0700

    第二次提交：commit_B

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    第一次提交：commit_A
```

那就输入命令：

```
git rebase -i ca82a6dff817ec66f44342007202690a93763949
```

接着就会看到进入文本编辑界面，类似如下：

```
pick a11bef0 commit_C
pick 085bb3b commit_B
pick ca82a6d commit_A
```

然后把 commit_B 前面的 pick 改成 drop 就可以了：

```
pick a11bef0 commit_C
drop 085bb3b commit_B
pick ca82a6d commit_A
```

接着与 vim 一样保存退出，然后就会看到提示：

```
Successfully rebased and updated refs/heads/master.
```

最后把这个提交结果推送到远程仓库：

```
git push origin HEAD --force
```

这样 commit_B 就被干掉了，到网页端的仓库提交历史记录检查一下，已经看不到这条记录了。