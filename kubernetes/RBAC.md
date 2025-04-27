RBAC(Role Based Access Control), Kubernetes nesnelerine erişimi kontrol etmek için kullanılan bir izin sistemidir. RBAC, kullanıcılara ve gruplara belirli Kubernetes nesnelerine erişimi kısıtlamak ve yetkileri sınırlandırmak için kullanılır.

RBAC kapsamında Kubernetes üzerinde 4 farklı obje kullanılmaktadır.
- Role
- RoleBinding
- ClusterRole
- ClusterRoleBinding


RBAC konusuna girmeden önce, Kubernetes üzerindeki resource'larımız nelerdir ve bu resource'lar üzerinde hangi requestler(verb) çalıştırılabilir, bunlara nasıl öğrenebileceğimize bakalım. Burada resource'ların hangi apigroup içerisinde yer aldığı ve hangi request'leri kullanabildikleri görülebilir.

```
kubectl api-resources -owide

NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND                               VERBS                                                        
bindings                                         v1                                true         Binding                            create
namespaces                          ns           v1                                false        Namespace                          create,delete,get,list,patch,update,watch
nodes                               no           v1                                false        Node                               create,delete,deletecollection,get,list,patch,update,watch
pods                                po           v1                                true         Pod                                create,delete,deletecollection,get,list,patch,update,watch 
roles                                            rbac.authorization.k8s.io/v1      true         Role                               create,delete,deletecollection,get,list,patch,update,watch
```


#### Role
Role, belirli Kubernetes nesneleri üzerinde belirli eylem izinlerinin tanımlanmasına olanak sağlayan Kubernetes nesnesidir. Role tanımları, başlangıçta minumum yetki gibi düşünüp, eklemeli olarak devam ederek oluşturulur(deny diyerek ilerleyemeyiz). 

Role tanımları yaparken bir namespace ile ilişkilendiririz ve ilgil role o namepace içerisinde geçerli olur.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Yukardaki örnekte,  default namespace'i içerisindeki `pod` resource'ları üzerinde sadece `get`, `watch`, ve `list` yapabilme yetkisi tanımlanmış bir role tanımlaması yapılmıştır.

#### RoleBinding
Clusterımızı üzerinde bulunan, çeşitli izin bilgileri içeren role tanımlarını, herhangi bir user, group ya da serviceaccount ile ilişkilendirerek kullanabiliriz. Mevcut haliyle sadece bir tanım dosyasıdır. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```




```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: whoami-account
  namespace: whoami
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: whoami
  name: whoami-role
rules:
- apiGroups: ["", "apps", "batch"]
  resources: ["pods", "deployments", "services", "configmaps", "jobs"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: whoami-account-binding
  namespace: whoami
subjects:
- kind: ServiceAccount
  name: whoami-account
  namespace: whoami
roleRef:
  kind: Role
  name: whoami-role
  apiGroup: rbac.authorization.k8s.io
```

Kubernetes 1.24 ve sonrasında ServiceAccount oluşturulduğunda otomatik olarak secret oluşturulması kaldırıldı. Artık manuel olarak oluşturmamız gerekiyor.

```
apiVersion: v1
kind: Secret
metadata:
  name: whoami-token
  namespace: whoami
  annotations:
    kubernetes.io/service-account.name: whoami-account
type: kubernetes.io/service-account-token
```

```
kubectl get secret whoami-token -n whoami -o jsonpath='{.data.token}' | base64 --decode

eyJhbGciOiJSUzI1NiIsImtpZCI6ImFUVUJhSDEyQ0dIUGtXc21HY21hbm1JalRBRy1PdUlGbThlQkJtU1J4TTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3aG9hbWkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoid2hvYW1pLXRva2VuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6Indob2FtaS1hY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYWRhZDFlNzctOGNlMC00MDRmLWI4M2ItMjZkNDVlZDE4YjNkIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Ondob2FtaTp3aG9hbWktYWNjb3VudCJ9.X0FnfAYedBSuGNbsYkuqeJtmITIKwBQvsRVOAJW7xPTC84kKhgG8bDsoXvXf6EnB0UaV_VjK0xznEaGI_LyMSCzp4AQPIQ2gRpjK3JG14BnBuBCMpljIwfWQ9qaIQmv_setCAshfWaVUKiw4pt--Wj_zZZm8ryifP9dzkFgOzULaOunT9hhTaMYO04fYPp4aXmfqiJYboi90Z0TJ8vfx8BKVAghHfAhusAsEYhf58c9JJYzxTulGZoM0g-PLcSZGEiV6Dloq3i-ZsjULEA34m2jj_A1YT1MOc9yYxyCHruYGxxpAcgmnGEkjwo5xyJGEx9a5A757M69SaTzL_raxgQ%
```

```
kubectl get secret whoami-token -n whoami -o jsonpath='{.data.ca\.crt}' | base64 --decode
-----BEGIN CERTIFICATE-----
MIIDBTCCAe2gAwIBAgIINeIeyT6ZV6gwDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yNTA0MjMxNDAwNTdaFw0zNTA0MjExNDA1NTdaMBUx
EzARBgNVBAMTCmt1YmVybmV0ZXMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCZA9CWrtkzp9sObRIgV6Izv4Z6YWANejWLbfT5yi8wc4ImPTs7wheEdhc4
IvbzV2m0juhkgVcNCrOFMq9/+fTt0pH/g9JwD8wir9hagotfLQaYQS0ZZczHDSGz
2YOi5Og6oAmm/JOkEc0uxcrpRE0HG7yp8Aznq8xmjaIkh6bqcZXed7/hEeP3tDeC
eqMnxm7fpxmbNVDETJ5m2I9WqxKPz4RDIoxYl39mhYjnd+HNGhBKMVK5X9dPdmN6
Agqvyso4dFkR3xr46AOMfEOqgkT/TJOdwVG+5B0yTMGmcSC/yGv/RM013sk7pKyU
VLDGIsLxBpyrgYS0bRN0+4ssqkcBAgMBAAGjWTBXMA4GA1UdDwEB/wQEAwICpDAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBQEFcCXmo5fz9COMC0ubtTIfoikfjAV
BgNVHREEDjAMggprdWJlcm5ldGVzMA0GCSqGSIb3DQEBCwUAA4IBAQBJtcm+l5m1
oCWNF8mWntEC1DjfmtJ0CfVWlQWXw/oP43mnuRIAQFimnRQYxCw97wfO0txTeoNR
Zy1uM240Ie7KwnDXIw/csP/thaGa4ti1ekqyC+EYuziOIupXsvXf1HHJKbbnsObV
xx6deypIi9+Gm4Lio0rxqq1FkQ/FPCiP/Mt521OyPK3Ml2xKkV7v/RQZw7ND2318
nMw3TolqX3o5xyT3yP3uwlDRFa9ug34IZODJq61DIKVFEAdGGbjxAy7BM6Kd2LAO
IYsnVQDadfzLSa17VG0l0dzbKWqGeZ+YRrX6W5xVU8EVwSqrqYDHmYY5odY517k/
atk5O+Ksc+ZV
-----END CERTIFICATE-----

```

```
TOKEN=$(kubectl get secret whoami-token -n whoami -o jsonpath='{.data.token}' | base64 --decode)
kubectl config set-credentials whoami-user --token="$TOKEN"
```

```
kubectl config set-cluster $CLUSTER_NAME \
  --server=$CLUSTER_SERVER \
  --certificate-authority=<(echo "$CA_CERT") \
  --embed-certs=true
```

```
kubectl config set-context whoami-context --cluster=kind-kurkoc-cluster --namespace=whoami --user=whoami-user
```

```
sudo kubectl config set-context whoami-context --cluster=kind-kurkoc-cluster --user=whoami-user
```

![rbac](../_images/rbac.png)

