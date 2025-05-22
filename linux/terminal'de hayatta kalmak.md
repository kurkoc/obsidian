
`hostname`

`uname`

`/etc/os-release`

`printenv`

`echo $`

`cd ~` veya direk olarak `cd` dersek kullanıcının home dizinine gideriz. peki home dizinini nasıl görebiliriz? `echo $HOME`

`cd ..` üst klasör

`cd -` bir önceki lokasyona gider. 

Bir satırda birden fazla komut çalıştırmak istersek komutların arasına ; koyarak ayırabiliriz. `echo hello; echo world`

Üstteki senaryoda, bir önceki komudun başarıyla çalıştığından emin olduktan sonra ikinciyi çalıştırmak istersek aralarına `&&` koyarak yazabiliriz. `sudo apt update && sudo apt upgrade`


`ctrl + R` ile shell üzerinde önceki çalıştırılan komutlar üzerinde arama yapılabilir. Arama sonuçlarında tekrardan `ctrl + R` yaparak gezilebilir.
![[Pasted image 20250522165049.png]]
 
`ctrl + P` bir önceki, `ctrl + R` ise bir sonraki komutu tekrardan getirir.


Özellikle uzun komutlar yazarken, satırın başına veya sonuna gitmeye ihtiyaç duyabiliriz. `ctrl + A` ile satır başına (home tuşu da aynı işi yapar), `ctrl + E` (end tuşu da aynı işi yapar) ile de satır sonuna gidebiliriz. `ctrl + arrow` yön tuşları aracılığıyla ileri geri olacak şekilde kelime kelime komutlar üzerinde gezebiliriz.(`alt + B` ve `alt + F` de kullanılır)

`ctrl + U` cursor'ın olduğu yerden satır başına kadar olan her şeyi siler.

`ctrl + K` cursor'ın olduğu yerden satır sonuna kadar olan her şeyi siler.

`ctrl + shift + C` copy `ctrl + shift + V` paste

`!!` ile bir önceki komutu tekrardan çalıştırabiliriz. örnek olarak systemctl status docker çalıştırdık ama sudo istedi bizden, `sudo !!` yaptığımızda otomatik olarak sudo systemctl status docker çalıştırır.

`!$` ile bir önceki komutun son argümanı tekrardan kullanılır. `!^` son komuttaki ilk argüman, `!*` son komuttaki tüm argümanlar

```
ls /etc/nginx
...

cd !$
cd /etc/nginx

```

