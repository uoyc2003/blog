---
title: Hexo博客标题和表格同行情况解决
tags:
  - Hexo
categories:
  - 问题日志
headimg: https://pic.zeng.cyou/wide/202409042025524.webp
abbrlink: 32918
date: 2024-09-09 22:08:58
---

## 问题描述

主题：Volantis

在写文章的时候，如果`h2/h3`标题下没有其他内容，而是直接放一个表格上去的话，会出现下面这种情况。

![202409092235543](https://pic.zeng.cyou/post/32918/202409092235543.webp)

## 解决思路

获取文章里所有的`h2/h3`标签，然后判断下一个元素是否是表格，如果是的话，就加一个`p`标签。

在主题文件`volantis/source/js/`的目录下创建一个文件夹用于存放自定义的`js`文件。

```js
let postBody = document.getElementById("post-body");

let titles = postBody.querySelectorAll("h2, h3");

for (let i = 0; i < titles.length; i++) {
    let next = titles[i].nextElementSibling;
    if (next && next.tagName === "TABLE") {
        const p = document.createElement("p");
        postBody.insertBefore(p, next);
    }
}
```

之后在`layout/layout.ejs`文件里导入这个`js`文件。

```js
<script type="text/javascript" src="/js/modify/custom.js"></script>
```

![202409092235544](https://pic.zeng.cyou/post/32918/202409092235544.webp)
