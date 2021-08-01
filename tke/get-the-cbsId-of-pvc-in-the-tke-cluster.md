使用tke的过程中，有时候我们需要获取下pvc对应的cbs id，可以用下面get_pvc_cbs_id.sh脚本去快速获取

```
#!/bin/bash

if [ $# = 0 ];then
  echo "Use "--h" for more information about a given command."
fi

if [[ $1 = "--h" ]];then
   echo " Please enter the pvc name for the first parameter, and empty the name for the second parameter
       Usage:
          sh get_pvc_cbs_id.sh pvc-name namespace"
fi

ns=$2
pvc_name=$1


main(){
  pv=`kubectl get pvc ${pvc_name} -n $ns -o jsonpath={.spec.volumeName}`

  cbs_id=`kubectl get pv ${pv} -o jsonpath={.spec.qcloudCbs.cbsDiskId}`

  echo $cbs_id
}

if [ $# = 2 ];then

main

fi
```

执行脚本，第一个参数输入pvc名称，第二个参数输入对应命名空间

```
[root@VM-0-13-centos script]# sh get_pvc_cbs_id.sh nwx-mysql mysql
disk-xxxxx
```