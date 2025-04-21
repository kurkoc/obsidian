Liveness probe, bir container'ın hala düzgün çalışıp çalışmadığını belirlemek için kullanılan bir kontrol mekanizmasıdır. Bazı durumlarda, oluşturduğumuz pod çalışıyor görünmesine rağmen trafiğe cevap veremeyecek duruma düşerek, etkin şekilde çalışmıyor olabilir. Bu tarz durumlarda liveness probe'ları kullanabiliriz. 

Liveness probe'ları genellikle uygulama kilitlenmelerini veya kendi kendine düzeltilemeyen yanıt vermeyen durumları tespit etmek için kullanılır.

Liveness probe beklediğimiz sonucu içermiyorsa, Kubernetes işlevselliği geri yüklemek için container'ı sonlandırır ve container için belirlenen `restartPolicy` uyarınca bir aksiyon alınır. Restart policy, `Always` ya da `OnFailure` olarak belirlenmişse container yeniden başlar.

### Liveness Probe Türleri

Farklı uygulama türleri için tasarlanmış liveness probe türleri mevcuttur. 
- HTTP Probe
- TCP Probe
- gRPC Probe
- exec Probe

#### HTTP

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

Bu tanımda, `periodSecond` değeri ile, kubelet'in liveness probe'u kaç saniye aralıklarla tekrardan çalıştıracağını belirtiyoruz. `initialDelaySeconds` ile de ilk probe çalıştırılmadan önce kaç saniye bekleyeğini belirtiyoruz.

Bu tanıma göre, container ayağa kalktıktan 3 sn sonra başlamak kaydıyla, kubelet tarafından 3 sn aralıklarla container üzerinde çalışan uygulamaya 8080 portu üzerinden /healthz endpointine http get isteği gönderilecek. Gelen response'un http status code'u 200(dahil)-400 arasındaysa liveness true kabul edilir.


#### TCP

```yaml
livenessProbe:  
  tcpSocket:  
    port: 80  
  initialDelaySeconds: 5  
  periodSeconds: 10
```


#### gRPC

```yaml
livenessProbe:  
  grpc:  
    port: 2379  
  initialDelaySeconds: 10
```

#### exec

```yaml
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

