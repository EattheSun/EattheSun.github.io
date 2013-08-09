---
layout: post
title: 利用 html meta 属性
---

# 如何利用html文件进行跳转

    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <meta http-equiv="Content-Language" content="zh-CN">
            <meta http-equiv="refresh" content="0.1;url=http://docs.qiniu.com/java-sdk/v6/index.html">
        </head>
        <body></body>
    </html>

# 如何使用html禁止cache

pragma与no-cache 属性值 -- 定义页面缓存
pragma与no-cache用于定义页面缓存
pragma出现在http-equiv属性中，使用content属性的no-cache值表示是否缓存网页

    <meta http-equiv="pragma" content="no-cache" />
