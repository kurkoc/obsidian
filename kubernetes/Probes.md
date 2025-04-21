Probe'lar bir container'ın, başarıyla başlayıp başlamadığını, hala çalışır durumda ve trafik almaya hazır olduğunu, farklı fazlarda ve genellikle periyodik olarak sınayan kontrol mekanizmalarıdır. Bu kontrol sonucunda üretilen `Success`, `Failure` ve `Unknown` değerlerine göre aksiyonlar alınır.

Kubernetes için geliştirilmiş üç farklı probe türü vardır.
- Liveness Probe
- Readiness Probe
- Startup Probe

### Liveness Probes

Liveness probe, bir container'ın hala düzgün çalışıp çalışmadığını belirlemek için kullanılan bir kontrol mekanizmasıdır. Bazı durumlarda, oluşturduğumuz pod çalışıyor görünmesine rağmen trafiğe cevap veremeyecek duruma düşerek, etkin şekilde çalışmıyor olabilir. Bu tarz durumlarda liveness probe'ları kullanabiliriz. 

Liveness probe'ları genellikle uygulama kilitlenmelerini veya kendi kendine düzeltilemeyen yanıt vermeyen durumları tespit etmek için kullanılır.

Liveness probe beklediğimiz sonucu içermiyorsa, Kubernetes işlevselliği geri yüklemek için container'ı sonlandırır ve container için belirlenen `restartPolicy` uyarınca bir aksiyon alınır. Restart policy, `Always` ya da `OnFailure` olarak belirlenmişse container yeniden başlar.

#### Liveness Probe Türleri

Farklı uygulama türleri için tasarlanmış liveness probe türleri mevcuttur. 
- HTTP Probe
- TCP Probe
- gRPC Probe
- exec Probe

##### HTTP

```
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

HTTP Probe'ları için ayarlanabilecek konfigürasyonlar,
- `host` : İstek atılacak host adı. Belirtilmezse pod'un default ip adresidir. httpHeaders içerisinde Host olarak da belirtilebilir.
- `scheme` : Host'a bağlanılacak protokol(HTTP/HTTPS). Default değeri HTTP'dir.
- `path` : İstek atılacak url. Default değeri "/"
- `httpHeaders`: Gönderilecek custom header'lar
- `port` : Erişilmek istenen container port'u


Belirtilen bağlantı noktasına, kubelet tarafından HTTP request'i gönderilir ve request için gelen response status code'u takip edilir. Request atılırken, 
- `User-Agent: kube-probe/{version}` 
- `Accept: */*  ` 
header'ları default gönderilir. İhtiyaç halinde override edilebilir.

Bu tanımda, `periodSecond` değeri ile, kubelet'in liveness probe'u kaç saniye aralıklarla tekrardan çalıştıracağını belirtiyoruz. `initialDelaySeconds` ile de ilk probe çalıştırılmadan önce kaç saniye bekleyeceğini belirtiyoruz.

Bu tanıma göre, container ayağa kalktıktan 3 sn sonra başlamak kaydıyla, kubelet tarafından 3 sn aralıklarla container üzerinde çalışan uygulamaya 8080 portu üzerinden /healthz endpointine http get isteği gönderilecek. Gelen response'un http status code'u 200(dahil)-400 arasındaysa liveness true kabul edilir.


##### TCP

```yaml
livenessProbe:  
  tcpSocket:  
    port: 80  
  initialDelaySeconds: 5  
  periodSeconds: 10
```


##### gRPC

```yaml
livenessProbe:  
  grpc:  
    port: 2379  
  initialDelaySeconds: 10
```

##### exec

```yaml
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Readiness Probes

Kubernetes üzerinde çalışan ve hazır olmadan önce veri yüklemesi, büyük yapılandırma dosyalarına erişmesi veya diğer hizmetlere erişmesi gerektiği için başlatılması biraz zaman alan bir uygulamanız olduğunu düşünün. Doğru yanıt vermeye tam olarak hazır olmadan önce bu hizmete istek sunmaya başlamak istemezsiniz, çünkü trafiği zamanından önce göndermek hatalara veya düşük performansa neden olabilir, işte bu noktada Hazırlık probları devreye girer.

### Startup Probes

Startup Probe, bir container'ın doğru şekilde başlayıp başlamadığını kontrol etmek için kullanılır. Periyodik olarak çalışan liveness, readiness probe'ların aksine startup probe sadece başlangıçta bir kez çalışır. Özellikle, başlaması uzun süren container'larda kullanılmak için tasarlanmıştır. Startup probe başarısız olursa, Kubernetes container'ı yeniden başlatmayı deneyecektir.

Startup probe, başarılı olana kadar liveness ve readiness probe'ları başlamaz, böylelikle uygulamanın talep edilen şekilde çalıştığından emin olmadan, gereksiz yere liveness, readiness probe sorguları atılmaz. 



### Genel Ayarlar

Probe türlerinde, türe özel parametreler dışında genel olarak kullanılan konfigürasyon parametreleri aşağıdaki gibidir.

`initialDelaySeconds` : container çalışmaya başladıktan sonra startup, liveness ve readiness probe'ları çalışmaya başlamadan önce beklenecek süre. startup probe tanımı yaptıysak eğer, liveness, readiness probe'ları çalışmaya başlamadan önce onun başarılı sonuçlanması lazım. `periodSeconds`, `initialDelaySeconds` değerinden büyükse ilk bekleme süresi olarak periodSecond dikkate alınır.

`periodSeconds` : Probe'un ne sıklıkta tekrar çalıştırılacağı. Default değeri 10'dur. Yani tekrarlı probe'lar 10 sn'de bir çalıştırılır.

`timeoutSeconds` : Probe timeout süresi. Default değeri 1'dir.

`successThreshold` : Probe'un başarılı sayılması için minimum ardışık başarılı istek sayısı. Default değeri 1'dir.

`failureThreshold` : Bir probe üstüste kaç kez başarısız sonuç üretirse başarısız olarak değerlendirilecek. Default değeri 3.

`terminationGracePeriodSeconds` : container'ın başarısız olduktan sonra, gracefully şekilde kapatılabilmesi için talep edilen bekleme süresi. Default değeri 30.