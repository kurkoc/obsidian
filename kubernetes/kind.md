```
kind create cluster
```


```
kind get clusters

kind
```

```
sudo kind delete cluster --name kind
```


```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker

```

```
sudo kind create cluster --name kurkoc-cluster --config kind-config.yaml

Creating cluster "kurkoc-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.32.2) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kurkoc-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-kurkoc-cluster

Thanks for using kind! 😊
```


kind bizim için 1'i master, 2'si worker olan 3 node'lu bir cluster oluşturdu.

```
kubectl get nodes -o wide

NAME                           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
kurkoc-cluster-control-plane   Ready    control-plane   10m   v1.32.2   172.20.0.2    <none>        Debian GNU/Linux 12 (bookworm)   6.10.14-linuxkit   containerd://2.0.2
kurkoc-cluster-worker          Ready    <none>          10m   v1.32.2   172.20.0.3    <none>        Debian GNU/Linux 12 (bookworm)   6.10.14-linuxkit   containerd://2.0.2
kurkoc-cluster-worker2         Ready    <none>          10m   v1.32.2   172.20.0.4    <none>        Debian GNU/Linux 12 (bookworm)   6.10.14-linuxkit   containerd://2.0.2
```

Eğer istersek, cluster ile bir port bağlantısı kurabiliriz. 

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 30080
- role: worker
- role: worker
```


