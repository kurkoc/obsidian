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
kubectl set image deployment/whoami-deployment whoami=kurkoc/whoami:136

deployment.apps/whoami-deployment image updated
```

Bu şekilde direk image'i update edebileceğimiz gibi;

```
kubectl edit deployment -n whoami whoami-deployment
```

diyerek default text editöründe açılan dosya üzerinde güncelleme yaparak da rollout işlemini sağlayabiliriz.


```
kubectl rollout status deployment/whoami-deployment -n whoami

deployment "whoami-deployment" successfully rolled out
```



Şimdi replicaset'lere bakarsak eğer;

```
kubectl get rs -n whoami

NAME                           DESIRED   CURRENT   READY   AGE
whoami-deployment-5786548f4d   0         0         0       4d5h
whoami-deployment-f564f7fc5    5         5         5       4m25s
```

Eski replicaset'teki bütün pod'lar sonlandırılırken, yeni oluşturulan replicaset'te 5 yeni pod oluşturulduğunu görebiliriz.


```
kubectl describe deployments -n whoami whoami-deployment

Name:                   whoami-deployment
Namespace:              whoami
CreationTimestamp:      Thu, 24 Apr 2025 11:25:03 +0300
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=whoami
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=whoami
  Containers:
   whoami:
    Image:         kurkoc/whoami:136
    Port:          8080/TCP
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
OldReplicaSets:  whoami-deployment-5786548f4d (0/0 replicas created)
NewReplicaSet:   whoami-deployment-f564f7fc5 (5/5 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set whoami-deployment-f564f7fc5 from 0 to 2
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled down replica set whoami-deployment-5786548f4d from 5 to 4
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set whoami-deployment-f564f7fc5 from 2 to 3
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled down replica set whoami-deployment-5786548f4d from 4 to 3
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set whoami-deployment-f564f7fc5 from 3 to 4
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled down replica set whoami-deployment-5786548f4d from 3 to 2
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set whoami-deployment-f564f7fc5 from 4 to 5
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled down replica set whoami-deployment-5786548f4d from 2 to 1
  Normal  ScalingReplicaSet  10m   deployment-controller  Scaled down replica set whoami-deployment-5786548f4d from 1 to 0

```

Burada da `events` kısmında eski replicaset'in giderek scale down olduğunu, yeni replicaset'in ise scale up olduğunu görebiliriz. Yine `OldReplicaSets` ve `NewReplicaSet` kısmında da eski ve yeni replicaset'lerin isimlerini görebiliriz.