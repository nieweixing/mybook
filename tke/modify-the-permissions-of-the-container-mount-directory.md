我们在部署应用到k8s集群中，很多时候容器的启动用户不是root，并且会挂载数据目录到pvc上，但是又要通过启动用户写文件到挂载目录上，这样就会出现一个问题，就是启动用户没有权限读我们的数据挂载目录，今天我们提供一个init修改目录权限的方法来解决这个问题

首先我们部署一个grafana，并将grafana的数据存储目录/var/lib/grafana挂载到cbs上，这里pvc提前创建好了

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: grafana
    qcloud-app: grafana
  name: grafana
  namespace: monitor
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: grafana
      qcloud-app: grafana
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: grafana
        qcloud-app: grafana
    spec:
      containers:
      - image: grafana/grafana:master
        imagePullPolicy: IfNotPresent
        name: grafana
        resources:
          limits:
            cpu: "1"
            memory: 2Gi
          requests:
            cpu: "1"
            memory: 1Gi
        securityContext:
          privileged: false
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/grafana
          name: vol
          subPath: grafana
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: qcloudregistrykey
      
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: grafana
```

当我们启动pod之后会发现，pod一直在重启，查看日志报错是没有/var/lib/grafana这个目录的权限，因为grafana镜像的启动用户是grafana，而/var/lib/grafana这个目录，grafana是没有权限读写的。

```
[root@VM-0-13-centos ~]# kubectl logs -f grafana-84774b67d9-r79fv -n monitor
GF_PATHS_DATA='/var/lib/grafana' is not writable.
You may have issues with file permissions, more information here: http://docs.grafana.org/installation/docker/#migrate-to-v51-or-later
mkdir: can't create directory '/var/lib/grafana/plugins': Permission denied
```

下面我们用一个init容器来修改/var/lib/grafana这个目录的权限， 再看下pod能否启动成功，这里重点关注init容器的配置

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: grafana
    qcloud-app: grafana
  name: grafana
  namespace: monitor
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: grafana
      qcloud-app: grafana
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: grafana
        qcloud-app: grafana
    spec:
      containers:
      - image: grafana/grafana:master
        imagePullPolicy: IfNotPresent
        name: grafana
        resources:
          limits:
            cpu: "1"
            memory: 2Gi
          requests:
            cpu: "1"
            memory: 1Gi
        securityContext:
          privileged: false
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/grafana
          name: vol
          subPath: grafana
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: qcloudregistrykey
      initContainers:
      - args:
        - -c
        - mkdir -p /var/lib/grafana && chmod -R 777 /var/lib/grafana
        command:
        - /bin/sh
        image: centos:7
        imagePullPolicy: Always
        name: init
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
        - mountPath: /var/lib/grafana
          name: vol
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: grafana
```

这里将上面的yaml部署后，查看pod是running，直接访问3000端口也是可以通的，说明grafana是可以正常启动的。

```
[root@VM-0-13-centos ~]# kubectl get pod -n monitor -o wide | grep grafana
grafana-84dd4d4454-sp2xx             1/1     Running   0          2m4s   10.0.2.83    10.0.0.3    <none>           1/1
[root@VM-0-13-centos ~]# curl 10.0.2.83:3000
<a href="/login">Found</a>.
```

这里我们简要说下我们的解决方案，其实就是用一个init容器创建/var/lib/grafana这个目录，然后挂载到cbs卷上，并修改权限为777，init容器执行完之后，grafana再运行去读这个目录的权限就是777，即使grafana的启动用户不是root，也有权限读/var/lib/grafana这个目录，容器也就可以正常运行了

```
[root@VM-0-13-centos ~]# kubectl  exec -it grafana-84dd4d4454-sp2xx bash
Error from server (NotFound): pods "grafana-84dd4d4454-sp2xx" not found
[root@VM-0-13-centos ~]# kubectl  exec -it grafana-84dd4d4454-sp2xx bash -n monitor
bash-5.0$ cd var/lib/
bash-5.0$ ls -al
total 24
drwxr-xr-x    1 root     root          4096 Nov 26  2020 .
drwxr-xr-x    1 root     root          4096 Oct 21  2020 ..
drwxr-xr-x    2 root     root          4096 Oct 21  2020 apk
drwxrwxrwx    4 root     root          4096 Jun 13 12:03 grafana
drwxr-xr-x    2 root     root          4096 Oct 21  2020 misc
drwxr-xr-x    2 root     root          4096 Oct 21  2020 udhcpd
```