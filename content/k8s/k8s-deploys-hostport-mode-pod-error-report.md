# k8s部署hostport模式pod报错端口冲突

最近有客户在使用hostport的模式部署deployment的时候，有出现这个错误0/4 nodes are available: 4 node(s) didn't have free ports for the requested pod ports，导致deployment部署失败，从报错看是端口冲突了。

因为hostport是用的占用的node节点的端口，既然是端口冲突，我们到节点上查看下对应端口是否有监听，登录节点用netstat查看对应端口的监听，发现并没有被占用，端口没有被占用，为什么会出现部署失败报错端口被占用的问题。

后面我根据报错的yaml进行测试下，具体现象如下

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: hostport-test
    qcloud-app: hostport-test
  name: hostport-test
  namespace: tke-test
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: hostport-test
      qcloud-app: hostport-test
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: hostport-test
        qcloud-app: hostport-test
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: hostport-test
        ports:
        - containerPort: 80
          hostPort: 80
          name: http
          protocol: TCP
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

部署了hostPort的pod后，我们登录节点查看80端口的监听，发现并没有被监听

```
[root@VM-0-10-centos ~]# netstat -an | grep tcp | grep 80
tcp        0      0 0.0.0.0:30080           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:32580           0.0.0.0:*               LISTEN
tcp        0      0 10.0.0.10:46548         169.254.0.71:80         ESTABLISHED
tcp        0      0 10.0.0.10:33762         169.254.0.71:80         ESTABLISHED
```

接下来我们接着往hostport-test所在的节点部署一个hostport模式的pod看看

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: hostport-new
    qcloud-app: hostport-new
  name: hostport-new
  namespace: tke-test
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: hostport-new
      qcloud-app: hostport-new
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: hostport-new
        qcloud-app: hostport-new
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - 10.0.0.10
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: hostport-new
        ports:
        - containerPort: 80
          hostPort: 80
          name: http
          protocol: TCP
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
      hostNetwork: true
      imagePullSecrets:
      - name: qcloudregistrykey
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

我们看下hostport-new的事件，看下是否有出现报错

```
[root@VM-0-10-centos ~]# kubectl describe  pod -n tke-test hostport-new-86786bc46f-kl8ph
Name:           hostport-new-86786bc46f-kl8ph
Namespace:      tke-test
.................
Events:
  Type     Reason                        Age                   From                  Message
  ----     ------                        ----                  ----                  -------
  Normal   FailedSchedulingOnEkletNodes  7m49s (x16 over 12m)  admission-controller  node(s) didn't match node selector
  Warning  FailedScheduling              65s (x10 over 12m)    default-scheduler     0/5 nodes are available: 1 Too many pods, 2 node(s) didn't have free ports for the requested pod ports, 2 node(s) didn't match node selector.
```

从事件日志看，部署报错，也是报错端口没法使用，但是我们到节点查看80端口的确没有被监听，这是怎么回事呢？

这里肯定是哪里配置不正常导致，我们仔细分析了下部署的yaml文件，发现hostport-test配置的hostport，但是没有配置hostNetwork，在k8s里面如果要hostport生效，需要配置hostNetwork为true才行，hostport-test没有配置hostNetwork，所以到节点上查看80端口没有被监听。

既然节点端口没被监听，为什么部署hostport-new这个pod时候还是会报错呢？hostport-new是有配置hostNetwork，其实从事件可以看出pod是调度失败，pod的调度是从etcd里面获取信息去判断节点是否存在相同端口的pod，也就是说当你有一个deployment存在hostport为80的字段时，Scheduler就会认为节点80端口已经被占用了，Scheduler不会实际去检查你的节点是否有这个端口的监听。

从上面的分析，就可以看出为什么节点没有起80端口的监听，但是部署hostport会报错端口被占用的报错。