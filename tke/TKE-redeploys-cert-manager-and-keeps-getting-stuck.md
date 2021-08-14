最近研究了一下在tke上部署cert-manager来实现为ingress免费签发域名证书，具体的部署文档可以参考<https://www.niewx.cn/cert-manager/2021/08/07/cert-manager-issues-free-certificates-for-domain-names/>

这里为了测试，我先部署了cert-manager，然后又删除了，第二次部署的时候发现一直卡在创建crd上，导致一直部署不成功，具体现象如下。

```
[root@VM-0-13-centos ~]# kubectl apply --validate=false -f https://raw.githubusercontent.com/TencentCloudContainerTeam/manifest/master/cert-manager/cert-manager.yaml
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
^C
```

那么这里是怎么回事呢？既然创建crd一直卡在，那么肯定是创建crd的过程哪里除了问题，当时想的是不是之前创建的crd资源没有清理干净导致的呢？

首先我查下和cert-manager相关的crd资源

```
[root@VM-0-13-centos ~]# kubectl get crd | grep cert
certificaterequests.cert-manager.io                             2021-08-09T10:55:18Z
certificates.cert-manager.io                                    2021-08-09T10:55:18Z
challenges.acme.cert-manager.io                                 2021-06-02T06:45:43Z
```

这里我们清理下未删除的crd，当我们删除challenges.acme.cert-manager.io 这个crd一直无法删除成功，那么问题就明朗了，就是crd无法删除，导致创建crd一直卡主，我这边尝试强制删除下

```
[root@VM-0-13-centos ~]# kubectl delete crd challenges.acme.cert-manager.io --grace-period=0 --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
customresourcedefinition.apiextensions.k8s.io "challenges.acme.cert-manager.io" force deleted
[root@VM-0-13-centos ~]# kubectl get crd challenges.acme.cert-manager.io
NAME                              CREATED AT
challenges.acme.cert-manager.io   2021-06-02T06:45:43Z
```

强制删除后发现crd资源还存在，这里删除失败了，那么要清楚这种删除不掉的crd资源呢？

```
[root@VM-0-13-centos ~]# kubectl patch crd challenges.acme.cert-manager.io -p '{"metadata":{"finalizers":[]}}' --type=merge
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io patched
[root@VM-0-13-centos ~]# kubectl delete crd challenges.acme.cert-manager.io --grace-period=0 --force
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
Error from server (NotFound): customresourcedefinitions.apiextensions.k8s.io "challenges.acme.cert-manager.io" not found
```

执行上面命令，先将finalizers字段配置信息删除，然后强制删除即可，删除了相关crd后，我们再部署下

```
[root@VM-0-13-centos ~]# kubectl apply --validate=false -f https://raw.githubusercontent.com/TencentCloudContainerTeam/manifest/master/cert-manager/cert-manager.yaml
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
namespace/cert-manager created
serviceaccount/cert-manager-cainjector created
serviceaccount/cert-manager created
serviceaccount/cert-manager-webhook created
clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificates unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-challenges unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged
clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificates unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-challenges unchanged
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim unchanged
role.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection unchanged
role.rbac.authorization.k8s.io/cert-manager:leaderelection unchanged
role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
rolebinding.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection unchanged
rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection configured
rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
service/cert-manager created
service/cert-manager-webhook created
deployment.apps/cert-manager-cainjector created
deployment.apps/cert-manager created
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook configured
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook configured
```

再次部署就可以成功了。

**总结**

当我们在集群重新部署crd资源卡主的时候，可以搜下集群是不是存在上次未删除的crd资源，然后清理下，如强制删也无法删除，用patch修改下finalizers字段后再删除，清理成功后，再部署即可。