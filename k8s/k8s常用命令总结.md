# k8s常用命令

## 获取hostNetwork网络模式的deployment

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.hostNetwork==true)]} {"namespace: "} {.metadata.namespace}{" name:"} {.metadata.name} {"\n"} {end}'
```
## 获取特权模式的deployment

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.containers[*].securityContext.privileged==true)]} {"namespace: "}{.metadata.namespace}{"name:"} {.metadata.name} {"\n"} {end}'
```

## 获取控制进程可以获得超出其父进程的特权的deployment

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.containers[*].securityContext.allowPrivilegeEscalation==true)]} {"namespace: "}{.metadata.namespace}{"name:"} {.metadata.name} {"\n"} {end}'
```

## 获取所有deployment的容器名称和namespace

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.containers[*])]} {"namespace: "}{.metadata.namespace}{"name:"} {.metadata.name} {"\n"} {end}'
```

## 获取所有deployment的容器端口

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[*]} {"namespace: "}{.metadata.namespace}{" name:"} {.metadata.name} {" "}{.spec.template.spec.containers[*].ports} {"\n"} {end}'
```

## 获取配置了hostport的deployment

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[*]} {"namespace: "}{.metadata.namespace}{" name:"} {.metadata.name} {" "}{.spec.template.spec.containers[*].ports} {"\n"} {end}' |grep map | grep hostPort | awk '{print $1 $2 " "$3 $4}'
```
 
## 获取配置了capabilities属性的deployment

```
kubectl get deployment --all-namespaces -o jsonpath='{range .items[*]} {"namespace: "}{.metadata.namespace}{" name:"} {.metadata.name} {" "}{.spec.template.spec.containers[*].securityContext} {"\n"} {end}' |grep capabilities | awk '{print $1 $2 " "$3 $4}'
```
 
## 获取所有配置了hostNetwork的DaemonSet

```
kubectl get ds --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.hostNetwork==true)]} {"namespace: "} {.metadata.namespace}{" name:"} {.metadata.name} {"\n"} {end}'
```

## 获取所有配置了hostIPC模式的DaemonSet

```
kubectl get ds --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.hostIPC==true)]} {"namespace: "} {.metadata.namespace}{" name:"} {.metadata.name} {"\n"} {end}'
```

## 获取所有配置了hostPID模式的DaemonSet

```
kubectl get ds --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.hostPID==true)]} {"namespace: "} {.metadata.namespace}{" name:"} {.metadata.name} {"\n"} {end}'
```

## 获取所有配置了allowPrivilegeEscalation模式的DaemonSet

```
kubectl get ds --all-namespaces -o jsonpath='{range .items[?(@.spec.template.spec.containers[*].securityContext.allowPrivilegeEscalation==true)]} {"namespace: "}{.metadata.namespace}{"name:"} {.metadata.name} {"\n"} {end}'
```
 
## 获取配置了hostport的DaemonSet

```
kubectl get ds --all-namespaces -o jsonpath='{range .items[*]} {"namespace: "}{.metadata.namespace}{" name:"} {.metadata.name} {" "}{.spec.template.spec.containers[*].ports} {"\n"} {end}' |grep map | grep hostPort | awk '{print $1 $2 " "$3 $4}'
```

## 获取所有pod的ip和所在node的ip

```
kubectl get pods --all-namespaces    -o=jsonpath='{range .items[*]}[nodeip:{.status.hostIP}, podip:{.status.podIP}]{"\n"}{end}'
```