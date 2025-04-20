Deployment, ReplicaSet gibi Kubernetes objeleri, bir pod'un, Kubernetes üzerindeki nodelarda belirlenen replika sayısı kadar çalışmasını garanti ederler. Fakat hangi node üzerinde kaç tanesinin yer alacağına dair herhangi bir garanti sunmazlar. 

DaemonSet ise belirli bir podu, her bir node üzerinde bir daemon gibi çalıştırmamızı garanti eden Kubernetes objesidir. Bu açıdan bakıldığında, DaemonSet tanımı replica sayısı parametresi içermez.

![daemonset](https://kerteriz.net/content/images/2023/05/kubernetes-daemonset-nedir-1.png)
 

DaemonSet'ler genellikle;
- Node monitoring sistemleri
- Node üzerinden log toplama 
- Node backup işlemleri
- Security veya networking gibi tool'lar

gibi her node üzerinde kesin olarak çalışacağını bilmek isteyeceğimiz uygulama senaryolarında kullanılır.

#### DaemonSet Oluşturmak

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
        - name: fluentd-elasticsearch
          image: quay.io/fluentd_elasticsearch/fluentd:latest
```


> [!Not -1]
> Kubernetes cluster'ımızdaki node, `unschedulable` olarak belirlense dahi daemonset bildirimi üzerinden pod ataması yapılır.

> [!Not 2]
> Bir daemonset tanımı yaptıktan sonra cluster'a yeni bir node eklenirse, daemonset bildirimindeki pod otomatik olarak yeni pod üzerinde de oluşturulur. Node silindiğinde ise üzerindeki pod silinir ve başka bir node üzerinde oluşturulmaz.


```
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```

#### NodeSelector ile kapsamı daraltma

Üstte daemonset'ın davranışından bahsederken, talep edilen pod'un tüm nodelar üzerinde oluşturulacağını söylemiştik ancak bu konuda bir nüans var aslında. DaemonSet tanımlarımızda, **`nodeSelector`** parametresini yapılandırarak her node yerine belirli koşulları sağlayan tüm node'larda pod oluşturmasını sağlayabiliriz.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      nodeSelector:
        - log-collection-enabled: "true"
      containers:
        - name: fluentd-elasticsearch
          image: quay.io/fluentd_elasticsearch/fluentd:latest
```


Yukardaki tanımlama doğrultusunda mesela, sadece log-collection-enabled: true olarak label belirlediğimiz node'lar üzerinde fluentd çalıştırılacak. Böylece kapsam dışı bıraktığımız node'lar haricinde bütün node'lar üzerinde ilgili pod oluşturulacak.

![](https://kerteriz.net/content/images/2023/05/kubernetes-daemonset-nedir-2.png)


Monitoring işlemi, sadece ssd disk içeren node'lar için gerçekleştirilecek.


```
kubectl get ds -A

NAMESPACE     NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   calico-node   6         6         6       6            6           kubernetes.io/os=linux   2d
kube-system   kube-proxy    6         6         6       6            6           kubernetes.io/os=linux   2d

```




