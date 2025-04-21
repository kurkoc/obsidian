Probelar ile bir container'ın başarıyla çalışıp çalışmadığı, trafik almaya hazır olup olmadığı gibi kontroller üzerinden container'ı sınayan kontrol mekanizmalarıdır. Bu kontrol sonucunda üretilen `Success`, `Failure` ve `Unknown` değerlerine göre aksiyonlar alınır.

`initialDelaySeconds` : container çalışmaya başladıktan sonra startup, liveness ve readiness probe'ları çalışmaya başlamadan önce beklenecek süre. startup probe tanımı yaptıysak eğer, liveness, readiness probe'ları çalışmaya başlamadan önce onun başarılı sonuçlanması lazım. `periodSeconds`, `initialDelaySeconds` değerinden büyükse ilk bekleme süresi olarak periodSecond dikkate alınır.

`periodSeconds` : Probe'un ne sıklıkta tekrar çalıştırılacağı. Default değeri 10'dur. Yani tekrarlı probe'lar 10 sn'de bir çalıştırılır.

`timeoutSeconds` : Probe timeout süresi. Default değeri 1'dir.

`successThreshold` : Probe'un başarılı sayılması için minimum ardışık başarılı istek sayısı. Default değeri 1'dir.

`failureThreshold` : 