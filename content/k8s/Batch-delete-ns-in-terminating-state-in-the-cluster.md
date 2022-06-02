# 批量删除k8s集群terminating状态的ns

有的时候删除ns会卡主，处于terminating状态，这里为什么会卡主可以参考这个文档<https://cloud.tencent.com/developer/article/1802531>

如果集群存在多个terminating状态的ns，一个个删除比较麻烦，这里提供下一个简单的小脚本batch-delete-terminating-ns.sh来删除下，脚本内容如下

```
#!/bin/bash

set -euxo pipefail

if [ -f "/etc/redhat-release" ]; then
   yum install jq -y
fi

if [ -f "/etc/lsb-release" ]; then
   apt-get install jq -y
fi

kubectl proxy &

PID=$!

for i in `kubectl get ns | grep Terminating  | awk -F  " " '{print $1}'`;
do
     kubectl get ns $i -o json | jq '.spec.finalizers=[]' > $i.json
     curl -X PUT http://localhost:8001/api/v1/namespaces/$i/finalize -H "Content-Type: application/json" --data-binary @${i}.json
     rm -rf $i.json
done

kill $PID
```