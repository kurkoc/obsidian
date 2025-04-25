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
