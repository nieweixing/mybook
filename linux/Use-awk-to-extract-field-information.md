日常工作中有的时候我们需要对我们的文本进行处理，比如获取某几列的内容，再列之间加入分割符，这里不得不用到awk这个工具了，awk是一种处理文本文件的语言，是一个强大的文本分析工具，下面我们来讲讲如何使用它

我们以下面这个region.txt文本来作为示例

```
[root@VM-0-3-centos ~]# cat region.txt
地域    名称    简写（不推荐）        是否全量
ap-beijing      华北地区(北京)  bj      是
ap-chengdu      西南地区(成都)  cd      是
ap-chongqing    西南地区(重庆)  cq      是
ap-guangzhou    华南地区(广州)  gz      是
```

# 获取文本的某几列

获取文本的地域和简写

```
[root@VM-0-3-centos ~]# cat region.txt | awk -F " " '{print $1,$3}'
地域 简写（不推荐）
ap-beijing bj
ap-chengdu cd
ap-chongqing cq
ap-guangzhou gz
```

# 输入文本指定分隔符

输入文本字段之间用|||分割

```
[root@VM-0-3-centos ~]# cat region.txt | awk -F " " -v OFS='|||' '{print $1,$3}'
地域|||简写（不推荐）
ap-beijing|||bj
ap-chengdu|||cd
ap-chongqing|||cq
ap-guangzhou|||gz
```

# 参考文档

awk还有很多其他用法，具体可以参考文档

<https://www.runoob.com/linux/linux-comm-awk.html>