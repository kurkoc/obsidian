git üzerinde, çalışma zamanı davranışlarını görebilmek ve belirlemek için config komutu kullanılır. bu konfigürasyonun belirlenebilmesi için sıralı önem düzeyine sahip bir takım config dosyası kullanılır.

```
git config --list
```

```
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=C:/Program Files/Git/mingw64/etc/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=false
core.editor="C:\\Program Files\\Notepad++\\notepad++.exe" -multiInst -notabbar -nosession -noPlugin
pull.rebase=false
credential.helper=manager
credential.https://dev.azure.com.usehttppath=true
init.defaultbranch=master
credential.https://tfs.tarim.gov.tr.provider=generic
safe.directory=D:/Projects/TayPortal
safe.directory=C:/Users/kursad.koc/source/repos/BTGM.Integrations
safe.directory=C:/Users/kursad.koc/Desktop/dev/devhub-ui
safe.directory=C:/Users/kursad.koc/source/repos/JsonOperations
safe.directory=C:/Users/kursad.koc/source/repos/SerilogLoki
user.name=kursad.koc
user.email=kursad.koc@tarimorman.gov.tr
init.defaultbranch=main
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
```

ya da tekil olarak bir config key'inin değerini görmek istersek de;

```
git config init.defaultbranch

main
```

---

git config, 3 farklı scope icin farklı lokasyonlarda config dosyası arar.

* System
* Global
* Local

bunlar için bakilan lokasyonlar

1- git_install_root/etc/gitconfig dosyasi = sistemdeki tum kullanicilar icin saklanan degerlerdir. Ör: C:\Program Files\Git\etc\gitconfig ya da /usr/local/etc/gitconfig
2- ~/.gitconfig = o andaki kullanıcıya özel değerler içerir.
3- proje_repository_path/.git/config = o repository için kullanılan ayarları içerir.

Sırasıyla config değerlerine bakalım.

```
git config --list --system
```

```
diff.astextplain.textconv=astextplain
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
http.sslbackend=openssl
http.sslcainfo=C:/Program Files/Git/mingw64/etc/ssl/certs/ca-bundle.crt
core.autocrlf=true
core.fscache=true
core.symlinks=false
core.editor="C:\\Program Files\\Notepad++\\notepad++.exe" -multiInst -notabbar -nosession -noPlugin
pull.rebase=false
credential.helper=manager
credential.https://dev.azure.com.usehttppath=true
init.defaultbranch=master
```



```
git config --list --global
```

```
credential.https://tfs.tarim.gov.tr.provider=generic
safe.directory=D:/Projects/TayPortal
safe.directory=C:/Users/kursad.koc/source/repos/BTGM.Integrations
safe.directory=C:/Users/kursad.koc/Desktop/dev/devhub-ui
safe.directory=C:/Users/kursad.koc/source/repos/JsonOperations
safe.directory=C:/Users/kursad.koc/source/repos/SerilogLoki
user.name=kursad.koc
user.email=kursad.koc@tarimorman.gov.tr
init.defaultbranch=main
```



```
git config --list --local
```

```
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
```

Burada bir hususa dikkat çekmek istiyorum. `init.defaultbranch` değeri hem system scope'undaki dosyadan hem de global scope'undaki dosyadan gelmektedir. System scope'undaki dosyadan master değeri gelirken, global scope'undaki dosyadan main değeri gelmektedir. Peki bu key'in nihai değeri nedir?

```
git config --get --show-origin --show-scope init.defaultbranch

global	file:C:/Users/kursad.koc/.gitconfig	main
```

System scope'undaki master değerindense, global scope'ta yer alan main değerinin ana konfigürasyon değeri olarak belirlendiğini gördük.

Farklı seviyelerdeki config dosyalarında yer alan aynı key'ler söz konusu olduğunda; seviyenin kapsamı daraldıkça, önceliği artar. yani repository içerisindeki config dosyası en çok öneme sahip iken system config dosyası önemi en az olandır diyebiliriz. Sonuç olarak, git’in bir repository uzerinde calisirken kullanacagi config bu uc dosyadaki configlerin birlesiminden elde edilecek olan bir sonuç kümesidir. 

Aynı keyler için önem sırası yüksek olan dosyadaki değer diğer dosyalardaki değerleri ezecektir.

```
git config --get-all --show-scope --show-origin init.defaultbranch

system  file:C:/Program Files/Git/etc/gitconfig master
global  file:C:/Users/kursad.koc/.gitconfig     main

```

---

Üstteki örneklerde görüldüğü üzere, `--show-scope` diyerek config değerinin handi scope'tan getirildiğini, `--show-origin` diyerek de hangi fiziksel path'teki config dosyasından getirildiğini görebiliriz.

```
git config --show-origin --show-scope --list
```

```
system  file:C:/Program Files/Git/etc/gitconfig diff.astextplain.textconv=astextplain
system  file:C:/Program Files/Git/etc/gitconfig filter.lfs.clean=git-lfs clean -- %f
system  file:C:/Program Files/Git/etc/gitconfig filter.lfs.smudge=git-lfs smudge -- %f
system  file:C:/Program Files/Git/etc/gitconfig filter.lfs.process=git-lfs filter-process
system  file:C:/Program Files/Git/etc/gitconfig filter.lfs.required=true
system  file:C:/Program Files/Git/etc/gitconfig http.sslbackend=openssl
system  file:C:/Program Files/Git/etc/gitconfig http.sslcainfo=C:/Program Files/Git/mingw64/etc/ssl/certs/ca-bundle.crt
system  file:C:/Program Files/Git/etc/gitconfig core.autocrlf=true
system  file:C:/Program Files/Git/etc/gitconfig core.fscache=true
system  file:C:/Program Files/Git/etc/gitconfig core.symlinks=false
system  file:C:/Program Files/Git/etc/gitconfig core.editor="C:\\Program Files\\Notepad++\\notepad++.exe" -multiInst -notabbar -nosession -noPlugin
system  file:C:/Program Files/Git/etc/gitconfig pull.rebase=false
system  file:C:/Program Files/Git/etc/gitconfig credential.helper=manager
system  file:C:/Program Files/Git/etc/gitconfig credential.https://dev.azure.com.usehttppath=true
system  file:C:/Program Files/Git/etc/gitconfig init.defaultbranch=master
global  file:C:/Users/kursad.koc/.gitconfig     credential.https://tfs.tarim.gov.tr.provider=generic
global  file:C:/Users/kursad.koc/.gitconfig     safe.directory=D:/Projects/TayPortal
global  file:C:/Users/kursad.koc/.gitconfig     safe.directory=C:/Users/kursad.koc/source/repos/BTGM.Integrations
global  file:C:/Users/kursad.koc/.gitconfig     safe.directory=C:/Users/kursad.koc/Desktop/dev/devhub-ui
global  file:C:/Users/kursad.koc/.gitconfig     safe.directory=C:/Users/kursad.koc/source/repos/JsonOperations
global  file:C:/Users/kursad.koc/.gitconfig     safe.directory=C:/Users/kursad.koc/source/repos/SerilogLoki
global  file:C:/Users/kursad.koc/.gitconfig     user.name=kursad.koc
global  file:C:/Users/kursad.koc/.gitconfig     user.email=kursad.koc@tarimorman.gov.tr
global  file:C:/Users/kursad.koc/.gitconfig     init.defaultbranch=main
local   file:.git/config        core.repositoryformatversion=0
local   file:.git/config        core.filemode=false
local   file:.git/config        core.bare=false
local   file:.git/config        core.logallrefupdates=true
local   file:.git/config        core.symlinks=false
local   file:.git/config        core.ignorecase=true
```


---

Konfigürasyon değerlerinde güncelleme yapmak istersek, bunu dosyalar üzerinden yapabiliriz.

```
git config --edit --global
git config --edit --system
git config --edit --local
```

`--edit` komutu ile belirttiğimiz scope'daki dosya, yine konfigürasyon olarak belirlediğim text editörü üzerinde düzenleme yapmamız amacıyla otomatik olarak açılır.

dosya üzerinden olduğu gibi, cli üzerinden de config değerlerini güncelleyebiliriz.

```
git config --<scope> <someproperty> <value>
```

örnek olarak varsayılan editörümüzü nano olarak belirlemek istersek eğer;

```
git config --global core.editor nano
```
