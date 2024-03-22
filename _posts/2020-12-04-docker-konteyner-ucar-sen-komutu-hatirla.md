---
title: "Docker: Konteyner uçar sen komutu hatırla!"
layout: post
---

Merhaba sevgili okur,

Docker’ı yönetmek için kullanabileceğiniz (docker cli) temel komut listesini örnekler üzerinden anlatmaya çalıştım.

{% highlight shell %}
docker ps
{% endhighlight %}

Çalışan konteyner’ları listeler.

![docker](/assets/images/1_dP3wo4XAADSzpZnX8hrT9g.gif)

{% highlight shell %}
docker ps -a
{% endhighlight %}

Çalışan çalışmayan tüm konteyner’ları listeler.

![docker](/assets/images/1__-y6Ye9Ly3b40nIU21x6-Q.gif)

{% highlight shell %}
docker images
{% endhighlight %}

İndirilmiş image’ları listeler.

![docker](/assets/images/1_yv43McO2oZGKCqaq4jlOzg.gif)

{% highlight shell %}
docker pull alpine
{% endhighlight %}

Docker repository (hub.docker.com) den adı verilen (alpine) image’ı çeker.

Not: “alpine” küçük bir Linux dağıtımıdır.

![docker](/assets/images/1_L_od0qVCoVVaOGIErzmp8w.gif)

{% highlight shell %}
docker run alpine
{% endhighlight %}

Adı verilen image’dan bir konteyner oluşturur ve image’a eklemiş başlangıç kodunu çalıştırır. Bu başlangıç kodu dockerfile kısmında ayrıntılı açıklayacağım. Bu kod sonladıktan sonra konteyner kapalı duruma geçer.

![docker](/assets/images/1_syoBVVyN66IxymmMXQRxYQ.gif)

{% highlight shell %}
docker run -d jpetazzo/clock
{% endhighlight %}

Adı verilen image’dan bir konteyner oluşturur ve arka planda çalışmaya devam eder.

![docker](/assets/images/1_fhTpTiUZtTb-ASZCQJ0hqQ.gif)

{% highlight shell %}
docker run --rm hello-world
{% endhighlight %}

Adı verilen image’dan bir konteyner oluşturur ve konteyner’ın başlangıç kodu çalıştıktan sonra konteyner otomatik olarak silinir.

![docker](/assets/images/1_wZ1BOQE-xFs-gDljLBsBmQ.gif)

{% highlight shell %}
docker run -it alpine
{% endhighlight %}

Adıverilen image’dan bir konteyner oluşturur ve konteyner’e terminal bağlantısı sağlar.

![docker](/assets/images/1_YLpceVYFrmyY1KsXh8fTFQ.gif)

{% highlight shell %}
docker stop 09f8b2a5d25b
{% endhighlight %}

Çalışan konteyner’a kısa bir süre (10sn) sonra kapanacağını belirten bir sinyal gönderir ve süre tamamlandığında konteyner’ı sonladırılır.

![docker](/assets/images/1_fsWRCfgYoKhqllkLOY8liQ.gif)

{% highlight shell %}
docker kill 09f8b2a5d25b
{% endhighlight %}

Id’si verilen çalışan konteyner’ı sonladırır.

![docker](/assets/images/1_Y6rJGl4B7kTtgWcjnS2wGw.gif)

{% highlight shell %}
docker rm 99ae02566af7
{% endhighlight %}

Id’si verilen konteyner’i siler.

![docker](/assets/images/1_ETPVvVF_0Ikil3QzsL1ffQ.gif)

Not: Id kullanılan komutlar da id’nin benzersiz 3–4 karakterini de kullanabilirsiniz. (Örnek: 99ae02566af7 -> 99a)

{% highlight shell %}
docker rmi 321d39ea3f0f
{% endhighlight %}

Id’si verilen image’ı siler.

![docker](/assets/images/1_V-kmhEiVlxE21yzm8Agqog.gif)

Not: Image’ı kullanan konteyner’lar bulunuyorsa silme işlemi gerçekleşmez.

{% highlight shell %}
docker exec -it 321d39ea3f0f sh
{% endhighlight %}

Çalışan konteyner’a erişmek ve içinde komut çalıştırabilmek için kullanılır.

![docker](/assets/images/1_KcSM2yYa4B2tJWOeQ2aBEw.gif)

{% highlight shell %}
docker run -p 8080:80 nginx
{% endhighlight %}

Adı verilen image’dan bir konteyner oluşturur ve 8080 portuna gelen istekleri konteyner içerisindeki 80 portuna yönlendirir.

![docker](/assets/images/1_uCbbuRl6NEAG4OmzD1hVtQ-1.gif)

{% highlight shell %}
docker logs 321d39ea3f0f
{% endhighlight %}

Id’si verilen konteyner’ın oluşturmuş olduğu log’ları görüntüler.

![docker](/assets/images/1_oCQpZSKzgOaQoHw_hfkqiA.gif)

{% highlight shell %}
docker logs -f 321d39ea3f0f
{% endhighlight %}

Id’si verilen konteyner’ın oluşturmuş olduğu canlı log’ları görüntüler.

![docker](/assets/images/1_K63ZWqrp0voG6OdWeLufjg.gif)

Development ortamınız da burada ki komutlar (docker cli) ile Docker’ı yönetebilirsiniz. Fakat gerçek dünya da işler tabi ki böyle yürümüyor :( Her bir yazımda gerçek dünyaya bir adım daha yaklaşacağız. Bir sonraki postumda görüşmek üzere.
