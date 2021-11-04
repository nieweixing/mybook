# 强制删除Terminating状态ns


## kubectl get ns 查看处于Terminating的ns

```
[root@VM_1_4_centos ~]# kubectl get ns | grep testns
testns                   Terminating   21d
```

## 将处于Terminating的ns的描述文件保存下来

```
[root@VM_1_4_centos ~]# kubectl get ns testns -o json > tmp.json
[root@VM_1_4_centos ~]# cat tmp.json 
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2020-10-13T14:28:07Z",
        "name": "testns",
        "resourceVersion": "13782744400",
        "selfLink": "/api/v1/namespaces/testns",
        "uid": "9ff63d71-a4a1-43bc-89e3-78bf29788844"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    "status": {
        "phase": "Terminating"
    }
}
```

## 本地启动kube proxy

```
kubectl proxy --port=8081
```

## 新开窗口执行删除操作

```
curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8081/api/v1/namespaces/testns/finalize
```

如果上面方法无法删除namespace，可以通过如下方法看下namespace是不是还有什么资源没有清理

```
若命名空间依然无法删除，则查询命名空间哪些资源
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <命名空间>


然后删除这些资源：
$ kubectl -n p-4q9rv delete projectalertgroup.management.cattle.io/projectalert-workload-alert --grace-period=0 --force

 
若 Pod 还是无法删除，可以在 Pod 中添加补丁：
kubectl -n p-4q9rv patch projectalertgroup.management.cattle.io/projectalert-workload-alert -p '{"metadata":{"finalizers":[]}}' --type='merge' 
 

添加补丁后，强制删除：
kubectl -n p-4q9rv delete projectalertrule.management.cattle.io/memory-close-to-resource-limited --grace-period=0 --force

然后执行下面命令删除namespace
kubectl patch namespace <命名空间> -p '{"metadata":{"finalizers":[]}}' --type='merge'
kubectl delete namespace cattle-system --grace-period=0 --force
```

其实也可以直接将修改对应ns生成json文件

```
[root@master-1 ~]# vim tmp.json
删除spec字段后，执行以下curl命令，使用kube-apiserver的8081端口，执行删除操作

#注意修改@XXX.json ，修改 namespaces/XXX/finalize ,其中XXX 表示你要删除的命名空间名称
[root@master-1 ~]# curl -k -H "Content-Type: application/json" -X PUT --data-binary @tmp.json http://127.0.0.1:8081/api/v1/namespaces/mysql/finalize
```

用下面命令清理也可以

```
$ kubectl get ns delete-me -o json | jq '.spec.finalizers=[]' > ns-without-finalizers.json
cat ns-without-finalizers.json
$ kubectl proxy &
$ PID=$!
$ curl -X PUT http://localhost:8001/api/v1/namespaces/delete-me/finalize -H "Content-Type: application/json" --data-binary @ns-without-finalizers.json
$ kill $PID
```
