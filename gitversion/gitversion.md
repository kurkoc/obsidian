###### Fallback
Her zaman 0.0.0 döndürür ve etkilenen dalın artırma stratejisine bağlı olan bir sonraki sürümü hesaplamak için kullanılır (örneğin main'de bir sonraki sürüm 0.0.1 veya develop'ta 0.1.0'dır). Fallback yalnızca seçilen başka hiçbir strateji bir base version döndürmüyorsa kullanılır.

###### ConfiguredNextVersion
GitVersion.yaml dosyası üzerindeki `next-version` key'ine göre belirlenen değeri döndürür.

###### MergeMessage
Merge mesajı içerisinde bulunan versiyon bilgisini getirir. Örneğin, `Merge 'release/3.0.0' into 'main'` gibi bir merge mesajı bize 3.0.0 versiyonunu getirecektir.

###### TaggedCommit
Branch üzerinde geçerli olan ve geçerli commit'ten daha yeni olmayan, tag üzerinden versiyon bilgilerini çıkarır. 

###### TrackReleaseBranches
`track-release-branches: true` olarak belirlenmiş branch'ler için bir sonraki sürümü hesaplarken sürüm dallarından çıkarılan temel sürümü dikkate alır.

###### VersionInBranchName
Branch adından versiyon bilgisini getirir. Örneğin, `release/3.0.1` gibi bir branch adımız varsa 3.0.1 versiyonu hesaplanır.

###### Mainline
is-main-branch: true olarak belirlenmiş bütün branch'lerde, her commit için versiyon bilgisini artırır.
