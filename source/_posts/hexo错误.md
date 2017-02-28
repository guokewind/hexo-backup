---
date: 2017年02月13日 23:42:16
title: hexo命令报错
tags:
    hexo
---
```
YAMLException: end of the stream or a document separator is expected at line 26, column 1:
```

报错的行根本没有错。用排除法逐一删掉，再hexo g，结果发现是

作横线的 "----------" 后面少了一个空格的原因,改成"---------- "就好了。
