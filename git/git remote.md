
Dosya sistemine bir repository'i clone'larsak ilgili remote repository'i default olarak origin adıyla eklenir.

```
git clone https://kurkoc.visualstudio.com/whoami/_git/whoami
```

```
git remote -v

origin	https://kursadkoc@kurkoc.visualstudio.com/whoami/_git/whoami (fetch)
origin	https://kursadkoc@kurkoc.visualstudio.com/whoami/_git/whoami (push)
```

Bir repository birden fazla remote repository ile eşlenmiş olabilir. Tabi origin ismi sadece bir kez olabilir. Bu sebeple açık bir şekilde farklı bir isim verilmeli.

```
git remote add pb https://github.com/paulboone/ticgit
```

şimdi tekrardan kontrol edersek;

```console
git remote -v

origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

remote bir repository ile daha detaylı bilgi almak istersek eğer;

```
git remote show origin

* remote origin
  Fetch URL: https://kursadkoc@kurkoc.visualstudio.com/whoami/_git/whoami
  Push  URL: https://kursadkoc@kurkoc.visualstudio.com/whoami/_git/whoami
  HEAD branch: main
  Remote branch:
    main tracked
  Local branch configured for 'git pull':
    main merges with remote main
  Local ref configured for 'git push':
    main pushes to main (up to date)
```


`remove` komutu ile remote bir repository eşlemesi silinebilir

```console
git remote remove pb
```

`rename` komutu ile de repositıry eşleşmesine verilen alias değiştirilebilir.

```
git remote rename pb paulb
```

