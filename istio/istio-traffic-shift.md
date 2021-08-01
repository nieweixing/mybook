有时候我们需要将服务的部分流量转移到另外一个服务，灰度测试的时候，为了测试新版本是否有问题，我们需要将部分流量打到新版本上，本次任务，我们实现将请求流量20%打到v1版本，80%的流量打到v2版本。

# 部署nginx的v1和v2版本

```
apiVersion: v1
data:
  index.html: I am v1 version
kind: ConfigMap
metadata:
  name: index-v1
  namespace: mesh

---

apiVersion: v1
data:
  index.html: I am v2 version
kind: ConfigMap
metadata:
  name: index-v2
  namespace: mesh

---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    version: v1
  name: nginx-v1
  namespace: mesh
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      version: v1
  template:
    metadata:
      labels:
        app: nginx
        version: v1
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx-v1
        resources:
          limits:
            cpu: 500m
            memory: 1Gi
          requests:
            cpu: 250m
            memory: 256Mi
        volumeMounts:
        - mountPath: /usr/share/nginx/html/index.html
          name: vol
          subPath: index.html
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: qcloudregistrykey
      restartPolicy: Always
      volumes:
      - configMap:
          defaultMode: 420
          name: index-v1
        name: vol

---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
    version: v2
  name: nginx-v2
  namespace: mesh
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
      version: v2
  template:
    metadata:
      labels:
        app: nginx
        version: v2
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx-v2
        resources:
          limits:
            cpu: 500m
            memory: 1Gi
          requests:
            cpu: 250m
            memory: 256Mi
        volumeMounts:
        - mountPath: /usr/share/nginx/html/index.html
          name: vol
          subPath: index.html
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: qcloudregistrykey
      restartPolicy: Always
      volumes:
      - configMap:
          defaultMode: 420
          name: index-v2
        name: vol

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-v1
  labels:
    app: nginx
    service: nginx
spec:
  ports:
  - port: 80
    name: http
  selector:
    app: nginx

```

这里部署了一个统一的入口service关联到后端的v1和v2版本pod里面

# 创建DestinationRule

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: nginx
  namespace: mesh
spec:
  host: nginx
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

# 创建gateway

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: nginx-gw
  namespace: mesh
spec:
  servers:
    - port:
        number: 88
        name: HTTP-88-ebsi
        protocol: HTTP
      hosts:
        - '*'
  selector:
    app: istio-ingressgateway
    istio: ingressgateway
```

挂载ingressgateway作为统一的访问入口

# 创建VirtualService

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nginx
  namespace: mesh
spec:
  hosts:
    - '*'
  gateways:
    - mesh/nginx-gw
  http:
    - route:
      - destination:
          host: nginx
          subset: v1
        weight: 20
      - destination:
          host: nginx
          subset: v2
        weight: 80
```

VirtualService里面的weight字段代表的是流量转发的比例，这里将流量20%转发到v1，流量80%转到v2

# 访问测试

我们直接用ingress的LoadBalancer的vip和88端口访问10次看下

```
[root@VM-0-13-centos ~]# for i in {1..10};do curl 106.xx.xx.93:88 ; echo "" ;done
I am v2 version
I am v2 version
I am v2 version
I am v2 version
I am v2 version
I am v2 version
I am v2 version
I am v2 version
I am v1 version
I am v1 version
```

从上面的测试结果看，"I am v1 version"出现了2次，"I am v2 version"出现了8次，说明请求的流量是符合VirtualService的分配比例的