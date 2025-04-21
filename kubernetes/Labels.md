Label; bir Kubernetes objesine (pod, replica set veya service gibi) key-value şeklinde tanımlamalar eklemek için kullanılan bir etiketleme mekanizmasıdır. Labellar, nesneleri tanımlayan key-value çiftleridir. Labellar aracılığıyla, nesneleri gruplayabilir, belirli özelliklere göre sıralayabilir ve diğer nesnelerle ilişkilendirmeler yapabiliriz. 

Label tanımlarında kullanacağımız değerler, Kubernetes alt sistemleri için bir şey ifade etmemesine ragmen objeleri filtrelerken ya da seçerken tarafımızdan gruplayıcı ve ayırt edici componentler oluşturmak icin kullanılır. 

Kubernetes üzerinde objeler arası bağlantı da label'lar sayesinde kurulur. Service ve deployment objeleri, hangi pod'ların kendilerine ait olduğunu ve hangi pod'lar ile ilişki kuracaklarını label'lar sayesinde belirler.


![labels](https://kerteriz.net/content/images/2023/05/kubernetes-pod-10.png)

Label'lar aracılığıyla, daha anlaşılır ve yönetilebilir bir Kubernetes cluster'ımız olur.

> [!not] Labellar, objelere oluşturma sırasında eklenebildiği gibi, daha sonra herhangi bir zamanda da eklenebilir veya değiştirilebilir. 


```
apiVersion: v1
kind: Pod
metadata:
 name: pod-with-labels
 labels:
  scope: backend
  env: prod
spec:
 containers:
 - image: nginx
   name: fe
```

```
kubectl run custom-nginx --image=nginx --labels=team=backend,env=prod
```


```
kubectl get pods --show-labels 

NAME              READY   STATUS    RESTARTS   AGE   LABELS
pod-with-labels   1/1     Running   0          11s   env=prod,scope=backend
```

Yeni bir label eklemek istersek eğer;

```
kubectl label pod pod-with-labels team=payment 

pod/pod-with-labels labeled
```

```
kubectl get pods --show-labels

NAME              READY   STATUS    RESTARTS   AGE    LABELS
pod-with-labels   1/1     Running   0          110s   env=prod,scope=backend,team=payment
```

Var olan bir label'ı güncellemek istersek eğer;

```
kubectl label pod pod-with-labels team=order --overwrite

pod/pod-with-labels labeled
```

```
kubectl get pods --show-labels

NAME              READY   STATUS    RESTARTS   AGE    LABELS
pod-with-labels   1/1     Running   0          4m1s   env=prod,scope=backend,team=order
```


Var olan bir label'ı silmek istersek eğer;

```
kubectl label pod pod-with-labels team-                 

pod/pod-with-labels unlabeled
```

```
kubectl get pods --show-labels

NAME              READY   STATUS    RESTARTS   AGE   LABELS
pod-with-labels   1/1     Running   0          11m   env=prod,scope=backend
```

Filtreleme yaparken label değerlerine göre filtrelemeler yapabiliriz.

```
kubectl get pods -l "env" => env labelina sahip podlar 

kubectl get pods -l "!env" => env labelina sahip olmayan podlar

kubectl get pods -l "env=prod,team=backend" => env label değeri prod ve team label değeri backend olanlar 

kubectl get pods -l "env=prod,team!=backend" => env label değeri prod ve team label değeri backend olmayanlar 

kubectl get pods -l "env,team=backend" => herhangi bir env labeli olan ve team label değeri backend olanlar

```

```
kubectl get pods -l "env in (dev,stage)" 

kubectl get pods -l 'environment,environment notin (frontend)'
```


#### Tavsiye Edilen Label'lar

Kubernetes üzerinde nesneleri takip edebilmemize yarayan, pek çok monitoring ve dashboard uygulaması mevcuttur. Ortak bir label kümesi kullanmak, nesneleri tüm araçların anlayabileceği ortak bir kimlik halinde sunmamıza yardımcı olabilir.