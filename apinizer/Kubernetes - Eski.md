
#### Nodes

```
kubectl get nodes -owide

NAME        STATUS   ROLES    AGE      VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kmaster01   Ready    master   3y228d   v1.18.4   172.20.50.73   <none>        Ubuntu 20.04.3 LTS   5.4.0-212-generic   docker://20.10.8
kworker01   Ready    <none>   3y226d   v1.18.4   172.20.50.75   <none>        Ubuntu 20.04.3 LTS   5.4.0-189-generic   docker://20.10.8
kworker02   Ready    <none>   3y228d   v1.18.4   172.20.50.76   <none>        Ubuntu 20.04 LTS     5.4.0-189-generic   docker://20.10.8
master03    Ready    master   616d     v1.18.4   172.20.50.85   <none>        Ubuntu 20.04.5 LTS   5.4.0-212-generic   docker://24.0.5
worker03    Ready    <none>   616d     v1.18.4   172.20.50.86   <none>        Ubuntu 20.04.5 LTS   5.4.0-189-generic   docker://24.0.5
```

```
kubectl get svc -A

NAMESPACE     NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
apinizer      manager               NodePort    10.102.119.120   <none>        8080:32080/TCP           47d
prod          cache-http-service    ClusterIP   10.101.151.91    <none>        8090/TCP                 2y202d
prod          cache-hz-service      ClusterIP   10.108.243.212   <none>        5701/TCP                 274d
prod          worker-http-service   NodePort    10.96.221.48     <none>        8091:30080/TCP           624d

```

NodePort olarak çalışan 2 servis var.

Manager uygulaması 32080 portundan

Worker servisine de 30080 portundan ulaşabiliriz.

#### Pods

```
kubectl get pods -n apinizer -l app=manager -owide

NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
manager-6cd767d887-26m6c   1/1     Running   0          5d23h   10.244.3.22   kworker01   <none>           <none>
```


```
kubectl get pods -n prod -l app=worker -owide

NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
worker-f8d95c477-88gww   1/1     Running   0          5d23h   10.244.4.43   worker03    <none>           <none>
worker-f8d95c477-c5rnm   1/1     Running   0          5d23h   10.244.2.19   kworker02   <none>           <none>
```


```
kubectl get pods -n prod -l app=cache -owide

NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
cache-5cfd8dbbb4-vb5g4   1/1     Running   0          5d23h   10.244.2.20   kworker02   <none>           <none>
```


#### Secrets

```
kubectl get secret mongo-db-credentials -n apinizer  -o jsonpath={'.data.dbName'} | base64 -d

apinizerdb
```

```
kubectl get secret mongo-db-credentials -n apinizer  -o jsonpath={'.data.dbUrl'} | base64 -d

mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.73:25080,172.20.50.74:25080,172.20.50.85:25080/?authSource=admin&replicaSet=apinizer-replicaset
```


```
kubectl get secret mongo-db-credentials -n prod  -o jsonpath={'.data.dbName'} | base64 -d
apinizerdb
```

```
kubectl get secret mongo-db-credentials -n prod  -o jsonpath={'.data.dbUrl'} | base64 -d

mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.105:25080,172.20.50.106:25080,172.20.50.107:25080/?authSource=admin&replicaSet=apinizer-replicaset
```

```
kubectl get secret -n apinizer default-token-hhvxf -o jsonpath={'.data'}

{"ca.crt":"LS0tLS1CRUdJTi..."}
```

