ReplicaSet, belirtilen sayıda pod'un daima çalışmasını sağlamak için kullanılan bir Kubernetes nesnesidir. ReplicaSet, özellikle high-availability gerektiren uygulamalar için kullanılır. Belirtilen sayıda pod oluşturarak uygulamaların yüksek kullanılabilirliğini sağlar. Belirtilen sayıda pod oluşturduktan sonra, bunların düzgün olarak çalıştığından da emin olur. Bu açıdan, pod'larda oluşabilecek hatalar veya arızalar durumunda, ReplicaSet yeni bir pod oluşturarak uygulamanın arzulanan şekilde çalışmasını sağlar.


```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx
```

replicaset tanımımızı apply ettikten sonra bilgilerine bakarsak eğer;

```
kubectl describe rs/my-replicaset


Name:         my-replicaset
Namespace:    default
Selector:     app=my-app
Labels:       <none>
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=my-app
  Containers:
   my-container:
    Image:         nginx
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  5m13s  replicaset-controller  Created pod: my-replicaset-2mgdt
  Normal  SuccessfulCreate  5m13s  replicaset-controller  Created pod: my-replicaset-pwvvd
  Normal  SuccessfulCreate  5m13s  replicaset-controller  Created pod: my-replicaset-6fpvv

```


```
kubectl get pods

NAME                  READY   STATUS    RESTARTS   AGE
my-replicaset-2mgdt   1/1     Running   0          34s
my-replicaset-6fpvv   1/1     Running   0          34s
my-replicaset-pwvvd   1/1     Running   0          34s
```

pod'ların detayına bakarsak eğer; `metadata.ownerReference` alanında mevcut replicaset'in bilgisini görebiliriz.

```
kubectl get pod my-replicaset-2mgdt -o yaml

...
metadata:
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: my-replicaset
    uid: 17c72657-381c-4c4f-9e6d-d9cf7a90b947
...

```

