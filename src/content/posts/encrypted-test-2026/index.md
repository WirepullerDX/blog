---
title: "加密文章测试：输入密码才能看"
published: 2026-10-29
updated: 2026-10-29
description: "用于验证文章密码、密码提示、错误密码提示，以及解密后图片和脚本重新初始化的测试文章。"
image: "https://img.rxcdjg.dpdns.org/gallery/r2-test/photo-02.webp"
tags: ["测试", "密码", "加密"]
category: "站点维护"
draft: false
pinned: false
author: "阿杰"
comment: true
password: "jiege2026"
passwordHint: "杰哥的拼音 + 当前年份（全小写）"
series: "博客搭建"
seriesOrder: 2
---

## 解密成功

如果你能看到这一段，说明密码正确，正文已经由浏览器在本地解密。

这篇文章用于验证以下流程：

- 未输入密码时只显示密码输入框
- 密码提示能正常显示
- 输入错误密码时显示错误提示
- 输入正确密码后显示正文
- 刷新页面后，在当前浏览器会话内可以免输密码
- 解密后的 R2 图片和交互脚本能重新初始化
- 加密文章不显示评论区

## 解密后的图片

下面这张图片来自 R2，用来检查解密后图片是否正常加载：

![解密后加载的 R2 测试图片](https://img.rxcdjg.dpdns.org/gallery/r2-test/photo-02.webp)

## 测试结论记录

如果以上内容全部正常，说明文章级密码保护的前端流程可用。

> 这只是访问门槛，不是强加密保险箱。密码写在公开仓库的 Markdown 源码里，不要用它保存真正敏感的信息。
