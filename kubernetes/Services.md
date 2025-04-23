Service, bir veya birden fazla replica olarak çalışan bir pod grubuna, merkezi bir giriş noktası oluşturmak için kullanılan Kubernetes nesnesidir. Deployment, replicaset gibi birden fazla pod oluşturabileceğimiz objeleri kullanarak, kendisine ait IP adresi olan çok sayıda pod oluşturulabilir. Ancak zamanla, pod üzerinde oluşabilecek herhangi bir hata, ana deployment/replicaset objesinin scale edilmesi veya farklı herhangi bir sebep yüzünden çalışan bazı podlar sonlandırılabilir veya bu podlara yenileri eklenebilir. Bu açıdan, erişmemiz gereken uygulamalar söz konusu olduğunda bunu podlar üzerinden takip edebilmemiz zamanla imkansız hale gelir.

Service objesi yardımıyla, selectorler kullanarak, ip adresi, isim gibi detaylarını bilmediğimiz bir grup pod'a erişebilmemiz sağlanır. Bu açıdan bir load balancer gibi de çalışır. Artık dışardan gelen bir istek, service objesine gelecektir ve service objesi bunu arka planda selector tanımına uyan podlara otomatik olarak yönlendirecektir.


### Service Türleri

#### ClusterIP

ClusterIP tipi, Kubernetes Service'leri için varsayılan tiptir. Service oluştururken `type` parametresini belirtmezseniz, otomatik olarak ClusterIP tipinde bir servis oluşturulur.

ClusterIP tipindeki Service'ler, genelde DB veya backend uygulamaları gibi doğrudan dışarıdan gelen trafiğe kapalı olup, yalnızca cluster'ın içindeki frontend uygulamaları gibi aracı uygulamalarla iletişime geçilen senaryolar için uygundur. Bu tür bir servis, yalnızca Kubernetes cluster'ı içindeki bir diğer kaynağın ana servisimizle iletişim kurmasını sağlamak için kullanılır.

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
kubectl port-forward -n whoami service/whoami-service 8070:80

Forwarding from 127.0.0.1:8070 -> 8080
Forwarding from [::1]:8070 -> 8080
Handling connection for 8070
```



