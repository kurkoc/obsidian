#### Phase
##### Pending
Pod, Kubernetes cluster'ı tarafından kabul edilmiştir, ancak container'lardan biri veya daha fazlasıçalışmaya hazır hale getirilmemiştir. Bir pod'un schedule edilmeyi beklerken ve container image'ları indirmek için geçen süreyi de kapsar.

##### Running
Pod bir node'a atanmıştır ve üzerindeki en az bir konteyner çalışıyor durumdadır.

##### Succeeded
Pod üzerindeki tüm container'lar başarıyla sonlandırılmıştır.

##### Failed
Pod'daki tüm container'lar sonlandırılmıştır ancak en az bir container başarısızlıkla sonlandırılmıştır.

##### Unknown
Bazı nedenlerden dolayı Pod'un durumu elde edilemedi.


Kubernetes, Pod'un genel aşamasının yanı sıra Pod içindeki her bir container'ın da durumunu izler. Scheduler, bir Pod'u bir Node'a atadıktan sonra, kubelet bir container runtime kullanarak bu Pod için container'lar oluşturmaya başlar.

Container'ların olası durumları;
- Waiting
- Running
- Terminated



### init containers
InitContainer, pod içerisindeki ana container ayağa kalkmadan önce çalışan özelleştirilmiş öncül containerlardır. InitContainer kullanımına örnek olarak uygulamanın ayağa kalkması için gereken config ve secretlerı Pod'un dosya sistemine yazmamıza yardımcı olacak bir uygulama ya da bazı kurulum scriptleri barındıran bir uygulama olarak düşünebiliriz.

Bir pod içerisinde birden fazla container olabildiği gibi birden fazla init container da olabilir. Bu init container'lar ardışık olarak sıralı olarak çalışırlar. Yani bir init container içerisindeki process çalışır, başarılı şekilde sonlandırılır ve diğer init container çalışmaya başlar.

Eğer bir init container başarılı olmazsa, bu container kubelet tarafından başarılı olana kadar yeniden başlatılır. Ancak pod'un restartPolicy'si 'Never' ise pod 'Failed' statüsüne geçer.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

### sidecar containers

Sidecar containerlar, bir pod içerisinde ana container ile birlikte çalışan ikincil container'lardır. Bu container'lar, ana uygulama container'ının işlevselliğini genişletmek veya loglama, monitoring, güvenlik, veri senkronizasyonu gibi ek işlevler sağlamak amacıyla kullanılırlar.

Tavsiye edilen yaklaşım olarak, bir Pod üzerinde yalnızca bir container çalıştırılır. Örneğin, yerel bir web sunucusu gerektiren bir web uygulamanız varsa, yerel web sunucusu sidecar'dır ve web uygulamasının kendisi ise ana container olarak düşünülür. 

Kubernetes açısından, sidecar container'lar, init container'ların pod başlatıldıktan sonra da çalışmaya devam eden özel bir versiyonu gibi düşünülebilir. initContainers alanındaki initcontainer tanımımıza `restartPolicy` ekleyebiliriz. Artık bu container'lar ana container'ı ya da diğer init container'ları etkilemeden çalıştırılabilir ya da durdurulabilir.