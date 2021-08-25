# 修改命名空间下容器的resources配置

统一给单个命名空间下的容器配置resources，可以用下面脚本，具体的cpu和limit值设置，可以修改脚本，脚本内容如下：

```
#!/bin/sh

ns=$1

if [ $# = 0 ];then
  echo "Run 'sh modify-workload-resorces.sh --h' for more information on a command."
fi

if [[ $1 = "--h" ]];then
  echo "Enter the namespace to be repaired

Usage: sh modify-workload-resorces.sh [namespace]"
fi

main(){

for i in `kubectl get deploy,sts -n $ns |awk -F ' ' '{print $1}'  | grep -v NAME`;
do

   for j in `kubectl get $i -n $ns -o=jsonpath='{.spec.template.spec.containers[*].name}'`;
   do
      kubectl set resources $i -c=$j -n $ns  --requests=cpu=50m,memory=512Mi --limits=cpu=500m,memory=1000Mi
   done

done

}

if [ $# = 1 ];then

main

fi
```