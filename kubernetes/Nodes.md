Node'lar, cluster içerisinde yer alan ve kubernetes tarafından soyutlanarak sunulan, sanal veya fiziksel bir makinelerdir. Belirli fiziksel veya sanal makineleri yönetmek yerine, her bir node'u belirli bir havuzda, container'ları ve container'ları yöneten servisleri çalıştıran CPU ve RAM kaynakları olarak ele alabiliriz.

Kubernetes, container’ları node üzerinde bulunan podlara yerleştirerek çalıştırır. Her node, birden fazla pod'a sahip olabilir ve pod'ların içinde çalışan container'lar vardır. Her node, control plane tarafından yönetilir ve pod'ları çalıştırmak için gerekli servisleri içerir. Bir cluster, birkaç node’dan oluşabilir; test veya veya kaynakların sınırlı olduğu bir ortamda yalnızca bir node bile olabilir.


```
kubectl get nodes -o wide

NAME            STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
smasterappc01   Ready    control-plane   40h   v1.31.0   172.20.50.105   <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
smasterappc02   Ready    control-plane   40h   v1.31.0   172.20.50.106   <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
smasterappc03   Ready    control-plane   40h   v1.31.0   172.20.50.107   <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
sworkerappc01   Ready    <none>          40h   v1.31.0   172.20.50.95    <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
sworkerappc02   Ready    <none>          40h   v1.31.0   172.20.50.96    <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
sworkerappc03   Ready    <none>          40h   v1.31.0   172.20.50.97    <none>        Ubuntu 24.04.1 LTS   6.8.0-51-generic   containerd://1.7.25
```

Bir node üzerinde;
- kubelet
- kube-proxy
- CRI
bulunur.

```
kubectl describe node smasterappc01

Name:               smasterappc01
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=smasterappc01
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 172.20.50.105/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.240.25.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 16 Apr 2025 16:50:26 +0300
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  smasterappc01
  AcquireTime:     <unset>
  RenewTime:       Fri, 18 Apr 2025 09:55:59 +0300
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 16 Apr 2025 16:53:10 +0300   Wed, 16 Apr 2025 16:53:10 +0300   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Fri, 18 Apr 2025 09:52:57 +0300   Wed, 16 Apr 2025 16:50:25 +0300   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Fri, 18 Apr 2025 09:52:57 +0300   Wed, 16 Apr 2025 16:50:25 +0300   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Fri, 18 Apr 2025 09:52:57 +0300   Wed, 16 Apr 2025 16:50:25 +0300   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Fri, 18 Apr 2025 09:52:57 +0300   Wed, 16 Apr 2025 16:53:02 +0300   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  172.20.50.105
  Hostname:    smasterappc01
Capacity:
  cpu:                8
  ephemeral-storage:  102626232Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8130000Ki
  pods:               110
Allocatable:
  cpu:                8
  ephemeral-storage:  94580335255
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             8027600Ki
  pods:               110
System Info:
  Machine ID:                 cd31a309f6fc406d98d0cb93ffe9146d
  System UUID:                1f2d0e42-700f-dc3f-7b70-7680161c6c8d
  Boot ID:                    932d8a85-c340-4d17-b5d3-e8c5eb4698dd
  Kernel Version:             6.8.0-51-generic
  OS Image:                   Ubuntu 24.04.1 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.7.25
  Kubelet Version:            v1.31.0
  Kube-Proxy Version:         
PodCIDR:                      10.240.0.0/24
PodCIDRs:                     10.240.0.0/24
Non-terminated Pods:          (6 in total)
  Namespace                   Name                                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                     ------------  ----------  ---------------  -------------  ---
  kube-system                 calico-node-gcmlv                        250m (3%)     0 (0%)      0 (0%)           0 (0%)         41h
  kube-system                 etcd-smasterappc01                       100m (1%)     0 (0%)      100Mi (1%)       0 (0%)         41h
  kube-system                 kube-apiserver-smasterappc01             250m (3%)     0 (0%)      0 (0%)           0 (0%)         41h
  kube-system                 kube-controller-manager-smasterappc01    200m (2%)     0 (0%)      0 (0%)           0 (0%)         41h
  kube-system                 kube-proxy-bml56                         0 (0%)        0 (0%)      0 (0%)           0 (0%)         41h
  kube-system                 kube-scheduler-smasterappc01             100m (1%)     0 (0%)      0 (0%)           0 (0%)         41h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                900m (11%)  0 (0%)
  memory             100Mi (1%)  0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:              <none>

```

### Node componentleri

#### kubelet

Kubelet, node üzerinde çalışan ve cluster'daki control plane ile iletişim kuran agenttir diyebiliriz. Control plane'in node'u izlemesine, üzerinde ne çalıştırdığını görmesine ve container runtime'ına talimatlar göndermesine olanak tanır.

Kubelet, çeşitli mekanizmalar aracılığıyla sağlanan bir dizi PodSpec'i alır ve bu PodSpec'lerde açıklanan container'ların çalışır durumda ve sağlıklı olmasını sağlar. Kubernetes bir pod'u belirli bir node üzerinde schedule etmek istediğinde pod'un PodSec'lerini kubelet'e gönderir.Kubelet, PodSpec'lerde belirtilen container'ın ayrıntılarını okur, image hub'ından image'ı çeker ve container'ı çalıştırır.

#### kube-proxy

Node üzerinde çalışan network proxy’sidir. Node üzerindeki network kurallarını belirler ve belirlenen bu kurallara göre cluster içindeki veya dışındaki nesneler ile Pod arasındaki ağ iletişimini sağlar.

#### container runtime

Kubernetes ortamında containerların çalıştırılmasını ve yaşam döngüsünü yönetmekten sorumludur.Aslında kubernetes container'ları durdurma ve başlatma gibi yaşam döngüsü süreçlerini yönetmeyi doğrudan üstlenmez. Kubelet, container runtime interface (CRI) destekleyen herhangi bir container runtime'ı ile Kubernetes cluster'ının ihtiyaçlarına göre talimatlar verir.

Kubernetes, containerd, CRI-O gibi container runtimelarını ve Kubernetes CRI (Container Runtime Interface) implemente eden runtime'ları destekler.



### Bazı komutlar

Node'ları listelerken labellarını da getirmek istersek
```
kubectl get nodes --show-labels
```

Node'ları listelerken daha fazla kolon
```
kubectl get nodes --output=wide
```

Bir node'a yeni bir label eklemek istersek
```
kubectl label nodes smasterappc01 disktype=ssd
```

