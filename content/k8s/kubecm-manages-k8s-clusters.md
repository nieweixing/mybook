kubecm是一个k8s集群管理工具，可以合并多个kubeconfig文件，切换集群等

# 安装kubecm

```
curl -Lo kubecm.tar.gz https://github.com/sunny0826/kubecm/releases/download/v0.15.3/kubecm_0.15.3_Linux_x86_64.tar.gz
tar -zxvf kubecm.tar.gz kubecm
cd kubecm
sudo mv kubecm /usr/local/bin/
```

设置自动补全

```
$ source <(kubecm completion bash)
$ echo "source <(kubecm completion bash)" >> ~/.bashrc
$ source  ~/.bashrc
```

# kubecm的使用

执行kubecm，如果显示下面内容说明安装成功

```
[root@VM-0-13-centos ~]# kubecm


        Manage your kubeconfig more easily.


██   ██ ██    ██ ██████  ███████  ██████ ███    ███
██  ██  ██    ██ ██   ██ ██      ██      ████  ████
█████   ██    ██ ██████  █████   ██      ██ ████ ██
██  ██  ██    ██ ██   ██ ██      ██      ██  ██  ██
██   ██  ██████  ██████  ███████  ██████ ██      ██

 Tips  Find more information at: https://kubecm.cloud

Usage:
  kubecm [command]

Available Commands:
  add         Add KubeConfig to $HOME/.kube/config
  alias       Generate alias for all contexts
  clear       Clear lapsed context, cluster and user
  completion  Generates bash/zsh completion scripts
  create      Create new KubeConfig(experiment)
  delete      Delete the specified context from the kubeconfig
  help        Help about any command
  list        List KubeConfig
  merge       Merge the KubeConfig files in the specified directory
  namespace   Switch or change namespace interactively
  rename      Rename the contexts of kubeconfig
  switch      Switch Kube Context interactively
  version     Print version info

Flags:
      --config string   path of kubeconfig (default "/root/.kube/config")
  -h, --help            help for kubecm

Use "kubecm [command] --help" for more information about a command.
```

## 添加集群

这里准备了3个集群的kubeconfig文件

```
[root@VM-0-13-centos .kube]# ll | grep config
-rw-r--r-- 1 root root  1823 Jun  2 14:25 eks.config
-rw-r--r-- 1 root root  5545 Jun  2 15:12 test.config
-rw-r--r-- 1 root root  5541 Jun  2 14:54 tke.config
```

这里我们先创建一个config文件，用来生成合并后的kubeconfig

```
[root@VM-0-13-centos .kube]# touch config
[root@VM-0-13-centos .kube]# kubecm add -f tke.config
Add Context: tke
👻 True
「tke.config」 write successful!
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|   CURRENT  |   NAME  |        CLUSTER        |        USER        |               SERVER              |   Namespace  |
+============+=========+=======================+====================+===================================+==============+
|      *     |   tke   |   cluster-k4m2g9mf44  |   user-k4m2g9mf44  |   https://cls-xxxxxxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+

[root@VM-0-13-centos .kube]# kubecm add -f eks.config
Add Context: eks
👻 True
「eks.config」 write successful!
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|   CURRENT  |   NAME  |        CLUSTER        |        USER        |               SERVER              |   Namespace  |
+============+=========+=======================+====================+===================================+==============+
|            |   eks   |   cluster-6t8847hhfb  |   user-6t8847hhfb  |    https://xx.xx.xx.xx:443/       |    default   |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|      *     |   tke   |   cluster-k4m2g9mf44  |   user-k4m2g9mf44  |   https://cls-xxxxxxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+

[root@VM-0-13-centos .kube]# kubecm add -f test.config
Add Context: test
👻 True
「test.config」 write successful!
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|   CURRENT  |   NAME  |        CLUSTER        |        USER        |               SERVER              |   Namespace  |
+============+=========+=======================+====================+===================================+==============+
|            |   eks   |   cluster-6t8847hhfb  |   user-6t8847hhfb  |    https://xx.xx.xx.xx:443/       |    default   |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|            |   test  |   cluster-9f86dg8h88  |   user-9f86dg8h88  |   https://cls-xxxxxxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|      *     |   tke   |   cluster-k4m2g9mf44  |   user-k4m2g9mf44  |   https://cls-xxxxxxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+

```

## kubecm查看集群


```
[root@VM-0-13-centos .kube]# kubecm ls
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|   CURRENT  |   NAME  |        CLUSTER        |        USER        |               SERVER              |   Namespace  |
+============+=========+=======================+====================+===================================+==============+
|            |   eks   |   cluster-6t8847hhfb  |   user-6t8847hhfb  |    https://xx.xx.xx.xx:443/    xxx|    default   |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|      *     |   tke   |   cluster-k4m2g9mf44  |   user-k4m2g9mf44  |   https://cls-xxxxxxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+

Cluster check succeeded!
Kubernetes version v1.18.4-tke.6
Kubernetes master is running at https://cls-xxxxx.ccs.tencent-cloud.com
[Summary] Namespace: 63 Node: 4 Pod: 155

```


## kubecm删除集群

```
[root@VM-0-13-centos .kube]# kubecm delete test
Context Delete:「test」
「/root/.kube/config」 write successful!
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|   CURRENT  |   NAME  |        CLUSTER        |        USER        |               SERVER              |   Namespace  |
+============+=========+=======================+====================+===================================+==============+
|            |   eks   |   cluster-6t8847hhfb  |   user-6t8847hhfb  |    https://xx.xx.xx.xx:443/    |    default   |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|      *     |   tke   |   cluster-k4m2g9mf44  |   user-k4m2g9mf44  |   https://cls-xxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+

```

这里我们删除了test集群

## kubecm切换集群

```
[root@VM-0-13-centos .kube]# kubectl  get node
NAME         STATUS   ROLES    AGE    VERSION
10.0.0.10    Ready    <none>   61d    v1.18.4-tke.8
10.0.0.157   Ready    <none>   147d   v1.18.4-tke.6
10.0.0.2     Ready    <none>   56d    v1.18.4-tke.8
10.0.0.3     Ready    <none>   189d   v1.18.4-tke.8
[root@VM-0-13-centos .kube]# kubecm switch eks
「/root/.kube/config」 write successful!
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|   CURRENT  |   NAME  |        CLUSTER        |        USER        |               SERVER              |   Namespace  |
+============+=========+=======================+====================+===================================+==============+
|      *     |   eks   |   cluster-6t8847hhfb  |   user-6t8847hhfb  |    https://xx.xx.xx.xx:443/    |    default   |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+
|            |   tke   |   cluster-k4m2g9mf44  |   user-k4m2g9mf44  |   https://cls-xxxxxxxx.ccs.tence  |    default   |
|            |         |                       |                    |            nt-cloud.com           |              |
+------------+---------+-----------------------+--------------------+-----------------------------------+--------------+

Switched to context 「eks」
[root@VM-0-13-centos .kube]# kubectl  get node
NAME                    STATUS   ROLES    AGE   VERSION
eklet-subnet-ktam6hp8   Ready    <none>   56d   v2.4.4-dirty
```

一开始我们默认操作集群是tke，现在切换到eks
