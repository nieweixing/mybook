随着业务的扩展，有时候希望客户能就近访问我们的网站，减少网络延迟，但是很多时候会在多地域部署相同的服务，其实有了istio后，我们可以利用就近接入来解决这个问题，这样无需在另一地域集群部署整套业务，只需在网格管理的另一个集群中部署边缘代理网关并配置好监听规则，即可以另一集群为入口访问电商网站业务

腾讯云上的服务网格如果不同地域通过云联网打通了可以通过同一个网格管理，下面我们通过同一个vpc下的2个集群模拟多个地域来配置下就近访问

首先我们在网格主集群A中部署一个nginx服务，通过gateway提供访问

```
apiVersion: apps/v1
kind: Deployment
metadata:
  generation: 5
  labels:
    k8s-app: nginx
    qcloud-app: nginx
  name: nginx
  namespace: mesh
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: nginx
      qcloud-app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: nginx
        qcloud-app: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
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

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-test
  namespace: mesh
spec:
  ports:
  - name: 80-80-tcp
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    k8s-app: nginx
    qcloud-app: nginx
  sessionAffinity: None
  type: ClusterIP
```

主集群A部署gateway

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

我们将子集群B加入网格内，也会部署一个istio-ingressgateway作为访问的入口，然后在B集群部署一个gateway来访问我们A集群部署的服务

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: child-gw
  namespace: mesh
spec:
  servers:
    - port:
        number: 80
        name: HTTP-80-6pnq
        protocol: HTTP
      hosts:
        - '*'
  selector:
    app: istio-ingressgateway-2
    istio: ingressgateway
```

然后我们创建一个VirtualService来关联这2个gateway

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: nearest-access-vs
  namespace: mesh
spec:
  hosts:
    - '*'
  gateways:
    - mesh/child-gw
    - mesh/nginx-gw
  http:
    - route:
        - destination:
            host: nginx-test.mesh.svc.cluster.local
            port:
              number: 80
```

现在我们在B集群没有部署nginx服务，然后我们用B集群的gateway来访问我们服务看是否能访问到

```
[root@VM-17-4-centos ~]# kubectl get svc -n istio-system
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
istio-ingressgateway-2   LoadBalancer   172.22.253.209   10.0.0.31     80:31199/TCP,15021:30299/TCP    4h5m
istiod-1-8-1             LoadBalancer   172.22.255.147   10.0.0.141    15012:32190/TCP,443:31540/TCP   4h28m
istiod-1-8-1-injector    ClusterIP      None             <none>        443/TCP                         4h28m
kube-mesh                LoadBalancer   172.22.254.32    10.0.0.144    443:30431/TCP                   4h28m
zipkin                   ClusterIP      172.22.255.45    <none>        9411/TCP                        4h27m
[root@VM-17-4-centos ~]# kubectl get pod -n mesh
No resources found in mesh namespace.
[root@VM-17-4-centos ~]# kubectl get svc -n istio-system
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
istio-ingressgateway-2   LoadBalancer   172.22.253.209   10.0.0.31     80:31199/TCP,15021:30299/TCP    4h5m
istiod-1-8-1             LoadBalancer   172.22.255.147   10.0.0.141    15012:32190/TCP,443:31540/TCP   4h28m
istiod-1-8-1-injector    ClusterIP      None             <none>        443/TCP                         4h28m
kube-mesh                LoadBalancer   172.22.254.32    10.0.0.144    443:30431/TCP                   4h28m
zipkin                   ClusterIP      172.22.255.45    <none>        9411/TCP                        4h27m
[root@VM-17-4-centos ~]# kubectl get pod -n mesh
No resources found in mesh namespace.
[root@VM-17-4-centos ~]# curl 10.0.0.31:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

从上面测试结果看，我们的ingressgateway的service是一个内网lb，并且我们在mesh命名空间下没有部署服务，我们通过lb的vip和80端口是可以访问到nginx服务的，这里说明我们访问B集群的流量被路由到了主集群A中。
