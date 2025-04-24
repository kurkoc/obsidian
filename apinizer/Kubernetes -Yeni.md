#### Nodes

```
kubectl get nodes -owide

NAME            STATUS   ROLES           AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
smasterappc01   Ready    control-plane   7d20h   v1.31.0   172.20.50.105   <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
smasterappc02   Ready    control-plane   7d20h   v1.31.0   172.20.50.106   <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
smasterappc03   Ready    control-plane   7d20h   v1.31.0   172.20.50.107   <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
sworkerappc01   Ready    <none>          7d20h   v1.31.0   172.20.50.95    <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
sworkerappc02   Ready    <none>          7d20h   v1.31.0   172.20.50.96    <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
sworkerappc03   Ready    <none>          7d20h   v1.31.0   172.20.50.97    <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25

```

#### Services

```
kubectl get svc -A -owide

NAMESPACE         NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE    SELECTOR
apinizer-portal   apinizer-portal-service   NodePort    10.102.87.28     <none>        8080:32090/TCP           2d7h   app=apinizer-portal
apinizer          manager                   NodePort    10.105.192.162   <none>        8080:32080/TCP           6d2h   app=manager
prod              cache-http-service        ClusterIP   10.97.132.189    <none>        8090/TCP                 6d2h   app=cache
prod              cache-hz-service          ClusterIP   10.111.184.42    <none>        5701/TCP                 6d2h   app=cache
prod              worker-http-service       NodePort    10.98.39.120     <none>        8091:30080/TCP           6d2h   app=worker

```


NodePort olarak çalışan 3 adet service var. 

Manager uygulaması 32080 portundan

![manager](../_images/apinizer_manager.png)

Portal uygulaması 32090 portundan

![portal](../_images/apinizer_portal.png)

Worker servisine de 30080 portundan ulaşabiliriz.

![worker](../_images/apinizer_worker.png)

#### Pods

```
kubectl get pods -n apinizer -l app=manager -owide

NAME                      READY   STATUS    RESTARTS   AGE     IP               NODE            NOMINATED NODE   READINESS GATES
manager-9c48f94f9-crfl7   1/1     Running   0          5d22h   10.240.209.197   sworkerappc02   <none>           <none>
```

```
kubectl get pods -n apinizer-portal -l app=apinizer-portal -owide

NAME                              READY   STATUS    RESTARTS   AGE    IP              NODE            NOMINATED NODE   READINESS GATES
apinizer-portal-7968db574-gfl9k   1/1     Running   0          2d3h   10.240.103.17   sworkerappc01   <none>           <none>
```

```
kubectl get pods -n prod -l app=worker -owide

NAME                     READY   STATUS    RESTARTS   AGE     IP               NODE            NOMINATED NODE   READINESS GATES
worker-797cd697f-l9bjq   1/1     Running   0          5d22h   10.240.209.200   sworkerappc02   <none>           <none>
worker-797cd697f-v7tpk   1/1     Running   0          5d22h   10.240.100.198   sworkerappc03   <none>           <none>

```

```
kubectl get pods -n prod -l app=cache -owide

NAME                    READY   STATUS    RESTARTS   AGE    IP               NODE            NOMINATED NODE   READINESS GATES
cache-ffccd776f-t5wdc   1/1     Running   0          6d2h   10.240.209.199   sworkerappc02   <none>           <none>
```

#### Secrets

```
kubectl get secret mongo-db-credentials -n apinizer  -o jsonpath={'.data.dbName'} | base64 -d

apinizerdb
```

```
kubectl get secret mongo-db-credentials -n apinizer  -o jsonpath={'.data.dbUrl'} | base64 -d

mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.105:25080,172.20.50.106:25080,172.20.50.107:25080/?authSource=admin&replicaSet=apinizer-replicaset
```



```
kubectl get secret mongo-db-credentials -n prod  -o jsonpath={'.data.dbName'} | base64 -d

apinizerdb
```

```
kubectl get secret mongo-db-credentials -n prod  -o jsonpath={'.data.dbUrl'} | base64 -d

mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.105:25080,172.20.50.106:25080,172.20.50.107:25080/?authSource=admin&replicaSet=apinizer-replicaset
```