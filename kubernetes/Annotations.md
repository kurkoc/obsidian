Annotation'lar, Kubernetes objelerine bilgi amaçlı metaveriler eklemek için kullanılan key-value çiftleridir. Annotation'lar, Label'lar gibi objelere metaveri eklemek için kullanılır, ancak Label'ların aksine objelerin gruplandırılması veya seçilmesinde kullanılmazlar. Annotation'lar, objelere ilgili ek bilgileri eklemek için kullanılır ve genellikle açıklama, sürüm numarası, oluşturma tarihi gibi bilgileri içerir.


```
apiVersion: v1
kind: Pod
metadata:
  name: annotationpod
  annotations:
    owner: "jane doe"
    notification-email: "admin@example.com"
    release-date: "01.01.2021"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  containers:
  - name: annotationcontainer
    image: nginx
    ports:
    - containerPort: 80
```


Kubectl komutu yardımıyla da annotation'lar ile ilgili ekleme, çıkarma, güncelleme işlemleri yapılabilir.

```
kubectl annotate pods foo description='my frontend'

kubectl annotate --overwrite pods foo description='my frontend running nginx'

kubectl annotate pods --all description='my frontend running nginx'

kubectl annotate pods foo description-
```
