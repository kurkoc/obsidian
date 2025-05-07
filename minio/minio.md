### Kurulum 

```
docker run -d -p 9000:9000 -p 9001:9001 --name minio -v ~/minio/data:/data -e "MINIO_ROOT_USER=admin" -e "MINIO_ROOT_PASSWORD=MinioPa++w0rd" quay.io/minio/minio server /data --console-address ":9001"
```

```
docker container logs 18e0

INFO: Formatting 1st pool, 1 set(s), 1 drives per set.
INFO: WARNING: Host local has more than 0 drives of set. A host failure will result in data becoming unavailable.
MinIO Object Storage Server
Copyright: 2015-2025 MinIO, Inc.
License: GNU AGPLv3 - https://www.gnu.org/licenses/agpl-3.0.html
Version: RELEASE.2025-04-22T22-12-26Z (go1.24.2 linux/amd64)

API: http://172.17.0.3:9000  http://127.0.0.1:9000 
WebUI: http://172.17.0.3:9001 http://127.0.0.1:9001  

Docs: https://docs.min.io

```


![](../_images/minio_login.png)

Minio'yu docker üzerinden çalıştırdıktan sonra, `mc` ismindeki client uygulamasını indirelim.

```
export PATH=$PATH:/opt/homebrew/bin
brew install minio/stable/mc 
```

`mc` ile birden fazla minio instance'ına, birden fazla kullanıcının credential'ı ile bağlanabilirim. Bağlantıyı alias olarak ekleyip, cli komutlarını çağırırken o alias ile beraber çalıştırırırız.

ilk olarak client uygulamamızda kullanabilmek adına, UI üzerinden admin kullanıcısı için bir access key ve secret key oluşturalım. Sonrasında aşağıdaki komut ile bu bağlantıyı kaydedebiliriz.

```
mc alias set local http://localhost:9000 Ri77cyd3rNhb3y1OgRZP bgVhQK8dqrHqjMVEGhMwUMPm69jzErOkIgAq8V0p
```

Artık `local` ismiyle lokalimizdeki minio sunucusuna admin kullanıcısı olarak bağlanıp, cli komutlarını çalıştırabileceğiz.

![mc_alias_ls](../_images/mc_alias_ls.png)

Artık komutlarımızda `local` target'ını belirterek cli komutlarının bu bağlantı üzerinden çalışmasını sağlarız.

```
mc admin info local
```

![mc_admin_local](../_images/mc_admin_local.png)

`mc ls` komutuyla object storage üzerindeki dosya sisteminde listeleme işlemi yapabiliriz.

```
mc ls local

[2025-05-07 10:03:36 +03]     0B tayportal/
[2025-05-07 10:03:36 +03]     0B tikas/
```

Bu haliyle çalıştırdığımızda aslında sistemimizdeki bucket'lar listelenmiş oluyor.

Eğer istersek bir bucket'ın içerisine bakabiliriz.

```
mc ls local/tikas

[2025-05-07 10:55:22 +03]  65KiB STANDARD logs_screenshot.png
[2025-05-07 11:42:04 +03]     0B documents/
```

Burada bucket içerisindeki `documents` ismindeki klasörü sadece klasör olarak gösterdi içerisindekiler gösterilmiyor. Direk path olarak verip içerisine bakabiliriz.

```
mc ls local/tikas/documents

[2025-05-07 11:41:39 +03]  15KiB STANDARD UygulamaKullaniciSayilari.xlsx
[2025-05-07 11:38:32 +03] 191KiB STANDARD download.pdf
```

Ya da  (`--versions` dosyaların version bilgisini getirebilmek için eklendi, olmasa da olur)

```
mc ls local/tikas --recursive --versions

[2025-05-07 11:41:39 +03]  15KiB STANDARD 0cb09f36-6a38-413a-a62e-c8a839205368 v1 PUT documents/UygulamaKullaniciSayilari.xlsx
[2025-05-07 11:38:32 +03] 191KiB STANDARD 724b60a7-7237-4096-9bbd-2262945cc0a5 v1 PUT documents/download.pdf
[2025-05-07 10:55:22 +03]  65KiB STANDARD 653802e8-ac4a-4a2c-a03e-cc9fd67bd50a v1 PUT logs_screenshot.png
```

`mc stat` ile bir dosya ile ilgil bilgi almak için;

```
mc stat local/tikas/documents/download.pdf

Name      : download.pdf
Date      : 2025-05-07 11:38:32 +03 
Size      : 191 KiB 
ETag      : 80c03d266ab78ac7d1575f8bddae9f1b 
VersionID : 724b60a7-7237-4096-9bbd-2262945cc0a5 
Type      : file 
Metadata  :
  Content-Type: application/pdf
```

Yeni bir bucket oluşturmak için;

```
mc mb --with-lock local/devapp
```


`mc du` komutu ile, bucketların ve klasörlerin disk kullanımı görülebilir.

```
mc du local
271KiB	3 objects

mc du local/tikas
271KiB	3 objects	tikas

mc du local/tikas --recursive
206KiB	2 objects	tikas/documents
271KiB	3 objects	tikas

```

`mc find` ile bucket'lar üzerinde arama yapabiliriz.

```
mc find local

local/tayportal
local/tikas
local/tikas/documents/UygulamaKullaniciSayilari.xlsx
local/tikas/documents/download.pdf
local/tikas/logs_screenshot.png
```

```
mc find local --name "*.pdf"

local/tikas/documents/download.pdf
```

`mc get` ile bir dosya indirilebilir.

```
mc get local/tikas/logs_screenshot.png C:\Users\kursad.koc\Downloads

```

bu haliyle bucket üzerindeki ismiyle download eder. Eğer istenirse farklı olarak isimlendirebiliriz.

```
mc get local/tikas/logs_screenshot.png C:\Users\kursad.koc\Downloads\logs.png
```



### Kimlik Yönetimi

#### Users

Kullanıcıları listelemek istersek eğer;
```
mc admin user ls local

enabled    tayportal-user        tayportal-policy    
enabled    tikas-user            tikas-policy        
enabled    gkls-user 
```

Yeni bir kullanıcı eklemek için;
```
mc admin user add local tbs TbsPa++w0rd

Added user `tbs` successfully.
```

Bir kullanıcı hakkında bilgi almak istersek eğer;

```
mc admin user info local tikas-user

AccessKey: tikas-user
Status: enabled
PolicyName: tikas-policy
MemberOf: []
```

Kullanıcıyı disable/enable etmek istersek eğer;

```
mc admin user disable local tbs

Disabled user `tbs` successfully.
```

Kullanıcıyı silmek istersek;

```
mc admin user rm local tbs

Removed user `tbs` successfully.
```


#### Groups

```
mc admin group add local appusers tikas-user tayportal-user gkls-user

Added members `tikas-user,tayportal-user,gkls-user` to group `appusers` successfully.
```

Users örneğinde olduğu gibi grouplar listelenebilir.

```
mc admin group ls local

appusers
```

Bir grubuna detayına bakılabilir.

```
mc admin group info local appusers

Group: appusers
Status: enabled
Policy: 
Members: tikas-user,tayportal-user,gkls-user
```

Disable/enable edilebilir.

```
mc admin group disable local appusers

Disabled group `appusers` successfully.
```

Silinebilir.

```
mc admin group rm local appusers

Removed group appusers successfully.
```

Var olan bir gruba sonradan kullanıcı eklemek için;

```
mc admin group add local appusers tayportal-user

Added members `tayportal-user` to group `appusers` successfully.
```


#### Policies

Minio bazı built-in policy'lerle beraber gelir. Bunlar;

- consoleAdmin
- readonly
- readwrite
- diagnostics
- writeonly
dir.

UI uygulaması üzerinden bu policy'leri incelemek, policy tanımlama süreçlerini anlama açısından faydalı olacaktır. 

Örnek olarak ilk önce `readwrite` policy'si için bakalım.

![minio_policy_readwrite](../_images/minio_policy_readwrite.png)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```

Action olarak s3:* ile s3 üzerindeki bütün işlemlere yetki verilmiş. Resource olarak da arn:aws:s3:::* ile bütün resource'lar üzerinde yetkilendirilmiş. Yani bütün bucket'lar üzerinde okuma, ekleme ve silme yapabilir bir policy.

Şimdi benzer şekilde `readonly` için bakalım.

![minio_policy_readonly](../_images/minio_policy_readonly.png)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```

Burada dikkat edersek eğer, action kısmında sadece GetObject ve GetBucketLocation action'larına izin verilmiş. Yani bu policy, bütün bucket'lar üzerinde sadece okuma yapabilir.

Benzer şekilde `writeonly` policy'si için, action olarak sadece `s3:PutObject` verilmiştir. Bu policy okuma yapamaz, sadece object ekleyebilir. 

`diagnostic` policy'si action olarak admin şemasındaki aşağıdaki bütün işlemlerde izinlidir.
  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "admin:BandwidthMonitor",
                "admin:ConsoleLog",
                "admin:OBDInfo",
                "admin:Profiling",
                "admin:Prometheus",
                "admin:ServerInfo",
                "admin:ServerTrace",
                "admin:TopLocksInfo"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```

`consoleAdmin` şemasına bakarsak; s3, admin ve kms(key management service) şemalarındaki bütün işlemlere izin verildiğini görebiliriz.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "admin:*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ]
        }
    ]
}
```

Biz de buradaki tanımlardan yola çıkarak kendi policy tanımlarımızı yapabiliriz.

**admin policy seçenekleri** (https://min.io/docs/minio/linux/administration/identity-access-management/policy-based-access-control.html#mc-admin-policy-action-keys)

- Heal
- StorageInfo
- DataUsageInfo
- TopLocksInfo
- Profiling
- ServerTrace
- ConsoleLog
- KMSCreateKey
- KMSKeyStatus
- ServerInfo
- OBDInfo
- ServerUpdate
- ServiceRestart
- ServiceStop
- ConfigUpdate
- CreateUser
- DeleteUser
- ListUsers
- EnableUser
- DisableUser
- GetUser
- AddUserToGroup
- RemoveUserFromGroup
- GetGroup
- ListGroups
- EnableGroup
- DisableGroup
- CreatePolicy
- DeletePolicy
- GetPolicy
- AttachUserOrGroupPolicy
- ListUserPolicies
- CreateServiceAccount
- UpdateServiceAccount
- RemoveServiceAccount
- ListServiceAccounts
- SetBucketQuota
- GetBucketQuota
- SetBucketTarget
- GetBucketTarget
- SetTier
- ListTier
- BandwidthMonitor
- Prometheus
- ListBatchJobs
- DescribeBatchJobs
- StartBatchJob
- CancelBatchJob
- Rebalance


**s3 policy seçenekleri** (https://min.io/docs/minio/linux/administration/identity-access-management/policy-based-access-control.html#supported-s3-policy-actions)

- CreateBucket
- DeleteBucket
- ForceDeleteBucket
- GetBucketLocation
- ListAllMyBuckets
- DeleteObject
- GetObject
- ListBucket
- PutObject
- PutObjectTagging
- GetObjectTagging
- DeleteObjectTagging
- GetBucketPolicy
- PutBucketPolicy
- DeleteBucketPolicy
- GetBucketTagging
- PutBucketTagging
- AbortMultipartUpload
- ListMultipartUploadParts
- ListBucketMultipartUploads
- PutBucketVersioning
- GetBucketVersioning
- DeleteObjectVersion
- ListBucketVersions
- PutObjectVersionTagging
- GetObjectVersionTagging
- DeleteObjectVersionTagging
- GetObjectVersion
- BypassGovernanceRetention
- PutObjectRetention
- GetObjectRetention
- GetObjectLegalHold
- PutObjectLegalHold
- GetBucketObjectLockConfiguration
- PutBucketObjectLockConfiguration
- GetBucketNotification
- PutBucketNotification
- ListenNotification
- ListenBucketNotification
- PutLifecycleConfiguration
- GetLifecycleConfiguration
- PutEncryptionConfiguration
- GetEncryptionConfiguration
- GetReplicationConfiguration
- PutReplicationConfiguration
- ReplicateObject
- ReplicateDelete
- ReplicateTags
- GetObjectVersionForReplication

**kms policy seçenekleri** (https://min.io/docs/minio/linux/administration/identity-access-management/policy-based-access-control.html#kms-policy-action-keys)

- Status
- Metrics
- API
- Version
- CreateKey
- ListKeys
- KeyStatus

Bir proxy tanımlamak için, izin verilen veya reddedilen action'ların hangi resource'lar üzerinde uygulanacağını belirten bir policy tanımı oluşturmalıyız. 

```
{
   "Version" : "2012-10-17",
   "Statement" : [
      {
         "Effect" : "Allow",
         "Action" : [ "s3:<ActionName>", ... ],
         "Resource" : "arn:aws:s3:::*",
         "Condition" : { ... }
      },
      {
         "Effect" : "Deny",
         "Action" : [ "s3:<ActionName>", ... ],
         "Resource" : "arn:aws:s3:::*",
         "Condition" : { ... }
      }
   ]
}
```


Yeni bir policy eklemek için öncelikle policy tanım dosyamızı oluşturalım.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
		"Effect": "Allow",
		"Action": [
			"s3:GetObject", 
			"s3:PutObject", 
			"s3:DeleteObject",
			"s3:ListBucket"
		],
		"Resource": "arn:aws:s3:::tikas/*"
    }
  ]
}
```


```
mc admin policy create local tikas-policy C:/minio/tikas-policy.json

Created policy `tikas-policy` successfully.
```

![minio_tikas_policy](../_images/minio_tikas_policy.png)


Var olan bir policy'i silmek için;

```
mc admin policy rm local tikas-policy

Removed policy `tikas-policy` successfully.
```

Policy'leri listelemek için;

```
mc admin policy ls local

readwrite
tikas-policy
writeonly
consoleAdmin
diagnostics
readonly
```


Bir policy'yi bir user ile eşleştirmek için;

```
mc admin policy attach local tikas-policy --user tikas-user

Attached Policies: [tikas-policy]
To User: tikas-user
```

Artık sadece tikas bucket'i içerisinde listeleme, ekleme, silme ve indirebilme action'ları olan bir policy tanımladık ve bunu tikas-user isimli kullanımızla eşleştirdik. Artık tikas-user'ımızın sadece bu bucket'ı görebilmesini ve izin verilen işlemleri yapabilmesini bekliyoruz.

Bunu iki farklı şekilde test edebiliriz. 

Birincisi belirlediğimiz parola ile web arayüzü üzerinden giriş yaparak.

![minio_tikas_yetki](../_images/minio_tikas_yetki.png)

Dikkat edersek, buckets sayfasında sadece tikas bucket'ı geldi. Diğer bucketları görünür durumda değil. Yeni bir bucket tanımı yapabilmesini sağlayan Create Bucket butonu pasif durumda çünkü s3:CreateBucket action'ını tanımlamadık.

Bucket detay sayfasına girdiğimizde, normalde bucket'ın bilgilerini güncelleme, bucket'ı silme gibi özelliklerin hepsinin de pasif olduğunu görebiliriz.

![minio_tikas_yetki2](../_images/minio_tikas_yetki2.png)

İkinci yöntem olarak da, bu kullanıcı üzerinden bir bağlantı tanımı oluşturup `mc` üzerinden bu kullanıcı olarak sorgular çalıştırıp.

```
mc admin accesskey create local tikas-user

Access Key: RNNROHDT6MMDE0SCR7BZ
Secret Key: mhBfICROXxnKmwHGFdx2sq6O2kFBIPve9tEhsRUZ
Expiration: NONE
Name: 
Description: 
```

Bu access key ve secret key ile beraber ilgili bağlantıyı alias olarak tanımlayalım.

```
mc alias set tikas-connection http://localhost:9000 RNNROHDT6MMDE0SCR7BZ mhBfICROXxnKmwHGFdx2sq6O2kFBIPve9tEhsRUZ

Added `tikas-connection` successfully.
```

Artık komutlarımızı `tikas-connection` alias'ı ile beraber, bu kullanıcı gibi çalıştırabiliriz.

```
mc ls tikas-connection

[2025-05-07 10:03:36 +03]     0B tikas/
```

ls ile bucket'ları listeleyelim dediğimizde, sadece tikas bucket'ı geldi görüldüğü üzere.

Yeni bir bucket oluşturmak isteyelim.

```
mc mb --with-lock tikas-connection/tikasuserbucket

mc.exe: <ERROR> Unable to make bucket `tikas-connection/tikasuserbucket`. Access Denied.
```



### Monitoring

```
mc admin prometheus generate local

scrape_configs:
- job_name: minio-job
  bearer_token: eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJwcm9tZXRoZXVzIiwic3ViIjoiUmk3N2N5ZDNyTmhiM3kxT2dSWlAiLCJleHAiOjQ5MDAyMjA2MjB9.6MLBdC__pogGcHQOzJ9yGtKdr8KlHC1v6Ku_CMZDqZXsbZYY-DIVDZOcsdTir4-6ffQ40YDux-_I1RumKS8scw
  metrics_path: /minio/v2/metrics/cluster
  scheme: http
  static_configs:
  - targets: ['localhost:9000']
```

