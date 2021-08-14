有时候由于节点的内存或者磁盘使用率较高导致集群中产生了大量的evited状态pod，这些pod如果不手动删除，会一直存在集群中，这里我们提供了脚本clean-evicted-pod.sh来一键清理集群中的evited状态pod。

```
#!/bin/bash

basepath=$(cd `dirname $0`; pwd)

kubectl get pods -A | grep Evicted | awk -F " " '{print $1,$2}'  >> $basepath/tmp.file

while read line
do
    ns=`echo $line | awk -F " " '{print $1}'`
    podname=`echo $line | awk -F " " '{print $2}'`
    kubectl delete pod $podname -n $ns
done < $basepath/tmp.file

rm $basepath/tmp.file
```

复制上面脚本，直接执行即可，如果想清除其他状态的pod，可以将grep Evited改成其他的，比如说grep Pending