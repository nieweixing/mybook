这里我们讲一下如何在tke集群上部署docker in docker的pod，这里前提条件是集群的runtime需要用的是docker类型，如果是containerd是不行的。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: docker-in-docker
    qcloud-app: docker-in-docker
  name: docker-in-docker
  namespace: tke-test
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: docker-in-docker
      qcloud-app: docker-in-docker
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: docker-in-docker
        qcloud-app: docker-in-docker
    spec:
      containers:
      - command:
        - sleep
        - 70d
        image: docker:latest
        imagePullPolicy: Always
        name: docker-in-docker
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
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: vol
          subPath: docker.sock
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: qcloudregistrykey
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /var/run
          type: DirectoryOrCreate
        name: vol
```

这里我们用官方提供的docker in docker镜像，镜像默认是没有常驻进程，需要加上sleep命令起一个常驻进程，然后我们将节点的/var/run/docker.sock挂载容器内。

接下来我们进入容器就可以执行docker命令了

```
[root@VM-0-13-centos ~]# kubectl exec -it  docker-in-docker-5b49479696-cm6gd -n tke-test /bin/sh
/ # docker version
Client:
 Version:           20.10.6
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        370c289
 Built:             Fri Apr  9 22:42:10 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          19.03.9
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       9d988398e7
  Built:            Fri May 15 00:28:17 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
/ #

```