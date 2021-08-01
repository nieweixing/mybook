# 问题现象

tke集群的pod访问某个外部域名发现很慢，超5s以上，访问其他域名不会有问题，在节点访问是正常，而且不是所有的容器都有问题，这里到底是怎么回事呢？


# 排查思路

首先只在某个pod出现这个问题，说明不是网络或者dns有问题，后面发现出现问题的都是alphie系统，为什么alphine会出现这个问题，这里我分别在centos和alpine抓包，看下导致耗时在哪里

# 抓包测试

首先我们在alpine镜像中测试访问域名，然后抓包

```
bash-4.4# time curl -w "%{time_namelookup}" --location --request POST 'https://elink.spic.com.cn/'
{"errcode":40014,"errmsg":"invalid access_token [logid:]"}5.073069
real    0m5.278s
user    0m0.017s
sys     0m0.003s

bash-4.4# tcpdump -i any port 53
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
09:13:49.074130 IP go-test-5f78b5569f-r2q9v.59071 > kube-dns.kube-system.svc.cluster.local.53: 47843+ A? elink.spic.com.cn.tke-test.svc.cluster.local. (62)
09:13:49.074164 IP go-test-5f78b5569f-r2q9v.59071 > kube-dns.kube-system.svc.cluster.local.53: 48334+ AAAA? elink.spic.com.cn.tke-test.svc.cluster.local. (62)
09:13:49.074257 IP go-test-5f78b5569f-r2q9v.44323 > kube-dns.kube-system.svc.cluster.local.53: 52788+ PTR? 140.52.16.172.in-addr.arpa. (44)
09:13:49.074596 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.59071: 47843 NXDomain*- 0/1/0 (155)
09:13:49.074660 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.59071: 48334 NXDomain*- 0/1/0 (155)
09:13:49.074710 IP go-test-5f78b5569f-r2q9v.42646 > kube-dns.kube-system.svc.cluster.local.53: 51281+ A? elink.spic.com.cn.svc.cluster.local. (53)
09:13:49.074744 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.44323: 52788*- 1/0/0 PTR kube-dns.kube-system.svc.cluster.local. (122)
09:13:49.075046 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.42646: 51682 NXDomain*- 0/1/0 (146)
09:13:49.075115 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.42646: 51281 NXDomain*- 0/1/0 (146)
09:13:49.075191 IP go-test-5f78b5569f-r2q9v.35865 > kube-dns.kube-system.svc.cluster.local.53: 61054+ A? elink.spic.com.cn.cluster.local. (49)
09:13:49.075242 IP go-test-5f78b5569f-r2q9v.35865 > kube-dns.kube-system.svc.cluster.local.53: 61474+ AAAA? elink.spic.com.cn.cluster.local. (49)
09:13:49.075419 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.35865: 61054 NXDomain*- 0/1/0 (142)
09:13:49.075477 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.35865: 61474 NXDomain*- 0/1/0 (142)
09:13:49.075535 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26687+ A? elink.spic.com.cn. (35)
09:13:49.075607 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:49.085340 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26687 1/0/0 A 39.155.244.138 (68)
09:13:50.761856 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail 0/0/0 (35)
09:13:50.761896 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:50.762014 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:50.762043 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:50.762114 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:50.762149 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:50.762247 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:50.762280 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:50.762341 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:51.576211 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:51.576403 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:51.576442 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:51.576660 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:51.576687 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:51.576851 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:51.576886 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:51.577039 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
09:13:51.577073 IP go-test-5f78b5569f-r2q9v.56020 > kube-dns.kube-system.svc.cluster.local.53: 26917+ AAAA? elink.spic.com.cn. (35)
09:13:51.577153 IP kube-dns.kube-system.svc.cluster.local.53 > go-test-5f78b5569f-r2q9v.56020: 26917 ServFail* 0/0/0 (35)
```

从上面发现，在alphine镜像会多次解析AAAA记录，造成netfilter race，详细的分析可以参考这个https://blog.quentin-machu.fr/2018/06/24/5-15s-dns-lookups-on-kubernetes/


我们又在对应的centos上测试了下，发现每次轮询dns server只会出现一个AAAA记录，时间比起alpine会短很多

```
[root@centos-684c48fccd-k24tp /]# time curl -w "%{time_namelookup}" --location --request POST 'https://elink.spic.com.cn/'
{"errcode":40014,"errmsg":"invalid access_token [logid:]"}1.510
real    0m1.836s
user    0m0.035s
sys     0m0.075s

09:17:13.970833 IP centos-684c48fccd-k24tp.41304 > kube-dns.kube-system.svc.cluster.local.domain: 7375+ A? elink.spic.com.cn.tke-test.svc.cluster.local. (62)
09:17:13.970870 IP centos-684c48fccd-k24tp.41304 > kube-dns.kube-system.svc.cluster.local.domain: 38125+ AAAA? elink.spic.com.cn.tke-test.svc.cluster.local. (62)
09:17:13.971520 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.41304: 38125 NXDomain*- 0/1/0 (155)
09:17:13.971567 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.41304: 7375 NXDomain*- 0/1/0 (155)
09:17:13.971625 IP centos-684c48fccd-k24tp.45222 > kube-dns.kube-system.svc.cluster.local.domain: 65110+ A? elink.spic.com.cn.svc.cluster.local. (53)
09:17:13.971669 IP centos-684c48fccd-k24tp.45222 > kube-dns.kube-system.svc.cluster.local.domain: 3169+ AAAA? elink.spic.com.cn.svc.cluster.local. (53)
09:17:13.971996 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.45222: 65110 NXDomain*- 0/1/0 (146)
09:17:13.972018 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.45222: 3169 NXDomain*- 0/1/0 (146)
09:17:13.972044 IP centos-684c48fccd-k24tp.36358 > kube-dns.kube-system.svc.cluster.local.domain: 23192+ A? elink.spic.com.cn.cluster.local. (49)
09:17:13.972069 IP centos-684c48fccd-k24tp.36358 > kube-dns.kube-system.svc.cluster.local.domain: 34973+ AAAA? elink.spic.com.cn.cluster.local. (49)
09:17:13.972427 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.36358: 23192 NXDomain*- 0/1/0 (142)
09:17:13.972658 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.36358: 34973 NXDomain*- 0/1/0 (142)
09:17:13.972698 IP centos-684c48fccd-k24tp.36416 > kube-dns.kube-system.svc.cluster.local.domain: 11372+ A? elink.spic.com.cn. (35)
09:17:13.972728 IP centos-684c48fccd-k24tp.36416 > kube-dns.kube-system.svc.cluster.local.domain: 3185+ AAAA? elink.spic.com.cn. (35)
09:17:13.973772 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.36416: 11372 1/0/0 A 39.155.244.138 (68)
09:17:15.218386 IP kube-dns.kube-system.svc.cluster.local.domain > centos-684c48fccd-k24tp.36416: 3185 ServFail 0/0/0 (35)
```

这边用优化后的alphine镜像geekidea/alpine-a:3.7，该镜像移除了AAAA记录，然后测试发现没有AAAA解析记录后会非常快

```
/ # time curl -w "%{time_namelookup}" --location --request POST 'https://elink.spic.com.cn/'
{"errcode":40014,"errmsg":"invalid access_token [logid:]"}0.003296
real    0m 0.17s
user    0m 0.01s
sys     0m 0.00s


09:12:07.919297 IP alpine-a-5684fcc8c7-mnfhp.37258 > kube-dns.kube-system.svc.cluster.local.53: 11480+ A? elink.spic.com.cn.tke-test.svc.cluster.local. (62)
09:12:07.919630 IP kube-dns.kube-system.svc.cluster.local.53 > alpine-a-5684fcc8c7-mnfhp.37258: 11480 NXDomain*- 0/1/0 (155)
09:12:07.919686 IP alpine-a-5684fcc8c7-mnfhp.59507 > kube-dns.kube-system.svc.cluster.local.53: 18200+ A? elink.spic.com.cn.svc.cluster.local. (53)
09:12:07.920076 IP kube-dns.kube-system.svc.cluster.local.53 > alpine-a-5684fcc8c7-mnfhp.59507: 18200 NXDomain*- 0/1/0 (146)
09:12:07.920119 IP alpine-a-5684fcc8c7-mnfhp.33037 > kube-dns.kube-system.svc.cluster.local.53: 63110+ A? elink.spic.com.cn.cluster.local. (49)
09:12:07.920426 IP kube-dns.kube-system.svc.cluster.local.53 > alpine-a-5684fcc8c7-mnfhp.33037: 63110 NXDomain*- 0/1/0 (142)
09:12:07.920508 IP alpine-a-5684fcc8c7-mnfhp.58282 > kube-dns.kube-system.svc.cluster.local.53: 51705+ A? elink.spic.com.cn. (35)
09:12:07.921322 IP kube-dns.kube-system.svc.cluster.local.53 > alpine-a-5684fcc8c7-mnfhp.58282: 51705 1/0/0 A 106.38.29.46 (68)
```

# 结论

这里在容器内解析域名需要5s以上是因为基础镜像的操作系统导致，主要看镜像内对AAAA记录解析次数，像alpine默认会对域名解析多次AAAA记录。



# 参考文档

<https://openforum.hand-china.com/t/topic/1111>
