我们讲下如何在gitbook中添加笔记，gitbook初始化之后默认会创建SUMMARY.md和README.md这2个文件

README.md里面的内容是你这本书的简介部分

```
# 简介

本书主要介绍了一些k8s的相关操作,运维知识的学习和讲解，test
```

SUMMARY.md文件主要是用来添加文章目录

```
# Summary

- [简介](README.md)

### 如何搭建属于自己的gitbook



### linux运维笔记

- [腾讯云cvm上搭建openvpn](linux/2021-03-25-Build-openvpn-intranet-access-vpc-on-cvm.md)

### docker运维笔记

- [1](docker/1.md)

### kubernetes运维笔记

- [强制删除Terminating状态ns](k8s/强制删除Terminating状态ns.md)


### TKE运维笔记

- [1](tke/1.md)
- [2](tke/2.md)
```

如果你需要添加笔记，可以先创建文件夹，存放你编写的markdown笔记，然后再SUMMARY.md配置上目录即可。