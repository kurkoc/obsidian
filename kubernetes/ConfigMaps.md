ConfigMap, uygulamalarımızda kullandığımız gizli olmayan konfigurasyon değerlerini, key-value çiftleri halinde depolayarak, sonrasında merkezileştirilmiş konfigürasyonları çeşitli şekillerde pod'ların kullanımına sunabileceğimiz kubernetes objesidir.

ConfigMap sayesinde özellikle ortama göre değişen, konfigürasyon değerlerimizi container imajlarımızdan ayırarak daha portable deployment süreçleri elde edebiliriz.

Oluşturulan config map değerleri, podlar tarafından; 
- environment variable
- command line argument
- volume
olarak kullanılabilir.

> [!not]
> configmap değerleri, etcd üzerinde saklanır.


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_host: "192.168.0.1"
  debug_mode: "1"
  log_level: "verbose"
```

```
kubectl apply -f config.yaml 
configmap/app-config created
```

```
kubectl get configmaps 
NAME               DATA   AGE
app-config         3      77s
```

```
kubectl describe configmap app-config
Name:         app-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
database_host:
----
192.168.0.1

debug_mode:
----
1

log_level:
----
verbose


BinaryData
====

Events:  <none>
```

```
kubectl get configmap app-config -o jsonpath='{.data}' | jq
{
  "database_host": "192.168.0.1",
  "debug_mode": "1",
  "log_level": "verbose"
}
```

Eğer istersek, clear text olarak sakladığımız verileri base64 olarak encode edilmiş halde de saklayabiliriz . Bu tarz alanları, `data` alanındansa, `binaryData` alanında base64 olarak tanımlamamız gerekir.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  database_host: "192.168.0.1"
  debug_mode: "1"
  log_level: "verbose"
binaryData:
  elastic_host: "ZWxhc3RpYzo5MjAw"
```

```
kubectl describe configmap demo-config

Name:         demo-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
log_level:
----
verbose

database_host:
----
192.168.0.1

debug_mode:
----
1


BinaryData
====
elastic_host: 12 bytes

Events:  <none>
```

```
kubectl get configmap demo-config -o jsonpath='{.binaryData.elastic_host}' | base64 -d

elastic:9200
```


### ConfigMap'lerin Pod'lar Üzerinde Kullanımı

#### Environment Variable Olarak

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: demo-config
```

bu pod'u çalıştırıp log'larına bakarsak eğer;

```
kubectl logs pod/demo-pod
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=demo-pod
SHLVL=1
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
debug_mode=1
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
log_level=verbose
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
database_host=192.168.0.1
```

Görüldüğü üzere, `envFrom` ile beraber `configMapRef` kullandığımızda, configmap üzerinde yer alan bütün key-value çiftleri otomatik olarak pod'un environment variable'larının arasına eklendi. 

Eğer istersek, bütün alanların otomatik olarak eklenmesi yerine, elle map'leyerek sadece belirli alan ya da alanların eklenmesini sağlayabiliriz. Bunun için `configMapKeyRef` diyerek hangi key'i eklemek istiyorsak açık şekilde belirtmeliyiz.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      env:
        - name: logging_mode
          valueFrom:
            configMapKeyRef:
              name: demo-config
              key: log_level
```


> [!NOT]
> Eğer kullanmaya çalıştığınız key ConfigMap içinde mevcut değilse ilgili container hata verir ve başlamaz. Fakat Pod içindeki diğer containerlar çalışmaya devam eder. Key mevcut olmadığında bile container'ın hata vermeden başlamasını istiyorsanız **`configMapKeyRef`** altına `optional: true` eklemelisiniz.


> [!NOT]
> envFrom ile tek seferde bütün configmap key'lerin environment variables olarak aktardığımızda, key'lerin başına prefix ekleyebiliriz. Örnek olarak, prefix değeri olarak `prefix: DOTNET_`  belirlediğimizde bütün variable'lar başında DOTNET_ olacak şekilde aktarılır.


#### Command line Argument Olarak

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "echo $(database_host)"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: demo-config
```

#### Volume Olarak

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
  labels:
    app: config-reader
spec:
  restartPolicy: Never
  containers:
  - name: config-reader
    image: busybox
    command: ["/bin/sh", "-c"]
    args:
    - echo "Listing files in /etc/configs:";
      ls -Al /etc/configs;
    volumeMounts:
    - name: configmap-file-volume
      mountPath: /etc/configs
  volumes:
  - name: configmap-file-volume
    configMap:
      name: configmap-basic
```

```
kubectl logs configmap-volume-pod

Listing files in /etc/configs:
total 4
drwxr-xr-x    2 root     root          4096 May  2 17:09 ..2025_05_02_17_09_15.449611640
lrwxrwxrwx    1 root     root            31 May  2 17:09 ..data -> ..2025_05_02_17_09_15.449611640
lrwxrwxrwx    1 root     root            20 May  2 17:09 database_host -> ..data/database_host
lrwxrwxrwx    1 root     root            17 May  2 17:09 debug_mode -> ..data/debug_mode
lrwxrwxrwx    1 root     root            16 May  2 17:09 log_level -> ..data/log_level
```

