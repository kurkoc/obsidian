Deployment gibi, bir StatefulSet de pod'ların dağıtımını ve ölçeklendirmesini yönetir ancak deployment'tan farklı olarak, statefulset, pod'larının her biri için yapışkan bir kimlik tutar. Bir deployment ile oluşturulan podlar, rastgele isimler ile tahmin edilemez sırayla başlatılırlar. Statefulset ise, podlara kalıcı ve sabit isimler vererek onların bir kimlik kazanmasını sağlar.

StatefulSet ile oluşturulan podlar, 0 index'i ile başlayarak {statefulset_name}{index} şeklinde isimlendirilirler ve bu sırayla çalıştırırlar. 

![statefulset](https://kerteriz.net/content/images/2023/06/kubernetes-statefulset-nedir-1.png)


Bir ReplicaSet, kapanan bir pod yerine yine rastgele bir isimle pod oluştururken, Statefulset kapanan pod'un aynı ismiyle yeni bir pod oluşturur.

scale up ettiğimiz bir deployment’ı tekrardan scale down ettiğimizde hangi pod’ların kapatılacağını kestiremeyiz, scale up işlemi sırasında yeni eklenen pod kapanabileceği gibi, ilk sırada oluşturulan pod da kapanabilir.

Bu durum birden fazla replica ya da cluster mantığında çoklu ortamda çalışabilen database, queue gibi podların oluşma sırasının ve hatta kimliğinin önemli olduğu sistemlerde sorunlara sebep olmaktadır.