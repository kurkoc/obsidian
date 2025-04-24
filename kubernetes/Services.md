Uygulamalarımızı çalıştırmak için deployment, replicaset gibi objeler kullandığımızda, bu objeler dinamik olarak Pod'lar oluşturabilir ve yok edebilir. Cluster üzerindeki bir deployment için, zamanın bir anından çalışır durumda olan pod kümesi, bir an sonra bu uygulamayı çalıştıran pod kümesinden farklı olabilir. Çünkü zamanla, pod üzerinde oluşabilecek herhangi bir hata, ana deployment/replicaset objesinin scale edilmesi veya farklı herhangi bir sebep yüzünden çalışan bazı podlar sonlandırılabilir veya bu podlara yenileri eklenebilir. Bu açıdan, erişmemiz gereken uygulamalar söz konusu olduğunda, bu uygulamalara podlar üzerinden erişebilmek imkansız hale gelir.


Service, bir veya birden fazla replica olarak çalışan bir pod grubuna, merkezi bir giriş noktası oluşturmak için kullanılan Kubernetes nesnesidir. Service objesi yardımıyla, selectorler kullanarak, ip adresi, isim gibi detaylarını bilmediğimiz bir grup pod'a erişebilmemiz sağlanır. Bu açıdan bir load balancer gibi de çalışır. Artık dışardan gelen bir istek, service objesine gelecektir ve service objesi bunu arka planda selector tanımına uyan podlara otomatik olarak yönlendirecektir. Kubernetes Servis nesneleri, herhangi bir Pod oluşturmaz. Sadece selector aracılığıyla eşleşen Pod'lara trafiği yönlendirir.

```
apiVersion: v1
kind: Namespace
metadata:
  name: whoami
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami-deployment
  namespace: whoami
spec:
  replicas: 5
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
      - name: whoami
        image: kurkoc/whoami:131
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
  namespace: whoami
spec:
  type: NodePort
  selector:
    app: whoami
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
```
### Service Türleri

#### ClusterIP

ClusterIP tipi, Kubernetes Service'leri için varsayılan tiptir. Service oluştururken `type` parametresini belirtmezseniz, otomatik olarak ClusterIP tipinde bir servis oluşturulur ve cluster IP havuzundan (`service-cluster-ip-range`) rastgele bir IP adresi atanır. 

```
kubectl cluster-info dump | grep service-cluster-ip-range
```

Örnek olarak, şu anki cluster'ım için bu değer 10.96.0.0/16 şeklinde.

Rastgele IP değilde bizim belirlediğimiz bir IP adresi atanmasını istersek `.spec.ClusterIP` alanına değer verebiliriz. Tabi yine üstteki CIDR'a uygun olmalıdır. Sabit IP verilmiş şekilde uygulamamızı kullanan ve konfigürasyonuna müdahale edemediğimiz legacy uygulamalar için ihtiyaç olabilir.

`.spec.ClusterIP` alanına "None" değeri vererek Kubernetes tarafından bir IP adresini atanmasının önüne geçebiliriz. Bu tarz servislere headless service denilir. Load balancing, discovery gibi mekanizmalara hiç dahil olmadan direk olarak pod üzerinden erişilebilir sadece. 

ClusterIP tipindeki Service'ler, genelde DB veya backend uygulamaları gibi doğrudan dışarıdan gelen trafiğe kapalı olup, yalnızca cluster'ın içindeki frontend uygulamaları gibi aracı uygulamalarla iletişime geçilen senaryolar için uygundur. Bu tür bir servis, yalnızca Kubernetes cluster'ı içindeki bir diğer kaynağın ana servisimizle iletişim kurmasını sağlamak için kullanılır.

![[clusterip.png]]

```
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
  namespace: whoami
spec:
  type: ClusterIP
  selector:
    app: whoami
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

```
kubectl get svc -n whoami -o wide

NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE     SELECTOR
whoami-service   ClusterIP   10.96.2.139   <none>        80/TCP    3m48s   app=whoami
```

#### NodePort

NodePort tipi Service kullanıldığında, Kubernetes cluster'ındaki her bir node üzerinde bir port cluster dışına açılır ve uygulamamıza cluster dışından erişmek isteyenler, herhangi bir node üzerindeki node port'u üzerinden uygulamaya erişir. 

![600](https://kerteriz.net/content/images/size/w1000/2023/05/kubernetes-service-3.png)

Örneğin yukarıdaki şemada, Node 1 veya Node 2'nin IP adresi üzerinden 30123 nolu port numarası ile Service'e ulaşabilirsiniz. Service isteğin hangi Node IP adresi üzerinden geldiğine bakmaksızın rastgele bir Pod'a trafiği yönlendirecektir.

NodePort servisi kullandığımızda, control plane `--service-node-port-range` değeri ile belirtilen aralıktan bir port numarası atar, varsayılan olarak 30000-32767'dir.  
Ancak, `.spec.ports[*].nodePort` değeri üzerinden, istediğiniz bir port numarasını da seçebilirsiniz. 

```
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
  namespace: whoami
spec:
  type: NodePort
  selector:
    app: whoami
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
```

```
kubectl get svc -n whoami -owide

NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE     SELECTOR
whoami-service   NodePort   10.96.130.241   <none>        80:30080/TCP   4h51m   app=whoami
```



#### LoadBalancer

Load balancer tipi serviceler, cloud providerlar için tasarlanmıştır. LoadBalancer tipinde bir servis oluşturduğumuzda, cloud provider dış dünyadan erişilebilecek bir public IP adresine sahip, bir LB oluşturur, sonrasında bu LB bizim service objesine ilişkilendirilir. Dolayısıyla bu public adrese ulaşan paketler içeriden pod 'lara yönlendirilir. NodePort'un önüne loadbalancer eklenmiş halidir denilebilir.

![loadbalancer](https://kerteriz.net/content/images/2023/05/kubernetes-service-7.png)


#### ExternalName

ExternalName türündeki servisler, bir servisi selectorler üzerinden bir pod grubuyla değil de, bir DNS adıyla eşler.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```



---

```
kubectl port-forward -n whoami service/whoami-service 8070:80

Forwarding from 127.0.0.1:8070 -> 8080
Forwarding from [::1]:8070 -> 8080
Handling connection for 8070
```
