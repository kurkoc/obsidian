### Giriş

Prometheus, SoundCloud firması tarafından Go dili ile geliştirilmiş, açık kaynak bir metrics-based monitoring ve alerting sistemidir. Kubernetes’ten sonra ikinci CNCF üyesidir 9 Ağustos 2018’den tarihinden beri “graduated project” olarak değerlendirilmektedir.

Prometheus kendisine verilen metric endpointlerden, metrik verilerini scrap ederek, time-series olarak toplar ve saklar. metrik verisi, bir zaman damgası ve metrik verisini tanımlayan key-value çiftleri şeklinde isteğe bağlı labellardan oluşur. Metrik dediğimiz şey, herhangi bir sayısal ölçüm değeridir diyebiliriz. Time series demek ise, zamana karşı olan değeri ifade eder. Bu açıdan, metrik verilerini time-series olarak saklamak, bir değerin zaman içerisindeki değişimini takip edebilmemize olanak sağlamaktadır. 

Metrikler, uygulamaların neden belirli bir şekilde çalıştığını anlamada önemli bir rol oynar. Örnek olarak, yavaş çalıştığını düşündüğümüz bir uygulamanın neden yavaş çalıştığını analiz etmede anlık istek sayısı, anlık kullanıcı sayısı gibi metriclerden faydalanabiliriz.


> Prometheus metric isimleri `[a-zA-Z_:][a-zA-Z0-9_:]*` regex formatına uygun olmalı ve ölçülmek istenen değer hakkında fikir vermelidir. Örnek olarak; `http_requests_total`

> Prometheus label'ları bir metric değerinin olası cardinality'sini, olası değerlerini ifade edecek belirteçlerdir. `[a-zA-Z_][a-zA-Z0-9_]*` regex formatına uygun isimlendirilmelidir.

Örnek olarak, metric adı api_http_requests_total ve labelları method=POST ve handler="/messages" olan bir time series aşağıdaki gibidir

```
api_http_requests_total{method="POST", handler="/messages"}
```

#### Metric Tipleri

##### Counter

Counter, sürekli olarak artan tek bir sayacı temsil eden metrikler için kullanılacak tiptir. Örneğin, sunulan isteklerin, tamamlanan görevlerin veya hataların sayısını temsil etmek için counter kullanabilirsiniz.
##### Gauge

Gauge, artıp azalabilen tek bir sayısal değeri temsil eden metrikler için kullanılacak tiptir. Örneğin, anlık çalışan işlemlerin sayısı, mevcut bellek kullanımı, eşzamanlı istek sayısı gibi yukarı ve aşağı gidebilen değerler için kullanılır.
##### Histogram

Histogram metric tipi, dağılımı ölçmek için kullanılır. Özellikle latency, size veya başka ölçülebilir değerlerin frekans dağılımını anlamak için idealdir. Histogram, ölçülen değerlerin belirli aralıklara (buckets) göre kaç kere gerçekleştiğini sayar. Bu sayede, değerlerin hangi aralıklarda yoğunlaştığını görürüz ve ortalama veya p95 gibi percentile hesaplarını yapabiliriz.

Bir histogram metriği 3 tip veri grubundan

oluşur:
- bucketlar `<basename>_bucket{le="<upper inclusive bound>"}`
- `<basename>_sum` Tüm ölçümlerin toplamını gösterir
- `<basename>_count` Toplam kaç ölçüm yapıldığını gösterir.

```
http_request_duration_seconds_bucket{le="0.1"} 240
http_request_duration_seconds_bucket{le="0.5"} 500
http_request_duration_seconds_bucket{le="1"}   800
http_request_duration_seconds_bucket{le="+Inf"} 1000

http_request_duration_seconds_count 1000
http_request_duration_seconds_sum   560.34
```

##### Summary

Histogram gibi ölçümlerin dağılımını izlemek için kullanılır, ama farklı bir yaklaşımla çalışır. count ve sum bulurken, kayan bir zaman penceresi üzerinde yapılandırılabilir nicelikleri hesaplar. 

```
http_request_duration_seconds{quantile="0.5"} 0.32
http_request_duration_seconds{quantile="0.9"} 0.55
http_request_duration_seconds{quantile="0.99"} 1.2

http_request_duration_seconds_sum  1347.1
http_request_duration_seconds_count 3560

```

### Prometheus Architecture

![prometheus_architecture](../_images/prometheus_architecture.png)

Prometheus, kaynakları izlemek için exporter olarak adlandırılan araçları(windows exporter, node-exporter, redis-exporter vb.) kullanır. Exporterlar, izlenecek olan kaynağa ait verileri Prometheus sisteminin anlayacağı biçime çevirerek HTTP protokolü(/metrics) üzerinden sunar. Prometheus, configuration dosyası üzerinde belirtilen exporter adreslerine belirlenen aralıklarla HTTP isteği yaparak(polling) verileri TSDB'ye kaydeder. 

Kaydedilen veriler, PromQL kullanılarak Prometheus Web GUI, Grafana gibi araçlar üzerinden servis edilir.

Rule tanımları yapılırsa, konfigüre edilen bir Alertmanager aracılığıyla, anomali içeren metric değerleri üzerinden Alertmanager kullanarak belirlenen notification arayüzleri aracılğıyla alert'ler üretilebilir.

Prometheus server, 3 kısımdan oluşur.

1- Retrieval; target’lardan pull edilen verileri alır ve prometheus üzerindeki local time series db’ye yazar
2- TSDB; time series database
3- HTTP server; TSDB üzerindeki verileri diğer araçlara sunar (örneğin grafana)


### Kurulum

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

```
docker volume create prometheus-data
docker network create monitoring
docker run -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml -v prometheus-data:/prometheus --network monitoring prom/prometheus
```

```
docker network create monitoring
docker run -d --name prometheus -p 9090:9090 -v C:\prometheus\configs:/etc/prometheus -v C:\prometheus\data:/prometheus --network monitoring prom/prometheus
```

Docker üzerinden çalıştırdığımızda, default olarak aşağıdaki flag'lerle çalışır.

```
/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/prometheus
```

Örnek konfigürasyon olarak, prometheus'a sadece kendi metriclerini toplaması için localhost:9090 adresini verdik. Prometheues, 15 saniyede bir localhost:9090/metrics url'i üzerinden kendi metriclerini toplayacak.

```
 curl localhost:9090/metrics
 
# HELP go_gc_cycles_automatic_gc_cycles_total Count of completed GC cycles generated by the Go runtime. Sourced from /gc/cycles/automatic:gc-cycles.
# TYPE go_gc_cycles_automatic_gc_cycles_total counter
go_gc_cycles_automatic_gc_cycles_total 20
# HELP go_gc_cycles_forced_gc_cycles_total Count of completed GC cycles forced by the application. Sourced from /gc/cycles/forced:gc-cycles.
# TYPE go_gc_cycles_forced_gc_cycles_total counter
go_gc_cycles_forced_gc_cycles_total 0
# HELP go_gc_cycles_total_gc_cycles_total Count of all completed GC cycles. Sourced from /gc/cycles/total:gc-cycles.
# TYPE go_gc_cycles_total_gc_cycles_total counter
go_gc_cycles_total_gc_cycles_total 20
# HELP go_gc_duration_seconds A summary of the wall-time pause (stop-the-world) duration in garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.4351e-05
go_gc_duration_seconds{quantile="0.25"} 9.3308e-05
go_gc_duration_seconds{quantile="0.5"} 0.000168452
go_gc_duration_seconds{quantile="0.75"} 0.000400366
go_gc_duration_seconds{quantile="1"} 0.000647535
go_gc_duration_seconds_sum 0.004785347
go_gc_duration_seconds_count 20
```

![prometheus_sample](../_images/prometheus_sample.png)

Healthcheck olarak 
```
http://localhost:9090/-/healthy
```

Readiness check olarak da
```
http://localhost:9090/-/ready
```

endpointleri kullanılabilir.

### Configuration

Prometheus, command line flag'leri ve bir configuration dosyası aracılığıyla yapılandırılabilir. Prometheus'un çalışması ile alakalı bazı ayarlar flag'ler üzerinden yapılırken(storage lokasyonu, disk, memory kısıtları vs.), scrap ile alakalı kullanacağımız hemen her konfigürasyon(job tanımları, job intervalleri vs.) ise configuration dosyası aracılığıyla yapılabilir.

Bazı command line flag'lerine bakacak olursak eğer;

`--config-file` : Prometheus konfigürasyon dosyası path'i. Default olarak prometheus.yml

`--storage.tsdb.path` : Metric'lerin saklanacağı path. Default olarak data/

`--storage.tsdb.retention.time` : Verilerin ne kadar süreyle storage üzerinde tutulacağı. size da belirtilmemişse default değeri 15d'dir.

`--storage.tsdb.retention.size` : 

`--web.listen-address` : Prometheus UI adresi. Default olarak 0.0.0.0:9090

`--web.enable-lifecycle` : Endpoint üzerinden reload ve shutdown yapabilmeyi aktifleştirir. Default olarak false

`--web.enable-admin-api` : Bazı yönetimsel endpointler için kullanılır. Default olarak false

`--web.page-title` : UI web sayfasının title'ı. Default olarak "Prometheus Time Series Collection and Processing Server"

`--log.level` : Default info. Olası değerler debug, info, warn, error

`--log.format` : Default değeri logfmt. Olası değerler logfmt, json
 
> [!Not] 
> Bütün flag'lere ve mevcut değerlerine http://localhost:9090/api/v1/status/flags endpoint'i üzerinden bakılabilir.


Bazı http api endpointleri; https://prometheus.io/docs/prometheus/latest/querying/api/

http://localhost:9090/api/v1/status/config

http://localhost:9090/api/v1/status/flags

http://localhost:9090/api/v1/status/runtimeinfo

http://localhost:9090/api/v1/status/buildinfo

http://localhost:9090/api/v1/status/tsdb

http://localhost:9090/api/v1/status/walreplay

http://localhost:9090/api/v1/targets

http://localhost:9090/api/v1/alertmanagers

http://localhost:9090/api/v1/alerts

http://localhost:9090/api/v1/rules

http://localhost:9090/api/v1/metadata

http://localhost:9090/api/v1/targets/metadata


Örnek bir konfigürasyon dosyası oluşturursak;

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'wmi'
    static_configs:
        - targets:
          - '172.20.50.143:9182'
          - '172.20.50.144:9182'
          - '172.20.50.145:9182'
          - '172.20.50.146:9182'
```

`scrape_interval` : Target'lardan ne sıklıkla scraping işlemi yapılacak. Default değeri 1m

`scrape_timeout` : Bir target’tan scraping yaparken kullanılacak timeout süresi. interval süresinden fazla olamaz doğal olarak. Default değeri 10 sn

`evaluation_interval` : Rule tanımlarımızın ne sıklıkla kontrol edileceği Defaul değeri 1m

`external_labels` : label_name:label_value şeklindeki label'lar

`query_log_file` : PromQL sorgularının kaydedileceği dosya

`scrape_failure_log_file` : Scraping hatalarının kaydedileceği dosya

`body_size_limit`: Scraping sonrasında gelen veri boyutu limiti. Size türünden belirtilir. Ör: 100MB . Default değeri 0'dır. Limit yoktur 

`sample_limit` : Scraping işlemi sırasında gelen sample limiti. Default değeri 0'dır. Limit yoktur

`label_limit` : Sample üzerinde belirtilen label adedi limiti. Default değeri 0'dır. Limit yoktur

`rule_files` : Burada belirtilen glob pattern'e uyan rule dosyalarından rule tanımları eklenir. 

`scrape_config_files`: Burada belirtilen glob pattern'e uyan config dosyalarından scraping tanımları eklenir.

`scrape_configs` : scrape_config tipinde scraping tanımları

#### scrape_config

`job_name`: Scraping job'ı için belirlenen isim

`scrape_interval`: Burada interval değeri verilirse global’deki değer ezilir, verilmezse default olarak global.scrape_interval değeri alınır

`scrape_timeout` : Burada timeout değeri verilirse global’deki değer ezilir, verilmezse de default olarak global.scrape_timeout değeri alınır

`metrics_path` : Metric'lerin HTTP üzerinden getirileceği path. Default değeri /metrics dir.

`schema` : Metric requestleri için hangi schema kullanılacak. Default değeri http'dir.

`http_config` : Scraping işlemi sırasında kullanılacak http client ayarları. authentication/authorization, http header, tls ayarları vs
- basic_auth : username ve password belirterek basic authentication bilgisi gönderilebilir.
- authorization : bütün request'lerde gönderilecek Authorization header'ı. type:Bearer, credentials: token bilgisi gönderilir.
- oauth2
- tls_config
- proxy_url
- http_headers

Prometheus, çalışmaya devam ederken -doğru formatta bir konfigürasyon tanımlanmışsa- konfigürasyonunu yeniden yükleyebilir. Bunu yapmanın iki yolu vardır; ilki prometheus process'ine SIGHUP göndermek veya /-/reload endpoint'ine istek atmak(`--web-enable-lifecycle` enabled edilmiş olması lazım)

```
kill -s SIGHUP <PID>
```

```
docker exec prometheus killall -HUP prometheus
```


Prometheus çalıştırılırken, `--web-enable-lifecycle` flag'i enabled olarak çalıştırılırsa; reload ve quit(graceful olarak sonlandırır prometheus'u) gibi management api endpointleri çalıştırılabilir.


### Querying

Prometheus, PromQL isminde bir sorgulama dili desteği sunar. 

### Storage

Prometheus, disk üzerinde local bir time-series veritabanı içerir, ancak isteğe bağlı olarak remote depolama sistemleriyle de entegre olabilir.

![](../_images/prometheus_data.png)

```
sudo ls /var/lib/docker/volumes/prometheus-data/_data -Al

drwxr-xr-x 3 nobody nogroup  4096 Apr 15 23:00 01JRXTWPQNK1QCXT74JAAE7R8E
drwxr-xr-x 3 nobody nogroup  4096 Apr 16 17:00 01JRZRP8TYQJXB6SPNKV7J38SY
drwxr-xr-x 3 nobody nogroup  4096 Apr 17 11:00 01JS1PFSBYMQBXYWSM9DH8QTSM
drwxr-xr-x 3 nobody nogroup  4096 Apr 18 05:00 01JS3M9BE0NY6Y86ZJNE5XTD87
drwxr-xr-x 3 nobody nogroup  4096 Apr 18 23:00 01JS5J2VR9MTTHFVSHHD17JG80
drwxr-xr-x 3 nobody nogroup  4096 Apr 19 17:00 01JS7FWDVZJHFBBH1S9YTZY0RG
drwxr-xr-x 3 nobody nogroup  4096 Apr 20 11:00 01JS9DNYBN4AYHB53R1N15CY5F
drwxr-xr-x 3 nobody nogroup  4096 Apr 21 05:00 01JSBBFGDMT22W44MXCFNQ2YC1
drwxr-xr-x 3 nobody nogroup  4096 Apr 21 23:00 01JSD990R086NK8TSVF6YHA4M0
drwxr-xr-x 3 nobody nogroup  4096 Apr 22 17:00 01JSF72JX2Q54Y1H3X0X4TJEZK
drwxr-xr-x 3 nobody nogroup  4096 Apr 23 11:00 01JSH4W3BBDXFP7XN0DGKDYNNH
drwxr-xr-x 3 nobody nogroup  4096 Apr 24 05:00 01JSK2NNCFFYBVY95GWNZXSRWA
drwxr-xr-x 3 nobody nogroup  4096 Apr 24 23:00 01JSN0F605P0JNX7XST573SEPW
drwxr-xr-x 3 nobody nogroup  4096 Apr 25 17:00 01JSPY8R5YZC7SWMZQT89M79NQ
drwxr-xr-x 3 nobody nogroup  4096 Apr 26 11:00 01JSRW289FJVC0FNN2M2D170G8
drwxr-xr-x 3 nobody nogroup  4096 Apr 27 05:00 01JSTSVTDFBH6Z0SYH58EKJ1PC
drwxr-xr-x 3 nobody nogroup  4096 Apr 27 23:00 01JSWQNARTH4SW6B9XDCBMQE0G
drwxr-xr-x 3 nobody nogroup  4096 Apr 28 17:00 01JSYNEX21DS0H20DVBG4KPBTA
drwxr-xr-x 3 nobody nogroup  4096 Apr 29 11:00 01JT0K8DBB40C01GGA7VCWJJPZ
drwxr-xr-x 3 nobody nogroup  4096 Apr 30 05:00 01JT2H1ZNS5M7YY422WYFS4VBK
drwxr-xr-x 3 nobody nogroup  4096 Apr 30 09:00 01JT2YSC279ADWXAGJAZ0E8TCV
drwxr-xr-x 3 nobody nogroup  4096 Apr 30 11:00 01JT35N3J5WXE70VBWAXJP4X8P
drwxr-xr-x 3 nobody nogroup  4096 Apr 30 11:00 01JT35N3XPNMRDV7H8CFW6FVHF
drwxr-xr-x 2 nobody nogroup  4096 Apr 30 11:01 chunks_head
-rw-r--r-- 1 nobody nogroup     0 Mar 25 08:10 lock
-rw-r--r-- 1 nobody nogroup 20001 Apr 30 12:42 queries.active
drwxr-xr-x 3 nobody nogroup  4096 Apr 30 11:00 wal

```

Prometheus topladığı verileri, iki saatlik bloklar halinde gruplar.

Her iki saatlik blok için bir klasör oluşturulur ve bu klasör içerisinde;

- chunks klasörü => bu zaman aralığı için toplanan bütün time-series data
- meta.json ismindeki metadata dosyası
- index dosyası (chunks klasöründeki verilerin metric isimleri ve labellarını indeksler)
- tombstones dosyası

yer alır.

![](../_images/prometheus_chunk.png)

```
drwxr-xr-x 2 nobody nogroup     4096 Apr 15 23:00 chunks
-rw-r--r-- 1 nobody nogroup 14951491 Apr 15 23:00 index
-rw-r--r-- 1 nobody nogroup      903 Apr 15 23:00 meta.json
-rw-r--r-- 1 nobody nogroup        9 Apr 15 23:00 tombstones
```

`chunks` klasöründe bulunan veriler, her biri 512 MB’a kadar ulaşabilen segmentler üzerinde gruplanarak saklanır.

Prometheus üzerinde, veriler API aracılığıyla silindiğinde, verileri chunk’tan hemen silmek yerine silme kayıtları `tombstones` dosyalarında saklanır.

Bir metrik verisi prometheus tarafından ilk toplandığı anda önce bellekte tutulur ve tam olarak kalıcı değildir. `wal` klasörü içerisinde 128 mb’lık segmentler halinde WAL dosyaları tutularak herhangi bir sorun sonucunda server’ın kapanması gibi durumlarda, ileri doğru sarılarak veriler güvence altına alınır. Bu dosyalar henüz sıkıştırılmamış ham veriler içerir; bu nedenle normal blok dosyalarından önemli ölçüde daha büyüktürler.

```
sudo ls /var/lib/docker/volumes/prometheus-data/_data/wal -Alh

-rw-r--r-- 1 nobody nogroup 107M Apr 30 07:00 00000840
-rw-r--r-- 1 nobody nogroup 108M Apr 30 09:00 00000841
-rw-r--r-- 1 nobody nogroup 109M Apr 30 11:00 00000842
-rw-r--r-- 1 nobody nogroup 100M Apr 30 12:50 00000843
drwxr-xr-x 2 nobody nogroup 4.0K Apr 30 09:00 checkpoint.00000839
```


### Rules

Prometheus, iki çeşit rule tipini destekler.

Bunlar; 
- recording rules
- alerting rules
'dir.

Prometheus üzerinde rule'lar eklemek için, rule tanımlarını içeren YAML formatında bir dosya oluşturup ana Prometheus konfigürasyonundaki `rule_files` alanında bildirimini yaparız. Bu işlemin ardından SIGHUP ile ya da reload ile Prometheus konfigürasyonunun yenilenmesini sağlarız.

>Rule tanımlarını bildirmeden önce, format olarak doğruluğunu kontrol etmek için promtool kullanabiliriz. Biz kurulumu docker üzerinden yaptığımız için container üzerinde /bin/promtool path'inde promtool binary'sini görebiliriz.
> `promtool check rules /path/to/example.rules.yml`

#### Recording Rules

Recording rules, hesaplama açısından pahalı ifadeleri veya sıkça belirli bir hesaplama sonrasında anlam ifade eden değerleri, önceden hesaplamanıza ve bunların sonuçlarını yeni bir time-series olarak kaydetmenize olanak tanır. Önceden hesaplanan sonucu sorgulamak, genellikle her ihtiyaç duyulan hesaplamayı her defasında çalıştırmaktan çok daha hızlı olacaktır.

Örnek olarak, varsayalım ki biz bütün hesaplamalarımızda, grafana dashboardlarımızda vs. anlık kullanılan memory oranı üzerinden işlem yapmak istiyoruz. Ancak elimizde `windows_memory_physical_free_bytes` ve `windows_memory_physical_total_bytes` değerleri olmasına rağmen, yüzde ile alakalı bir metric bulunmuyor. Gösterim yaptığımız ortamda, anlık olarak bu değerler üzerinden hesaplama yaparak yüzde değerini gösterebiliriz. Ancak birkaç farklı dashboard'da gösterecek olursak hepsi için ayrı ayrı bu hesapları tekrar tekrar yapmak durumunda kalacağız.

```
groups:
  - name: custom_rules
    [ interval: <duration> | default = global.evaluation_interval ]
    rules:
      - record: windows_memory_physical_free_percent
        expr: 100 - 100 * windows_memory_physical_free_bytes / windows_memory_physical_total_bytes
```

Bir recording rule tanımlayarak, yaptığımız hesaplama üzerinden yeni isimde bir metric oluşmasını sağlayabiliriz. Böylelikle ihtiyaç olan her yerde bu yeni metric'i kullanabiliriz. Bu bizi sorgulama sırasındaki hesaplama maliyetinden kurtardığı gibi(tabi bu sefer de veri yazarken bir maliyet olacaktır), değerin merkezi tek bir yerden oluşturulmasını da sağlamış olacaktır. 



