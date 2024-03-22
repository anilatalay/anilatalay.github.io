---
title: "Regex - Konuşuyorum ama yazamıyorum"
layout: post
---

Merhaba sevgili okur,

Düzenli ifadeler, temel olarak metin içerisinde belirli kurallara göre eşleştirme yapmak için kullanılır. Burada bilinmesi gereken en önemli şey her şeyin bir karakter olduğudur. İhtiyacımız olduğunda bizim yerimize regex ifadeyi oluşturan hazır araçlar olmasına karşın, oluşturulan ifadenin ne anlama geldiğini anlamak adına kendimce notlar aldım, umarım sizde faydalanırsınız.

## Karakter

{% highlight js %}
// metin: merhaba dünya.
// regex: /y/
{% endhighlight %}

tercüme: 'y' karakteri olmalı.

sonuç: merhaba dünya.

index: 8

## Karakter dizisi

{% highlight js %}
// metin: merhaba dünya.
// metin: /düny/
{% endhighlight %}

tercüme: 'düny' karakter dizisi olmalı.

sonuç: merhaba dünya.

index: 8

## Sayılar

{% highlight js %}
// metin: 11111 22222 33333 44444
// metin: /44444/
{% endhighlight %}

tercüme: 5 adet '4' karakteri yan yana olmalı.

sonuç: 11111 22222 33333 44444

index: 18

## Herhangi bir karakter “.” (nokta)

” . ” (nokta) herhangi bir karakterle (harf, rakam, boşluk, her şey) eşleşebilir. Desene karakter sayısı kadar ” . ” (nokta) eklenebilir.

Not: Bu durum gerçek nokta karakterinin eşleşmesini geçersiz kıldığı için ters eğik çizgi ” \. ” ile gerçek nokta karakteri eşleşmesi sağlanır.

{% highlight js %}
// metin: merhaba dünya.
// regex: /r..b/
{% endhighlight %}

not: ".." yerine nokta sayısı kadar herhangi bir karakter gelebilir demektir.

tercüme: 'r' karakteri ile başlamalı, herhangi iki karakter ile devam etmeli ve son karakteri 'b' olmalı.

sonuç: merhaba dünya.

index: 2

## Herhangi bir rakam “\d”

Metin içerisinde ” \d ” herhangi bir rakamla eşleşebilir. Desene rakam sayısı kadar ” \d ” eklenebilir.

{% highlight js %}
// metin: abc123xyz
// regex: /\d\dx/
{% endhighlight %}

not: "\d\d" yerine \d sayısı kadar herhangi bir rakam gelebilir demektir.

tercüme: herhangi iki rakam yan yana gelmeli ve 'x' karakteri ile devam etmeli.

sonuç: abc123xyz

index: 4

## Belirli karakterler ” [ ] ” (veya)

Karakterleri köşeli parantez içinde tanımlayarak içlerinden biri ile eşleşmesini sağlayabilirsiniz.

{% highlight js %}
// metin: merhaba dünya.
// regex: /[xyz]a/
{% endhighlight %}

tercüme: 'x', 'y' veya 'z' karakterlerinden biri ile başlamalı ve 'a' ile devam etmeli.

sonuç: merhaba dünya.

index: 11

## Belirli karakterler hariç

Karakterleri köşeli parantez ve ^ (şapka) içinde tanımlayarak belirli karakterleri dışlayan bir desen eşleşmesini sağlayabilirsiniz.

{% highlight js %}
// metin: merhaba bünya cünya dünya.
// regex: /[^bc]ünya/
{% endhighlight %}

tercüme: 'b' ve 'c' dışında herhangi bir karakter ile başlamalı, 'ünya' ile devam etmeli.

sonuç: merhaba bünya cünya dünya.

index: 20

## Karakter aralıkları

Belirli karakterlerle eşleşen veya bazı karakterleri dışlayan bir desenin nasıl oluşturulduğunu anlatmaya çalıştım. Sıralı karakterleri teker teker tanımlamak yerine karakter aralığı verebiliyoruz. Örneğin, [0–6] deseni yalnızca sıfırdan altıya kadar olan herhangi bir tek haneli karakterle eşleşir. Aynı şekilde [^n-p] n’den p’ye kadar olan harfler hariç, yalnızca herhangi bir karakterle eşleşir.

{% highlight js %}
// metin: merhaba dünya.
// regex: /[A-Za-z]ün[^cd]a/
{% endhighlight %}

tercüme: Büyük 'A' ile 'Z' arasında bulunan karakterlerden herhangi biri veya küçük 'a' ile 'z' arasında bulunan karakterlerden herhangi biri ile başlamalı, 'ün' ile devam etmeli, 'c' ve 'd' karakterleri dışında herhangi bir karakter ile devam etmeli ve son karakteri 'a' olmalı.

sonuç: merhaba dünya.

index: 8

## Tekrar karakterler ” { karakter sayısı } “

Tekrar eden karakterleri teker teker tanımlamak yerine süslü parantezler içinde tekrarlı karakter sayısını belirtebiliriz.

{% highlight js %}
// metin: merhaba dddünya.
// regex: /d{3}ünya/
{% endhighlight %}

tercüme: Yan yana 3 adet 'd' karakteri ile başlamalı, 'ünya' karakter dizisi ile devam etmeli.

sonuç: merhaba dddünya.

index: 8

## Limitsiz tekrar karakterler ” \* “

Tekrar eden karakter sayısını bilmediğimiz durumlarda kullanılabilir. Dikkat edilmesi gereken nokta ise tekrar eden karakterin hiç olmadığı senaryolar içinde eşleşme sağlıyor oluşudur. Örneğin, “aacccc” metni için a*c* desenini yazabileceğimiz gibi, a*b*c\* desenide doğru bir eşleşme sağlayacaktır. ‘b’ karakteri yokken de, n tane varken de aynı şeyi ifade etmektedir.

{% highlight js %}
// metin: merhaba dddünnnnya.
// regex: /düny/
{% endhighlight %}

tercüme: 'd' karakteri 0 veya n tane yan yana başlamalı, 'ü' karakteri ile devam etmeli, 'n' karakteri 0 veya n tane yan yana devam etmeli ve 'y' karakteri ile bitmeli.

sonuç: merhaba dddünnnnya.

index: 8

## Limitsiz tekrar karakterler ” + “

Tekrar eden karakter sayısını bilmediğimiz durumlarda kullanılabilir. ” \* ” dan farkı ise tekrar eden karakterden en az bir tane olma zorunluluğudur.

{% highlight js %}
// metin: merhaba dddünnnnya.
// regex: /d+ün+y/
{% endhighlight %}

tercüme: 'd' karakteri 1 veya n tane yan yana başlamalı, 'ü' karakteri ile devam etmeli, 'n' karakteri 1 veya n tane yan yana devam etmeli ve 'y' karakteri ile bitmeli.

sonuç: merhaba dddünnnnya.

index: 8

## İsteğe bağlı karakter ” ? “

Bir karakterin ” ? ” ile kullanılması o karakterin opsiyonel olduğu anlamına gelir. Karakterin metin de bulunup bulunması göz ardı edilir.
Not: Bu durum gerçek soru işareti karakterinin eşleşmesini geçersiz kıldığı için ters eğik çizgi ” \? ” ile gerçek soru işareti karakteri eşleşmesi sağlanır.

{% highlight js %}
// metin: merhaba dünya.
// regex: /düns?y/
{% endhighlight %}

tercüme: 'dün' karakter dizisi ile başlamalı, 's' karakteri ile devam etmeli veya 's' karakteri hiç olmamalı, 'y' karakteri ile bitmeli.

sonuç: merhaba dünya.

index: 8

## Boşluk karakteri ” \s “

Boşluk karakterini eşleştirmek için kullanılır.

Not: Boşluk dışında herhangi bir karakter demek için ” \S ” kullanılır.

{% highlight js %}
// metin: merhaba dünya.
// regex: /\sdün/
{% endhighlight %}

tercüme: ' ' (boşluk) karakteri ile başlamalı, 'dün' karakter dizisi ile bitmeli.

sonuç: merhaba dünya.

index: 7

## Başlangıç ve bitiş

Metnin başlangıcında ve bitişinde eşleştirme yapabilmek için ” ^ ” ve ” $ ” kullanılır.

{% highlight js %}
// metin: metin: merhaba dünya.
// regex: /^merhaba\sdünya.$/
{% endhighlight %}

tercüme: metnin (içinde değil!) başlangıcı kesinlike 'm' karakteri ile başlamalı, 'erhaba' karakter dizisi ile devam etmeli, '\s' bir boşluk karakteri ile devam etmeli, 'dünya' karakter dizisi ile devam etmeli, metin (içinde değil!) sonu kesinlikle '.' karakteri ile bitmeli.

sonuç: merhaba dünya.

index: 0

## Gruplar

Desen içerisinde karakter veya karakter dizilerini gruplamak için parantez ” ( ) ” kullanılır. Eşleşme sonuçlarını ayrı ayrı görebiliriz.

{% highlight js %}
// metin: metin: merhaba dünya.
// regex: /(.+)\s(.+)/
{% endhighlight %}

tercüme: herhangi bir karakter ile başlamalı, herhangi bir karakter dizisi ile devam etmeli, boşluk karakteri ile devam etmeli, herhangi bir karakter dizisi ile bitmeli.

sonuç: merhaba dünya.

sonuç: merhaba

sonuç: dünya

index: 0

## İç içe gruplar

Desen içerisinde iç içe geçmiş karakter veya karakter dizilerini gruplamak için parantez ” ( ) ” kullanılır. Eşleşme sonuçlarını ayrı ayrı görebiliriz.

{% highlight js %}
// metin: merhaba dünya. 123
// regex: /(.\s.\s(\d\d\d))/
{% endhighlight %}

tercüme: herhangi bir karakter ile başlamalı, herhangi bir karakter dizisi ile devam etmeli, boşluk karakteri ile devam etmeli, herhangi bir karakter dizisi ile devam etmeli, boşluk karakteri ile devam etmeli, yan yana 3 adet rakam ile bitmeli.

sonuç: merhaba dünya. 123

sonuç: merhaba dünya. 123

sonuç: 123

index: 0

## Veya ” | “

{% highlight js %}
// metin: merhaba dünya. 123
// regex: /.\*(222\|123)/
{% endhighlight %}

tercüme: herhangi bir karakter ile başlamalı, herhangi bir karakter dizisi ile devam etmeli, '222' veya '123' ile bitmeli.

sonuç: merhaba dünya. 123

sonuç: 123

index: 0

## Son Söz

Okuduğunuz için teşekkür ederim, bir başka post’ta görüşmek dileğiyle.
