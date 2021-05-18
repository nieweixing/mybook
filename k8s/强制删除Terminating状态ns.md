# 强制删除Terminating状态ns

```
kubectl patch namespace <命名空间> -p '{"metadata":{"finalizers":[]}}' --type='merge'
kubectl delete namespace cattle-system --grace-period=0 --force

若命名空间依然无法删除，则查询命名空间哪些资源
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <命名空间>


然后删除这些资源：
$ kubectl -n p-4q9rv delete projectalertgroup.management.cattle.io/projectalert-workload-alert --grace-period=0 --force

命名空间下资源添加补丁
 

若 Pod 还是无法删除，可以在 Pod 中添加补丁：
kubectl -n p-4q9rv patch projectalertgroup.management.cattle.io/projectalert-workload-alert -p '{"metadata":{"finalizers":[]}}' --type='merge' 
 

添加补丁后，强制删除：
kubectl -n p-4q9rv delete projectalertrule.management.cattle.io/memory-close-to-resource-limited --grace-period=0 --force



[root@master-1 ~]# vim tmp.json
删除spec字段后，执行以下curl命令，使用kube-apiserver的8080端口，执行删除操作
#注意修改@XXX.json ，修改 namespaces/XXX/finalize ,其中XXX 表示你要删除的命名空间名称
[root@master-1 ~]# curl -k -H "Content-Type: application/json" -X PUT --data-binary @mysql.json http://127.0.0.1:8080/api/v1/namespaces/mysql/finalize

```