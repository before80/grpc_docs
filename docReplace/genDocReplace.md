+++
title = "生成文档前的替换"
date = 2023-06-05T09:37:31+08:00
description = ""
isCJKLanguage = true
draft = false

+++

# 生成文档前的替换

```
{{< ref "">}}


```



## 替换掉 `.svg+xml` 符

```
// 查找匹配如下字符
.svg+xml

// 替换成
.svg
```



## 使用rsm程序替换掉目录下以`.svg+xml`结尾的文件



## 替换掉youbute的iframe

```
// 查找匹配如下正则表达式
<iframe src="https:\/\/www\.youtube\.com\/embed\/([A-Za-z0-9_\-]+)" [^>\n]+><\/iframe>

// 替换成
{{< youtube "$1">}}
```

