Kubernetes 支持对容器进行周期性探测，并根据探测结果判断容器的健康状态，执行额外的操作。

# 健康检查类别

健康检查分为以下类别：

* 容器存活检查：用于检测容器是否存活，类似于执行 ps 命令检查进程是否存在。如果容器的存活检查失败，集群会对该容器执行重启操作。如果容器的存活检查成功，则不执行任何操作。
* 容器就绪检查：用于检测容器是否准备好开始处理用户请求。例如，程序的启动时间较长时，需要加载磁盘数据或者要依赖外部的某个模块启动完成才能提供服务。此时，可通过容器就绪检查方式检查程序进程，确认程序是否启动完成。如果容器的就绪检查失败，集群会屏蔽请求访问该容器。如果容器的就绪检查成功，则会开放对该容器的访问。

# 健康检查方式

## TCP 端口探测

TCP 端口探测的原理如下：

对于提供 TCP 通信服务的容器，集群周期性地对该容器建立 TCP 连接。如果连接成功，证明探测成功，否则探测失败。选择 TCP 端口探测方式，必须指定容器监听的端口。

例如，一个 redis 容器，它的服务端口是6379。我们对该容器配置了 TCP 端口探测，并指定探测端口为6379，那么集群会周期性地对该容器的6379端口发起 TCP 连接。如果连接成功，证明检查成功，否则检查失败。


## HTTP 请求探测

HTTP 请求探测是针对于提供 HTTP/HTTPS 服务的容器，并集群周期性地对该容器发起 HTTP/HTTPS GET 请求。如果 HTTP/HTTPS response 返回码属于200 - 399范围，证明探测成功，否则探测失败。使用 HTTP 请求探测必须指定容器监听的端口和 HTTP/HTTPS 的请求路径。

例如，提供 HTTP 服务的容器，服务端口为 80，HTTP 检查路径为 /health-check，那么集群会周期性地对容器发起GET http://containerIP:80/health-check 请求。

## 执行命令检查

执行命令检查是一种强大的检查方式，该方式要求用户指定一个容器内的可执行命令，集群会周期性地在容器内执行该命令。如果命令的返回结果是0，检查成功，否则检查失败。
对于 TCP 端口探测 和 HTTP 请求探测，都可以通过执行命令检查的方式来替代：

* 对于 TCP 端口探测，可以写一个程序对容器的端口进行 connect。如果 connect 成功，脚本返回0，否则返回-1。
* 对于 HTTP 请求探测，可以写一个脚本来对容器进行 wget 并检查 response 的返回码。例如，wget http://127.0.0.1:80/health-check。如果返回码在200 - 399的范围，脚本返回0，否则返回 -1。

# 脚本进行健康探测

其实很多时候，我们对业务容器进行健康探测可能无法用简单的一条命令或者探测端口是否可达就判断业务是否正常，而是需要用较复杂的脚本的来实现，这样我们就需要来判断脚本的返回值了，很多人会直接在脚本输出0或者-1来进行判断，但是发现健康检查不生效。

其实这里不能用echo来输出对应的返回值，因为你用echo输出值，实际上执行脚本还是成功的，探针是获取的echo $?的值，这个值还是0，那么用脚本进行健康探测，需要怎么来写呢？

```
#!/bin/sh

num=$1

if [ $num = 1 ]; then
   echo "ok"
else
   echo "fail"
   exit 1
fi
```

这里我们如果想探测失败需要用exit 1，这样执行脚本获取的返回值就是1，探针就会认为这个是失败的。

下面我们把脚本放到镜像里面测试下，然后配置下探针，看下是否会生效，这里可以用这个镜像测试ccr.ccs.tencentyun.com/nwx_registry/health-check:latest

下面我们看下我们的deployment对应的健康检查配置

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: health-check
    qcloud-app: health-check
  name: health-check
  namespace: tke-test
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: health-check
      qcloud-app: health-check
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        k8s-app: health-check
        qcloud-app: health-check
    spec:
      containers:
      - image: ccr.ccs.tencentyun.com/nwx_registry/health-check:latest
        imagePullPolicy: Always
        livenessProbe:
          exec:
            command:
            - sh
            - /tmp/test.sh
            - "2"
          failureThreshold: 3
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: health-check
        resources:
          limits:
            cpu: 500m
            memory: 1Gi
          requests:
            cpu: 250m
            memory: 256Mi
        securityContext:
          privileged: false
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: qcloudregistrykey
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

根据我们配置的存活探针，这里检查脚本会返回1，也就是失败，连续探测3次失败后就会重启容器，我们验证下是不是这样的

```
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age               From                 Message
  ----     ------     ----              ----                 -------
  Normal   Scheduled  42s               default-scheduler    Successfully assigned tke-test/health-check-65b4d88575-sg7p9 to 10.0.0.157
  Normal   Started    40s               kubelet, 10.0.0.157  Started container health-check
  Warning  Unhealthy  2s (x3 over 22s)  kubelet, 10.0.0.157  Liveness probe failed: fail
  Normal   Killing    2s                kubelet, 10.0.0.157  Container health-check failed liveness probe, will be restarted
  Normal   Pulling    0s (x2 over 40s)  kubelet, 10.0.0.157  Pulling image "ccr.ccs.tencentyun.com/nwx_registry/health-check:latest"
  Normal   Pulled     0s (x2 over 40s)  kubelet, 10.0.0.157  Successfully pulled image "ccr.ccs.tencentyun.com/nwx_registry/health-check:latest"
  Normal   Created    0s (x2 over 40s)  kubelet, 10.0.0.157  Created container health-check
```

从事件日志，可以发现，这里返回1，探针就认为是失败的，所以这里连续3次就重启容器。

综上所述，用探针来检查容器状态，如果用脚本进行判断，当失败时候，用exit 1来返回脚本返回结果。
