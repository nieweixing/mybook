有时候我们为了测试我们的服务，不需要他一直运行着，当我们测试完，就希望将服务都停掉，避免占用集群资源，一般都是将负载的副本数设置为0，但是又希望对应的deployment还存在集群中，这样下次测试直接调整副本数就可以运行服务了。

下面提供了一个脚本scale-workload-replicas.sh可以调整命名空间的所有的deployment和statefulset的副本数量

```
#!/bin/sh

ns=$1
replicas=$2

if [ $# = 0 ];then
  echo "Run 'sh scale-workload-replicas.sh --h' for more information on a command."
fi

if [[ $1 = "--h" ]];then
  echo "Please enter the first parameter enters the namespace, the second parameter enters the number of replicas 

Usage: sh scale-workload-replicas.sh [namespace] [replicas]"
fi

main(){

for i in `kubectl get deploy,sts -n $ns |awk -F ' ' '{print $1}'  | grep -v NAME`;
do
   kubectl scale --replicas=$replicas -n $ns $i
done

}

if [ $# = 2 ];then

main

fi
```

如果我们需要停掉这个命名空间下所有服务，则执行脚本第一个参数输入命名空间，第二个参数输入副本数0

```
[root@VM-0-13-centos script]# kubectl get pod -n mesh
NAME                        READY   STATUS    RESTARTS   AGE
client-6fd64cfc6b-q9ml7     2/2     Running   0          4d23h
nginx-5dbf784b68-bmr86      2/2     Running   0          16d
nginx-v1-647887d8fd-qllgx   2/2     Running   0          39d
nginx-v2-bcccc8f9f-jb926    2/2     Running   0          39d
sleep-7f474bbcbd-tdgmf      2/2     Running   0          39d
[root@VM-0-13-centos script]# sh scale-workload-replicas.sh mesh 0
deployment.apps/client scaled
deployment.apps/nginx scaled
deployment.apps/nginx-v1 scaled
deployment.apps/nginx-v2 scaled
deployment.apps/sleep scaled
[root@VM-0-13-centos script]# kubectl get pod -n mesh
No resources found.
```

如果需要拉起我们的服务，也只需要执行脚本，第一个参数输入命名空间，第二个参数输入副本数（输入1代表所有负载起一个副本）

```
[root@VM-0-13-centos script]# sh scale-workload-replicas.sh mesh 1
deployment.apps/client scaled
deployment.apps/nginx scaled
deployment.apps/nginx-v1 scaled
deployment.apps/nginx-v2 scaled
deployment.apps/sleep scaled
[root@VM-0-13-centos script]# kubectl get pod -n mesh
NAME                        READY   STATUS    RESTARTS   AGE
client-6fd64cfc6b-ggz58     2/2     Running   0          23s
nginx-5dbf784b68-c22hl      2/2     Running   0          22s
nginx-v1-647887d8fd-n7bfh   2/2     Running   0          22s
nginx-v2-bcccc8f9f-lnm6q    2/2     Running   0          22s
sleep-7f474bbcbd-z2jj7      2/2     Running   0          22s
```


