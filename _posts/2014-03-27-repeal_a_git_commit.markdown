---
layout: post
title: github 撤销一个已经提交的 commit
tags:
    - git
---

# 事发有因

不小心将自己的公司邮箱密码和某重要密码 push 到了github上，虽说是 private repo，但有权限的人还是能看到。

# 怎么撤销?

* step1. 查看需要回退到的 commit id


    $ git log


* step2. 在本地回退到有问题的 commit 之前的版本


    $ git reset --hard [commit-id]


* step3. 将改动推到远程分支，注意一定要加 `--force` 选项，否则会 push 失败。


    $ git push origin [branch-name] --force


# 每文一图

![pic](http://wallpaper.u.qiniudn.com/amazingmilkywayiv.jpg)
