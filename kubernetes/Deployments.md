Kubernetes tarafında ilgilenilen en ufak çalışır çıktı pod olduğundan belki de en önemli component poddur diyebiliriz. Ancak production ortamlarda biz pod tanımlarımızı, genellikle pod komutları üzerinden yapmayız. Replikasyonu ve otomatizasyonu sağlamak adına Kubernetes daha üst seviye componentler sunar. Bu componentlerden, en sık kullanılanı ise deployment componentidir.

Deployment;

- Belirli bir uygulama sürümünü çalıştırmak için, verdiğimiz replica miktarınca bir dizi Pod oluşturulmasını ve bu podların sürekli olarak çalışmasını sağlar.
- Herhangi bir pod üzerinde hata oluşursa, Deployment bunu algılar ve sorunlu olan pod yerine otomatik olarak yeni bir pod oluşturulur ve desired state ve current state eşitlenir.
- Uygulama dağıtımı sırasında sürüm yönetimi yapılmasına, uygulamanın hızlı bir şekilde yeniden dağıtılmasına ve çeşitli ölçeklendirme stratejilerinin kullanılmasına da olanak tanır. Örneğin, uygulamanızın yeni bir sürümünü yayınlamak istediğinizde, Deployment stratejilerini kullanarak güncellemeyi kontrollü bir şekilde gerçekleştirebilirsiniz. Bu stratejiler, yavaş yavaş güncelleme, aynı anda tümünü güncelleme veya sıfır kesintiyle güncelleme gibi farklı yaklaşımları destekler.
- Ölçeklendirme işlevselliği sağlar. Uygulamanız yoğun trafik altında daha fazla kaynak talep ederse, Deployment onu otomatik olarak ölçeklendirerek yükü dengeleyebilir. Böylece, uygulamanızın performansı ve kullanılabilirliği artar.

Deployment'lar, bir ReplicaSet nesnesi oluşturarak çalışırlar. ReplicaSet, belirtilen sayıda pod'un oluşturulmasını ve çalıştırılmasını sağlar.

![](https://kerteriz.net/content/images/2023/06/kubernetes-deployment-1.png)


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```


Üstteki tanıma göre; adı `nginx-deployment` olan bir deployment objesi oluşturduk. `.spec.replicas` alanında belirttiğimiz üzere bizim için 3 adet pod oluşturacak ve bu podların içeriğinde, `.spec.template.spec` alanında belirttiğimiz konfigürasyonlarla image olarak verdiğimiz container çalıştırılacak. Bu oluşturulan podlara label olarak `app:nginx` verilecek.

`.spec.selector.matchLabels` alanında ise, bu deployment üzerinde pod'ları etkileyecek herhangi bir işlem yapıldığında(scaling olabilir), arka tarafta hangi pod'ları seçeceğini belirtiyoruz. biz app:nginx label'ına sahip podları istiyoruz. 


```
kubectl describe deployment nginx-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           2m4s
controlplane:~$ kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 22 Apr 2025 12:57:24 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:         nginx:1.14.2
    Port:          80/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-647677fc66 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m12s  deployment-controller  Scaled up replica set nginx-deployment-647677fc66 from 0 to 3

```

Deployment manifestimizi çalıştırdıktan sonra, cluster'ımızda bulunan replicaset'lere bakacak olursak eğer; 

```
kubectl get rs -owide

NAME                          DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-deployment-647677fc66   3         3         3       27m   nginx        nginx:1.14.2   app=nginx,pod-template-hash=647677fc66
```

`metadata.name` alanında belirttiğimiz deployment adını baz alarak bir replicaset oluşturulduğunu görebiliriz.

`.metadata.labels` alanında verdiğimiz label bilgisi eklenmiş pod'ların oluştuğunu görebiliriz. 

```
kubectl get pods --show-labels

NAME                                READY   STATUS    RESTARTS   AGE   LABELS
nginx-deployment-647677fc66-fxfgx   1/1     Running   0          21m   app=nginx,pod-template-hash=647677fc66
nginx-deployment-647677fc66-ns5bd   1/1     Running   0          21m   app=nginx,pod-template-hash=647677fc66
nginx-deployment-647677fc66-vd9rf   1/1     Running   0          21m   app=nginx,pod-template-hash=647677fc66
```


Oluşan Replicaset'e ve replicaset tarafından oluşturulan pod'lara bakarsak eğer; bizim verdiğimiz app=nginx label'ının dışında  pod-template-hash=647677fc66 şeklinde bir label daha olduğunu görüyoruz.

`pod-template-hash` label'ı, deployment controller tarafından bir deployment'un oluşturduğu replicaset'e eklenir. Bu label ile bir deployment'in oluşturduğu replicaset'lerin çakışmaması sağlanır. Bu değer, Deployment tanımındaki PodTemplate'in hashlenmesi ile oluşturulur.

#### Rollout
Kubernetes ortamında bir Deployment'ın rollout olması, o Deployment altında tanımlanmış uygulamanın yeni bir sürümünün (örneğin yeni bir container image'ı, yeni bir config değişikliği vb.) aşamalı olarak güncellenmesi süreci anlamına gelir. Rollout işlemi yalnızca, bir deployment'ın pod template(`.spec.template`) alanında bir değişiklik olursa gerçekleşir. Scaling gibi işlemler rollout tetiklemez.

##### Rollout sürecinde ne olur?

1. Yeni konfigürasyonla yeni bir ReplicaSet oluşturulur.
2. Yeni ReplicaSet'e göre yeni Pod'lar başlatılır.
3. Yeni Pod'lar hazır oldukça, eski versiyondaki Pod'lar yavaş yavaş devre dışı bırakılır.
4. Bu süreçte sistem kesintiye uğramadan, kullanıcılar hizmet almaya devam eder.

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```



```
kubectl get deployment whoami-deployment -n whoami -o yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"whoami-deployment","namespace":"whoami"},"spec":{"replicas":5,"selector":{"matchLabels":{"app":"whoami"}},"template":{"metadata":{"labels":{"app":"whoami"}},"spec":{"containers":[{"image":"kurkoc/whoami:131","name":"whoami","ports":[{"containerPort":8080}]}]}}}}
  creationTimestamp: "2025-04-24T08:25:03Z"
  generation: 1
  name: whoami-deployment
  namespace: whoami
  resourceVersion: "5670"
  uid: 4435de2e-dfbe-4087-8af1-634266d0a4df
spec:
  progressDeadlineSeconds: 600
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: whoami
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: whoami
    spec:
      containers:
      - image: kurkoc/whoami:131
        imagePullPolicy: IfNotPresent
        name: whoami
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 5
  conditions:
  - lastTransitionTime: "2025-04-24T08:25:04Z"
    lastUpdateTime: "2025-04-24T08:25:04Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2025-04-24T08:25:03Z"
    lastUpdateTime: "2025-04-24T08:25:04Z"
    message: ReplicaSet "whoami-deployment-5786548f4d" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 5
  replicas: 5
  updatedReplicas: 5

```