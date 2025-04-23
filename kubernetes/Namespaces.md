Namespace, cluster içerisindeki Kubernetes objelerini görevlerine ya da hizmet verdikleri alana göre sınıflandırarak, izolasyon için bir mekanizma sağlar. Birden fazla ekibe veya projeye yayılmış, çok sayıda kullanıcının bulunduğu ortamlarda kullanılmak üzere tasarlanmıştır. Örnek bir senaryo olarak, bir firma içindeki farklı ekipler kendi Namespaces'lerinde çalışabilir ve birbirlerinin kaynaklarını etkilemeden kendi uygulamalarını yönetebilirler. Aynı şekilde, farklı ortamlar (örneğin, dev, prod) için farklı Namespaces'ler oluşturarak bu ortamlar arasında izolasyon sağlanabilir.

Kubernetes üzerindeki objelerden bazıları namespace kullanarak gruplandırılabilirken, bazıları ise tüm cluster genelinde geçerli objelerdir. Namespace kullanabileceğimiz objeleri görebilmek için;

```
kubectl api-resources --namespaced=true

NAME                        SHORTNAMES   APIVERSION                     NAMESPACED   KIND
bindings                                 v1                             true         Binding
configmaps                  cm           v1                             true         ConfigMap
endpoints                   ep           v1                             true         Endpoints
events                      ev           v1                             true         Event
limitranges                 limits       v1                             true         LimitRange
persistentvolumeclaims      pvc          v1                             true         PersistentVolumeClaim
pods                        po           v1                             true         Pod
podtemplates                             v1                             true         PodTemplate
replicationcontrollers      rc           v1                             true         ReplicationController
resourcequotas              quota        v1                             true         ResourceQuota
secrets                                  v1                             true         Secret
serviceaccounts             sa           v1                             true         ServiceAccount
services                    svc          v1                             true         Service
controllerrevisions                      apps/v1                        true         ControllerRevision
daemonsets                  ds           apps/v1                        true         DaemonSet
deployments                 deploy       apps/v1                        true         Deployment
replicasets                 rs           apps/v1                        true         ReplicaSet
statefulsets                sts          apps/v1                        true         StatefulSet
localsubjectaccessreviews                authorization.k8s.io/v1        true         LocalSubjectAccessReview
horizontalpodautoscalers    hpa          autoscaling/v2                 true         HorizontalPodAutoscaler
cronjobs                    cj           batch/v1                       true         CronJob
jobs                                     batch/v1                       true         Job
leases                                   coordination.k8s.io/v1         true         Lease
networkpolicies                          crd.projectcalico.org/v1       true         NetworkPolicy
networksets                              crd.projectcalico.org/v1       true         NetworkSet
endpointslices                           discovery.k8s.io/v1            true         EndpointSlice
events                      ev           events.k8s.io/v1               true         Event
ingresses                   ing          networking.k8s.io/v1           true         Ingress
networkpolicies             netpol       networking.k8s.io/v1           true         NetworkPolicy
poddisruptionbudgets        pdb          policy/v1                      true         PodDisruptionBudget
rolebindings                             rbac.authorization.k8s.io/v1   true         RoleBinding
roles                                    rbac.authorization.k8s.io/v1   true         Role
csistoragecapacities                     storage.k8s.io/v1              true         CSIStorageCapacity

```


Kubernetes cluster'larında aşağıdaki namespace'ler başlangıçta otomatik olarak gelir;
- default
- kube-system
- kube-public
- kube-node-lease

`kubectl` ile çalıştırdığımız komutlar otomatik olarak default namespace'i üzerinde çalışır.

```
kubectl get pods 

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          28m
```

namespace'leri dikkate almadan bütün namespace'lerde yer alan objeleri görmek istersek eğer;

```
kubectl get pods -A

NAMESPACE            NAME                                      READY   STATUS    RESTARTS      AGE
default              nginx-pod                                 1/1     Running   0             29m
kube-system          calico-kube-controllers-fdf5f5495-dgc76   1/1     Running   2 (81m ago)   30d
kube-system          canal-9hc7x                               2/2     Running   2 (81m ago)   30d
kube-system          canal-b5cnm                               2/2     Running   2 (82m ago)   30d
kube-system          coredns-7695687499-2vdd4                  1/1     Running   1 (82m ago)   30d
kube-system          coredns-7695687499-ltw2v                  1/1     Running   1 (82m ago)   30d
kube-system          etcd-controlplane                         1/1     Running   3 (81m ago)   30d
kube-system          kube-apiserver-controlplane               1/1     Running   2 (81m ago)   30d
kube-system          kube-controller-manager-controlplane      1/1     Running   2 (81m ago)   30d
kube-system          kube-proxy-f7jnk                          1/1     Running   2 (81m ago)   30d
kube-system          kube-proxy-fbkjh                          1/1     Running   1 (82m ago)   30d
kube-system          kube-scheduler-controlplane               1/1     Running   2 (81m ago)   30d
local-path-storage   local-path-provisioner-5c94487ccb-bm6vp   1/1     Running   2 (81m ago)   30d
```

ya da namespace belirterek spesifik bir namespace altındaki objeleri görmek istersek eğer;

```
kubectl get pods --namespace default

kubectl get pods -n default
```


kubectl üzerindeki varsayılan davranış olan default namespace'i üzerinde arama değiştirilebilir.

```
kubectl config set-context --current --namespace=kube-system 

Context "kubernetes-admin@kubernetes" modified.
```

Artık namespace belirtmeden çalıştırılan komut çağrıları kube-system namespace'i altında çalışacak.

```
kubectl get pods 

NAME                                      READY   STATUS    RESTARTS      AGE
calico-kube-controllers-fdf5f5495-dgc76   1/1     Running   2 (87m ago)   30d
canal-9hc7x                               2/2     Running   2 (87m ago)   30d
canal-b5cnm                               2/2     Running   2 (87m ago)   30d
coredns-7695687499-2vdd4                  1/1     Running   1 (87m ago)   30d
coredns-7695687499-ltw2v                  1/1     Running   1 (87m ago)   30d
etcd-controlplane                         1/1     Running   3 (87m ago)   30d
kube-apiserver-controlplane               1/1     Running   2 (87m ago)   30d
kube-controller-manager-controlplane      1/1     Running   2 (87m ago)   30d
kube-proxy-f7jnk                          1/1     Running   2 (87m ago)   30d
kube-proxy-fbkjh                          1/1     Running   1 (87m ago)   30d
kube-scheduler-controlplane               1/1     Running   2 (87m ago)   30d
```

Bu bilgiye context bilgisine bakarak ulaşabiliriz.

```
kubectl config view 

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.30.1.2:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: kube-system
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```

