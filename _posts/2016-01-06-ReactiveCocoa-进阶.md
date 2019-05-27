---
layout:     post
title:      babel-polyfill的一个坑
subtitle:   babel-polyfill
date:       2018-01-06
author:     BY
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - 前端
    - js
    - babel
    - babel-polyfill
---
# 前言
接手的系统几天前出现了兼容性问题，`Array.flat is not a function`,一看原来是没有引用babel-polyfill，直接import了babel-polyfill，发现并没有解决问题。

# 