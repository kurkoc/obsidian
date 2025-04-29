Prometheus, SoundCloud firması tarafından Go dili ile geliştirilmiş, açık kaynak bir metrics-based monitoring ve alerting sistemidir. Kubernetes’ten sonra ikinci CNCF üyesidir 9 Ağustos 2018’den tarihinden beri “graduated project” olarak değerlendirilmektedir.

Prometheus kendisine verilen metric endpointlerden, metrik verilerini scrap ederek, time-series olarak toplar ve saklar. metrik verisi, bir zaman damgası ve metrik verisini tanımlayan key-value çiftleri şeklinde isteğe bağlı labellardan oluşur. Metrik dediğimiz şey, herhangi bir sayısal ölçüm değeridir diyebiliriz. Time series demek ise, zamana karşı olan değeri ifade eder. Bu açıdan, metrik verilerini time-series olarak saklamak, bir değerin zaman içerisindeki değişimini takip edebilmemize olanak sağlamaktadır. 

Metrikler, uygulamaların neden belirli bir şekilde çalıştığını anlamada önemli bir rol oynar. Örnek olarak, yavaş çalıştığını düşündüğümüz bir uygulamanın neden yavaş çalıştığını analiz etmede anlık istek sayısı, anlık kullanıcı sayısı gibi metriclerden faydalanabiliriz.


> Prometheus metric isimleri `[a-zA-Z_:][a-zA-Z0-9_:]*` regex formatına uygun olmalı ve ölçülmek istenen değer hakkında fikir vermelidir. Örnek olarak; `http_requests_total`

> Prometheus label'ları bir metric değerinin olası cardinality'sini, olası değerlerini ifade edecek belirteçlerdir. `[a-zA-Z_][a-zA-Z0-9_]*` regex formatına uygun isimlendirilmelidir.

Örnek olarak, metric adı api_http_requests_total ve labelları method=POST ve handler="/messages" olan bir time series aşağıdaki gibidir

```
api_http_requests_total{method="POST", handler="/messages"}
```

### Prometheus Architecture

![prometheus_architecture](../_images/prometheus_architecture.png)

Prometheus, kaynakları izlemek için exporter olarak adlandırılan araçları(windows exporter, node-exporter, redis-exporter vb.) kullanır. Exporterlar, izlenecek olan kaynağa ait verileri Prometheus sisteminin anlayacağı biçime çevirerek HTTP protokolü(/metrics) üzerinden sunar. Prometheus, configuration dosyası üzerinde belirtilen exporter adreslerine belirlenen aralıklarla HTTP isteği yaparak(polling) verileri TSDB'ye kayıt eder. Kaydedilen veriler, PromQL kullanılarak Prometheus Web GUI, Grafana gibi araçlar üzerinden servis edilir.

Prometheus server, 3 kısımdan oluşur.

1- Retrieval; target’lardan pull edilen verileri alır ve prometheus üzerindeki local time series db’ye yazar
2- TSDB; time series database
3- HTTP server; TSDB üzerindeki verileri diğer araçlara sunar (örneğin grafana)


### Kurulum

```
docker run -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml -v prometheus-data:/prometheus --network monitoring prom/prometheus
```

### Configuration

Prometheus, command line flag'leri ve bir configuration dosyası aracılığıyla yapılandırılabilir. Prometheus'un çalışması ile alakalı bazı ayarlar flag'ler üzerinden yapılırken(storage lokasyonu, disk, memory kısıtları vs.), scrap ile alakalı kullanacağımız hemen her konfigürasyon(job tanımları, job intervalleri vs.) ise configuration dosyası aracılığıyla yapılabilir.

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
`job_name`: 
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