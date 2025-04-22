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
kubectl get pods -A -l app -L app

NAMESPACE         NAME                              READY   STATUS    RESTARTS   AGE     APP
apinizer-portal   apinizer-portal-7968db574-gfl9k   1/1     Running   0          8m1s    apinizer-portal
apinizer          manager-9c48f94f9-crfl7           1/1     Running   0          3d19h   manager
prod              cache-ffccd776f-t5wdc             1/1     Running   0          3d19h   cache
prod              worker-797cd697f-l9bjq            1/1     Running   0          3d19h   worker
prod              worker-797cd697f-v7tpk            1/1     Running   0          3d19h   worker

```
 
`-l` ile filtreleme yaparken, `-L` ile de verdiğimiz label değerinin liste sonuç kolonları arasında gösterilmesini sağladık.


```
kubectl get pods -l "env in (dev,stage)" 

kubectl get pods -l 'environment,environment notin (frontend)'
```


#### Tavsiye Edilen Label'lar

Kubernetes üzerinde nesneleri takip edebilmemize yarayan, pek çok monitoring ve dashboard uygulaması mevcuttur. Ortak bir label kümesi kullanmak, nesneleri tüm araçların anlayabileceği ortak bir kimlik halinde sunmamıza yardımcı olabilir.

| Key                            | Description                                                                                                             | Example        | Type   |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | -------------- | ------ |
| `app.kubernetes.io/name`       | The name of the application                                                                                             | `mysql`        | string |
| `app.kubernetes.io/instance`   | A unique name identifying the instance of an application                                                                | `mysql-abcxyz` | string |
| `app.kubernetes.io/version`    | The current version of the application (e.g., a [SemVer 1.0](https://semver.org/spec/v1.0.0.html), revision hash, etc.) | `5.7.21`       | string |
| `app.kubernetes.io/component`  | The component within the architecture                                                                                   | `database`     | string |
| `app.kubernetes.io/part-of`    | The name of a higher level application this one is part of                                                              | `wordpress`    | string |
| `app.kubernetes.io/managed-by` | The tool being used to manage the operation of an application                                                           | `Helm`         | string |

Label'ların Kubernetes üzerinde bir diğer ve daha önemli kullanım şekli ise; service, deployment gibi nesneler hedefledikleri pod kümesini bir label seçici üzerinden tanımlar.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

Service ve ReplicationController gibi objeler sadece üstteki gibi equality-based selector'u kullanabilirken. Job, Deployment, ReplicaSet, DaemonSet gibi objeler set-based selectorleri de desteklerler.

```yaml
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - { key: tier, operator: In, values: [cache] }
    - { key: environment, operator: NotIn, values: [dev] }
```

