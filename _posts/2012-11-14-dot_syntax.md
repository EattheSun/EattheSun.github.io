---
title: 使用DOT语法画状态转移图
layout: post
---

依赖包：graphviz

编辑一个dot文件pt_notify.dot：

    digraph G{
        NP_NOT_EXIST [shape=box];
        NP_NOT_EXIST->NP_PENDING_1 [label="P"];
        NP_NOT_EXIST->NP_NOT_EXIST [label="R"];
        NP_PENDING_1->NP_PENDING_1 [label="xn P"];
        NP_PENDING_1->NP_NEED_NOTIFY_1 [label="P"];
        NP_NEED_NOTIFY_1 [style=dotted];
        NP_PENDING_1->NP_CLOSED [label="R"];
        NP_CLOSED [shape=box];
        NP_NEED_NOTIFY_1->NP_NOTIFIED_1 [label=" [problem notify sent] "];
        NP_NOTIFIED_1->NP_PENDING_2 [label="P"];
        NP_NOTIFIED_1->NR_NEED_NOTIFY [label="R"];
        NP_PENDING_2->NP_PENDING_2 [label="xm P"];
        NP_PENDING_2->NP_NEED_NOTIFY_2 [label="P"];
        NP_NEED_NOTIFY_2 [style=dotted];
        NP_PENDING_2->NR_NEED_NOTIFY [label="R"];
        NP_NEED_NOTIFY_2->NP_NOTIFIED_2 [label=" [problem notify sent] "];
        NP_NOTIFIED_2->NP_PENDING_2 [label="P"];
        NP_NOTIFIED_2->NR_NEED_NOTIFY [label="R"]
        NR_NEED_NOTIFY->NP_PENDING_2 [label="P"];
        NR_NEED_NOTIFY->NR_NEED_NOTIFY [label="xk R"];
        NR_NEED_NOTIFY->NR_CLOSED [label=" [recovery notify send] "];
        NR_NEED_NOTIFY [style=dotted];
        NR_CLOSED [shape=box];
    }

敲下命令： **dot -Tpng pt_notify.dot -o pt_notify.png**

![dotpic](http://i.imgur.com/mddnql.png)

更多语法： [wikipedia](http://zh.wikipedia.org/zh-cn/DOT%E8%AF%AD%E8%A8%80)
