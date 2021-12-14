这里我们讲一下如何用客户端证书和ca证书生成一份自定义用户的kubeconfig

进入节点上/etc/kubernetes/目录，我们发现节点有下面这几个文件，client的证书和秘钥是从kubelet-kubeconfig这个文件中提取出来的，提取方式参考文档<https://cloud.tencent.com/developer/article/1814668>，cluster-ca.crt是每个节点默认有的。

```
[root@VM-0-3-centos kubernetes]# ll
total 56
-rw-r--r-- 1 root root 1135 Apr 17 21:55 client-cert.pem
-rw-r--r-- 1 root root 1679 Apr 17 21:55 client-key.pem
-rw-r--r-- 1 root root 1025 Nov 26  2020 cluster-ca.crt
-rw-r--r-- 1 root root 5483 Nov 26  2020 kubelet-kubeconfig
```

下面我们执行下面命令来生成一份niewx.kubeconfig的kubeconfig，这里我们是指定了kubeconfig的名称，会生成在当前目录下，如果你不想指定名称，去掉--kubeconfig=niewx.kubeconfig这个，kubeconfig会默认生成在$HOME/.kube/config


```
# 配置kubernetes集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/cluster-ca.crt \
  --embed-certs=true \
  --server=https://cls-xxxxx.ccs.tencent-cloud.com \
  --kubeconfig=niewx.kubeconfig

# 配置客户端认证参数
kubectl config set-credentials niewx \
  --client-certificate=/etc/kubernetes/client-cert.pem \
  --embed-certs=true \
  --client-key=/etc/kubernetes/client-key.pem \
  --kubeconfig=niewx.kubeconfig

# 设置上下文参数 
kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=niewx \
  --kubeconfig=niewx.kubeconfig

# 设置默认上下文
kubectl config --kubeconfig=niewx.kubeconfig  use-context kubernetes
```

执行完上述命令就发现会生成一个user为niewx的kubeconfig文件，然后我们可以指定这个kubeconfig来访问集群，设置上下文非必须设置，不设置直接指定kubeconfg访问即可，或者将niewx.kubeconfig拷贝到$HOME/.kube/config这个文件，进行访问。

```
[root@VM-0-3-centos kubernetes]# ll | grep niewx
-rw------- 1 root root 5477 Jun  2 19:10 niewx.kubeconfig
[root@VM-0-3-centos kubernetes]# cat niewx.kubeconfig
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01UR
    server: https://cls-xxxxxx.ccs.tencent-cloud.com
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: niewx
  name: kubernetes
current-context: ""
kind: Config
preferences: {}
users:
- name: niewx
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURHVENDQWdHZ0F3SUJBZ0lJS2NXVHpIY0ZudFl3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6Q
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBd3NGWHdtNjlFVkY1WW1DNGx5bXVocFR2cGt6bCsxd0dTdWxGSnJqU0VpSTlWSTV6Ck9Zclh1UkM2VmtTTnRVa

[root@VM-0-3-centos kubernetes]# kubectl --kubeconfig=niewx.kubeconfig get node
NAME         STATUS   ROLES    AGE    VERSION
10.0.0.10    Ready    <none>   60d    v1.18.4-tke.8
10.0.0.157   Ready    <none>   147d   v1.18.4-tke.6
10.0.0.2     Ready    <none>   55d    v1.18.4-tke.8
10.0.0.3     Ready    <none>   189d   v1.18.4-tke.8
```