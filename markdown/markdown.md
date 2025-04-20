markdown ile html'i birlikte kullanabiliriz.

---

### Heading

HTML heading elementleri h1..6 omak üzere 6 tanedir. Ve başlık seviyesini, # işareti ile temsil ederek markdown üzerinde kullanabiliriz.

```
# - <h1> 
## - <h2> 
### - <h3>
#### - <h4>
##### - <h5>
###### - <h6>

```




---

### Typography

Bold yapmak istediğimiz metinden önce ve sonra iki adet yıldız ekleyerek `**bold yapılmak istenen metin**`  şeklinde bold yapabiliriz. **metni kalın yapabiliriz**

Italic yapmak istediğimiz metinden önce ve sonra tek yıldız ekleyerek `*italic yapılmak istenen metin*` şeklinde italic yapabiliriz. *bu şekilde italic yapabiliriz*

Hem bold hem de italic yapmak istersek eğer, üç yıldız ekleyerek `***hem bold hem de italic yapılmak istenen metin***` şeklinde yapabiliriz.   ***bu şekilde yapabiliriz***

---

### Blockquote
Blockquote yapmak istersek tek yapmamız gereken ">" ifadesi ile başlatmak ifadeyi

> ** bu şekilde blockquote yapabiliriz **
> 
> bu satıra da ekleyebiliriz

---


### Url ve Email

url kullanmak istersek eğer; 

- doğrudan url kullandığımızda editor ifadeyi url olarak tanır. https://duckduckgo.com
- ancak bazen url'ler uzun olduğunda ya da direk olarak text üzerinden bu [şekilde](https://duckduckgo.com) görünsün istiyorsak eğer `[url_text](url)` şeklinde kullanabiliriz.
- url ifadesinden sonra eğer istersek title verebiliriz. `[url_text](url "title ifadesi")`  [title bilgisi olan url](https://duckduckgo.com "ben bir title bilgisiyim")

email kullanmak istersek eğer; 

`<email>` şeklinde kullanabiliriz. örnek:  <kursadkoc@live.com>

----


### Code

markdown ile bir kod bloğu göstermek istersek; üç backtick ile şekildeki gibi ` ``` code` kullanabiliriz.

```json
{
 "firstName": "John",
 "lastName": "Smith",
 "age": 25
 }
 ```

ya da tab veya 4 space ile de kod gösterimi yapabiliriz.


    var a = "sample variable"

---

### Image

url üzerinden bir image göstermek için; normal url gösterimine ek olarak ifadenin başına ! ekleyebiliriz.`![image_alt](image_url)`

![tech image](https://images.unsplash.com/photo-1519389950473-47ba0277781c?q=80&w=2070&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D)

ya da dosya sistemindeki bir image dosyasını gösterebiliriz.

![belvedere sarayı bahçesinden bir kare](belvedere.jpg)


---


### Horizontal Line
`---` ifadesi ile horizontal bir line çizebiliriz. `***` ile de çizebiliriz. `___` ile de çizebiliriz.

***

___

---


### Emoji

`:smile:` şeklinde emoji gösterebiliriz.  

:smile: 

### Table

Tablo eklemek için, her sütunun başlığını oluşturmak üzere üç veya daha fazla kısa çizgi (---) kullanın ve her sütunu ayırmak için boru (|) kullanın. 

| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |



 > vault içerisinde yer alan başka bir note'a link vermek istersek eğer `[[ internal note ]]` şeklinde ya da `ctrl + shift + L` hotkey'i ile verebiliriz. (Bu kombinasyonu kendim verdim hotkeys kısmında)
> Örnek olarak, [[git remote]] 

> embed ile bir note'u ya da resource'u aktif note içerisine gömebilirim. `![[ embed ]]` şeklinde ya da `ctrl + shift + B` hotkey'i ile verebiliriz.


![[Turkish_Sentiment_Analysis_Using_BERT.pdf]]