Secret, parolalar, API key'leri gibi hassas bilgilerin depolanması ve yönetimi için kullanılan bir Kubernetes nesnesidir. Bu tür bilgileri, bir pod üzerinde ya da container tanımında kullanmak güvenlik açısından tercih edilen bir yöntem değildir. Secret aracılığıyla bu tarz hassas veriler, podlardan bağımsız bir şekilde olusturulduğundan, pod'ların oluşturulması, görüntülenmesi ve düzenlenmesi iş akışı sırasında açığa çıkma riski daha azdır.

> [!not]
> Secret'lar etcd üzerinde base64 encode edilerek saklanır. Bu yüzden etcd erişimi olanlar secret'ları görebilir ya da düzenleyebilirler. Bu açıdan, ileri güvenlik önmeli olarak etcd verisi encrypt edilebilir ve etcd erişimi sınırlandırılabilir. 


Uygulama pod'larına Secret'ları enjekte ederek, bu hassas bilgilere güvenli bir şekilde erişebilirsiniz. Secret objesi, YAML formatında tanımlanabilir ve içerisinde anahtar-değer çiftleri bulunur.

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
data:
 password: my-awesome-password
```

Saklamak istediğimiz parola bilgisini kullanarak secret oluşturmak istiyoruz.

```bash
kubectl apply -f secret.yaml

Error from server (BadRequest): error when creating "secret.yaml": Secret in version "v1" cannot be handled as a Secret: illegal base64 data at input byte 2
```

Görüldüğü üzere password key'i ile oluşturmak istediğimiz data base64 ile encode edilmiş bir veri olmadığından ötürü hata aldık. 

```bash
echo -n "my-awesome-password" | base64

bXktYXdlc29tZS1wYXNzd29yZA==
```

şimdi dosyamızı güncelleyelim.

```yaml
apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
data:
 password: bXktYXdlc29tZS1wYXNzd29yZA==
```


```
kubectl apply -f secret.yaml  

secret/my-secret created
```

Ya da manifest dosyamızda `data` field'ını `stringData` yaparak clear text bir tanımlama yapabiliriz.

``` yaml
apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
stringData:
 password: my-awesome-password
```

Clear text oluşturduğumuzda dahi kubernetes üzerinden erişmeye çalıştığımızda base64 olarak saklandığını görebiliriz.

```
kubectl get secret my-secret -o jsonpath='{.data}

{"password":"bXktYXdlc29tZS1wYXNzd29yZA=="}

kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode

	my-awesome-password

```


İmperatif bir şekilde `kubectl create secret` ile oluşturduğumuz tanımlarda base64 encode etmek gibi bir kısıtlama yoktur.

```
kubectl create secret generic db-user-pass --from-literal=username=admin --from-literal=password='S!B\*d$zDsb='

secret/db-user-pass created
```

`--from-literal` ile direk değeri verebildiğimiz gibi `--from-file` diyerek değerin dosyadan okunmasını da sağlayabiliriz.

```
kubectl create secret generic secret-from-file --from-file=username=username.txt --from-file=password=password.txt

secret/secret-from-file created
```


```
kubectl get secret secret-from-file -o jsonpath='{.data.username}' | base64 -d
file-username
```

```
kubectl get secret secret-from-file -o jsonpath='{.data.password}' | base64 -d
file-password
```

Yukardaki örnekte file'lar için ayrı ayrı dosyalar oluşturduk ve bunları username ve password isimli key'lerle eşleştirdik. Eğer istersek, bir klasör içerisinde yer alan bütün dosyaların otomatik olarak secret değişkeni gibi tanımlanmasını sağlayabiliriz. Bu kullanımda, dosya adı key olarak kullanılırken, dosya içeriği ise value değeri olur. 

secrets klasörü içerisine username ve password isminde iki dosya oluşturduğumuzda,

```
kubectl create secret generic secret-all-files --from-file=secrets
secret/secret-all-files created
```

```
kubectl get secret secret-all-files -o jsonpath='{.data}'
{"password":"ZmlsZS1wYXNzd29yZAo=","username":"ZmlsZS11c2VybmFtZQo="}

```



Clusterımızda bulunan secretleri listelemek istersek eğer; 

```
kubectl get secrets       

NAME           TYPE     DATA   AGE
db-user-pass   Opaque   2      3m55s
my-secret      Opaque   1      13m
```

```
kubectl describe secret db-user-pass

Name:         db-user-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes
username:  5 bytes
```

Oluşturduğumuz secret'taki değeri görmek için

```
kubectl get secret db-user-pass -o jsonpath='{.data.username}' | base64 --decode

admin
```


### Secret Tipleri

#### Opaque Secrets
Default secret tipidir.

#### ServiceAccount Token Secrets
Bir ServiceAccount için oluşturduğumuz token'ı saklamak istersek kullanabiliriz.

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: "sa-name"
type: kubernetes.io/service-account-token
data:
  extra: YmFyCg==
```

Bu secret türünü kullanırken, annotation olarak  `kubernetes.io/service-account.name` key'i ile mevcut bir ServiceAccount adını vermemiz gerekir.

#### Docker Config Secrets
Docker Registry'lerine kimlik doğrulama yapmak için kullanılan bir Secret tipidir. Bu tip, Docker imajlarını private registry'den çekmek için gereken kimlik bilgilerini içerebilir.

#### Basic Authentication Secret
Type olarak `kubernetes.io/basic-auth` kullanarak, basic authentication için gerekli olan bilgileri depolamak amaçlı secret oluşturabiliriz. Bu yüzden bu secret tipini kullanıyorsak, secret key'lerimiz `username` ve `password` olmalıdır. Tabi yine base64 olarak vermeliyiz.

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
data:
  username: YWRtaW4=
  password: dDBwLVNlY3JldA==
```

Clear text olarak kullanmak istersek eğer, `stringData` field'ı içerisinde tanımlama yapabiliriz.

``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin
  password: t0p-Secret
```

#### SSH Authentication Secrets
SSH kimlik doğrulama bilgilerini saklamak için kullanılan bir Secret tipidir. Bu tip, SSH sunucusuna erişim sağlamak için kullanılan kullanıcı adı ve private key bilgilerini içerebilir. Type olarak, `kubernetes.io/ssh-auth` vererek kullanabiliriz. 

Bu secret tipini kullanırken, data(veya stringData) alanımızdaki key'lerde `ssh-privatekey` değerini belirtmeniz gerekecektir.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-ssh-auth
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: UG91cmluZzYlRW1vdGljb24lU2N1YmE=...
```


#### TLS Secrets
TLS sertifikalarını ve private key'i saklamak için kullanılan bir Secret tipidir. Bu tip, HTTPS trafiğini şifrelemek veya uygulamalar arasında güvenli iletişim sağlamak için kullanılan sertifikaları içerebilir. Type olarak, `kubernetes.io/tls` vererek kullanabiliriz. 

Bu secret tipini kullanırken, data(veya stringData) alanımızdaki key'lerde `tls.key` ve `tls.crt` değerlerini belirtmeniz gerekecektir.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-tls
type: kubernetes.io/tls
data:
  tls.crt: ...   
  tls.key: ...
```

#### Bootstrap Token Secrets
Node bootstrap process'inde kullanılacak olan token'ları saklamak için kullanılır.Type olarak, `bootstrap.kubernetes.io/token` vererek kullanabiliriz.Genellikle namespace olarak `kube-system` veririz ve isimlendirme olarak da `bootstrap-token-<token-id>` kullanırız.

Bu secret tipini kullanırken, data(veya stringData) alanımızdaki key'lerde `token-id` ve `token-secret` değerlerini belirtmeniz gerekecektir.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-5emitj
  namespace: kube-system
type: bootstrap.kubernetes.io/token
data:
  auth-extra-groups: c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=
  expiration: MjAyMC0wOS0xM1QwNDozOToxMFo=
  token-id: 5emitj
  token-secret: a3E0Z2lodnN6emduMXAwcg==
  usage-bootstrap-authentication: dHJ1ZQ==
  usage-bootstrap-signing: dHJ1ZQ==
```



### Secret'ların Podlar Üzerinde Kullanımı

Kubernetes üzerinde tanımladığımız Secret'ları, bir pod tanımlaması yaparken kullanabilmek için iki farklı yöntem vardır.

![](https://cdn.prod.website-files.com/635e4ccf77408db6bd802ae6/6704d695156a64ece7cf5a04_AD_4nXfX2cwvThqXrzoP34YAe6LKA_IG1vZDRyfMqt93iJYlOdBenirHSYWqT55SkMI-8ARfukhP6j8EAJDmsRdSK48qzFDIOCNvh2Xhp1m4dS8CS6U76GjmwGv4GvEAof3UcNmnd1jokyRO2Sdcojynp5tGL0zN.png)

#### Environment olarak kullanım

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: mycontainer
      image: myimage
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
```

#### Volume olarak kullanım

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: mycontainer
      image: myimage
      volumeMounts:
        - name: secret-volume
          mountPath: "/etc/secret-volume"
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret

```
