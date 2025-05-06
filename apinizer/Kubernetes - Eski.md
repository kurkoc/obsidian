
#### Nodes

```
kubectl get nodes -owide

NAME        STATUS   ROLES    AGE      VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kmaster01   Ready    master   3y228d   v1.18.4   172.20.50.73   <none>        Ubuntu 20.04.3 LTS   5.4.0-212-generic   docker://20.10.8
kworker01   Ready    <none>   3y226d   v1.18.4   172.20.50.75   <none>        Ubuntu 20.04.3 LTS   5.4.0-189-generic   docker://20.10.8
kworker02   Ready    <none>   3y228d   v1.18.4   172.20.50.76   <none>        Ubuntu 20.04 LTS     5.4.0-189-generic   docker://20.10.8
master03    Ready    master   616d     v1.18.4   172.20.50.85   <none>        Ubuntu 20.04.5 LTS   5.4.0-212-generic   docker://24.0.5
worker03    Ready    <none>   616d     v1.18.4   172.20.50.86   <none>        Ubuntu 20.04.5 LTS   5.4.0-189-generic   docker://24.0.5
```

```
kubectl get svc -A

NAMESPACE     NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
apinizer      manager               NodePort    10.102.119.120   <none>        8080:32080/TCP           47d
prod          cache-http-service    ClusterIP   10.101.151.91    <none>        8090/TCP                 2y202d
prod          cache-hz-service      ClusterIP   10.108.243.212   <none>        5701/TCP                 274d
prod          worker-http-service   NodePort    10.96.221.48     <none>        8091:30080/TCP           624d

```

NodePort olarak çalışan 2 servis var.

Manager uygulaması 32080 portundan

Worker servisine de 30080 portundan ulaşabiliriz.

#### Pods

```
kubectl get pods -n apinizer -l app=manager -owide

NAME                       READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
manager-6cd767d887-26m6c   1/1     Running   0          5d23h   10.244.3.22   kworker01   <none>           <none>
```


```
kubectl get pods -n prod -l app=worker -owide

NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
worker-f8d95c477-88gww   1/1     Running   0          5d23h   10.244.4.43   worker03    <none>           <none>
worker-f8d95c477-c5rnm   1/1     Running   0          5d23h   10.244.2.19   kworker02   <none>           <none>
```


```
kubectl get pods -n prod -l app=cache -owide

NAME                     READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
cache-5cfd8dbbb4-vb5g4   1/1     Running   0          5d23h   10.244.2.20   kworker02   <none>           <none>
```


#### Secrets

```
kubectl get secret mongo-db-credentials -n apinizer  -o jsonpath={'.data.dbName'} | base64 -d

apinizerdb
```

```
kubectl get secret mongo-db-credentials -n apinizer  -o jsonpath={'.data.dbUrl'} | base64 -d

mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.73:25080,172.20.50.74:25080,172.20.50.85:25080/?authSource=admin&replicaSet=apinizer-replicaset
```


```
kubectl get secret mongo-db-credentials -n prod  -o jsonpath={'.data.dbName'} | base64 -d
apinizerdb
```

```
kubectl get secret mongo-db-credentials -n prod  -o jsonpath={'.data.dbUrl'} | base64 -d

mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.105:25080,172.20.50.106:25080,172.20.50.107:25080/?authSource=admin&replicaSet=apinizer-replicaset
```

```
kubectl get secret -n apinizer default-token-hhvxf -o jsonpath={'.data'}

{"ca.crt":"LS0tLS1CRUdJTi..."}
```

**Worker pod'lardan birisine bağlanalım**

```
kubectl exec -it -n prod worker-f8d95c477-88gww -- bash
```

Sonrasında

```
cat /etc/hosts

# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
10.244.4.43     worker-f8d95c477-88gww

# Entries added by HostAliases.
100.64.208.10   services.turkpatent.gov.tr
100.64.64.178   kpsv.nvi.gov.tr
100.64.64.179   kpsv2.nvi.gov.tr        gisserver.nvi.gov.tr
100.64.64.184   kimlikdogrulama.nvi.gov.tr
100.64.64.185   kpsbasvuru.nvi.gov.tr
100.64.48.9     linkobs.ogm.gov.tr      api.ogm.gov.tr
100.64.114.26   bys.dsi.gov.tr
100.64.2.140    eraportest.saglik.gov.tr
100.64.2.138    erapor.saglik.gov.tr
100.64.35.37    apitest.tse.org.tr
100.64.34.250   mebkamunetservis.meb.gov.tr
100.64.15.9     ws.ticaret.gov.tr
100.64.1.28     kamu.sgk.gov.tr uygkamu.sgk.gov.tr      testkamu.sgk.gov.tr
100.64.10.13    webservices2.vedop2.ggm.gov.tr  webservices2.gib.gov.tr webservis2.gib.gov.tr
100.64.10.19    borcuyoktur.gib.gov.tr  borcuyokturyeni.gib.gov.tr
100.64.10.20    borcuyokturtest.gib.gov.tr
100.64.9.20     services.tkgm.gov.tr
100.64.9.22     cbsservis.tkgm.gov.tr
100.64.9.200    mimariproje.tkgm.gov.tr
100.64.10.11    webservis.gib.gov.tr
100.64.10.41    katpweb.gib.gov.tr
100.64.27.12    webapi.tuik.gov.tr
100.64.31.22    gis-prod-api.csb.gov.tr
100.64.125.57   esbenttest.tnb.org.tr
100.64.125.58   stsenttest.tnb.org.tr
100.64.125.90   esbent.tnb.org.tr
100.64.125.91   stsent.tnb.org.tr
100.64.3.18     test-ws-muhasebe.muhasebat.gov.tr
100.64.8.28     apigw.uab.gov.tr        tapdk.test.uab.gov.tr
172.16.77.89    yonetimekotaban.tarimorman.gov.tr
100.64.10.34    katptest.gib.gov.tr
100.64.10.33    katp.gib.gov.tr servis.islem.webservice.katp.gib.gov.tr
```


```
printenv

KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
WORKER_HTTP_SERVICE_PORT_8091_TCP=tcp://10.96.221.48:8091
CACHE_HTTP_SERVICE_PORT_8090_TCP_ADDR=10.101.151.91
SPRING_OUTPUT_ANSI_ENABLED=ALWAYS
HOSTNAME=worker-f8d95c477-88gww
LANGUAGE=en_US:en
JAVA_HOME=/opt/java/openjdk
CACHE_HZ_SERVICE_PORT=tcp://10.108.243.212:5701
CACHE_HTTP_SERVICE_PORT_8090_TCP=tcp://10.101.151.91:8090
JAVA_OPTS=-server -XX:MaxRAMPercentage=75 -Dhttp.maxConnections=4096
CACHE_HTTP_SERVICE_SERVICE_PORT=8090
CACHE_HZ_SERVICE_SERVICE_PORT=5701
WORKER_HTTP_SERVICE_SERVICE_HOST=10.96.221.48
PWD=/
tuneRoutingConnectionPoolMaxConnectionTotal=4096
CACHE_HTTP_SERVICE_PORT=tcp://10.101.151.91:8090
tuneBacklog=10000
WORKER_HTTP_SERVICE_PORT_8091_TCP_ADDR=10.96.221.48
SPRING_PROFILES_ACTIVE=prod
HOME=/root
LANG=en_US.UTF-8
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
CACHE_HZ_SERVICE_PORT_5701_TCP=tcp://10.108.243.212:5701
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
CACHE_HTTP_SERVICE_PORT_8090_TCP_PROTO=tcp
tuneWorkerThreads=1024
SPRING_DATA_MONGODB_DATABASE=apinizerdb
JHIPSTER_SLEEP=0
WORKER_HTTP_SERVICE_PORT_8091_TCP_PROTO=tcp
CACHE_HZ_SERVICE_SERVICE_HOST=10.108.243.212
WORKER_HTTP_SERVICE_SERVICE_PORT=8091
WORKER_HTTP_SERVICE_PORT_8091_TCP_PORT=8091
WORKER_HTTP_SERVICE_PORT=tcp://10.96.221.48:8091
TERM=xterm
tuneRoutingConnectionPoolMaxConnectionPerHost=1024
JAVA_TOOL_OPTIONS=-XX:+IgnoreUnrecognizedVMOptions -XX:+IdleTuningGcOnIdle -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,readonly,nonFatal
SHLVL=1
CACHE_HTTP_SERVICE_SERVICE_HOST=10.101.151.91
CACHE_HZ_SERVICE_PORT_5701_TCP_ADDR=10.108.243.212
CACHE_HTTP_SERVICE_PORT_8090_TCP_PORT=8090
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
CACHE_HZ_SERVICE_PORT_5701_TCP_PROTO=tcp
CACHE_HZ_SERVICE_PORT_5701_TCP_PORT=5701
KUBERNETES_SERVICE_HOST=10.96.0.1
LC_ALL=en_US.UTF-8
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
tuneBufferSize=16384
tuneWorkerMaxThreads=4096
SPRING_DATA_MONGODB_URI=mongodb://apinizer:yXjLF5qRJhNXmUB2@172.20.50.105:25080,172.20.50.106:25080,172.20.50.107:25080/?authSource=admin&replicaSet=apinizer-replicaset
JAVA_VERSION=jdk8u362-b09_openj9-0.36.0
tuneIoThreads=4
_=/usr/bin/printenv
```

**Elasticsearch**

```
{
  "name" : "172.20.50.77",
  "cluster_name" : "ApinizerEsCluster",
  "cluster_uuid" : "53Um70XhRuaDwlNsWmp6kA",
  "version" : {
    "number" : "7.9.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "d34da0ea4a966c4e49417f2da2f244e3e97b4e6e",
    "build_date" : "2020-09-23T00:45:33.626720Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
```
