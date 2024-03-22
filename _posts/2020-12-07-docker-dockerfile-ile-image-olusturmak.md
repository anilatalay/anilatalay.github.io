---
title: "Docker: Dockerfile ile Image Oluşturmak"
layout: post
---

Merhaba sevgili okur,

Docker image’ları oluşturmak için Dockerfile yönergelerini nasıl kullanabileceğinizi anlatmaya çalıştım. Dockerfile için giriş veya gelişme yazısı olarak düşünebilirsiniz. Bir önce ki yazım da Docker’ı yönetmek için kullanabileceğiniz (docker cli) temel komut listesini paylaşmıştım. Keyifli okumalar.

Docker, Dockerfile’daki talimatları okuyarak image’ları otomatik olarak oluşturur.

Dockerfile başka bir dizinde ise -f argümanı ile dosya yolu verilebilir.

{% highlight shell %}
docker build -f files/Dockerfile .
{% endhighlight %}

Image etiketlemek için -t argümanı kullanılabilir.

{% highlight shell %}
docker build -t anilatalay/example:1.0.1 .
{% endhighlight %}

Etiket argümanı birden fazla kez kullanılabilir.

{% highlight shell %}
docker build -t anilatalay/example:1.0.1 -t anilatalay/example:latest .
{% endhighlight %}

Not: Docker daemon, Dockerfile’daki talimatları çalıştırmadan önce Dockerfile için bir ön doğrulama gerçekleştirir ve sözdizimi yanlışsa bir hata döndürür.

Not: Dockerfile’daki talimatlar sırayla ve tek tek çalıştırılır. (Deneysel bazı çalışmalar bulunuyor paralel çalıştırmak gibi..)

Docker, derleme sürecini önemli ölçüde hızlandırmak için değişiklik yok ise ara adımları (önbellek) yeniden kullanır.

\# ile başlayan satırları yorum olarak değerlendirilir.

\ kodun alt satırdan devam ettiği anlamına gelir.

{% highlight shell %}
RUN echo hello \
world
{% endhighlight %}

## FROM

DockerHub’da daha önce hazırlanmış olan temel(base) image’lar üzerine bir şeyler ekleyip-çıkartarak kendi nihai image’zımızı oluştuyoruz.
Not: Dockerfile bir FROM talimatı ile başlamalıdır.

{% highlight shell %}
FROM ubuntu
{% endhighlight %}

Docker build işlemi yaptığımızda bir ubuntu image’ı oluşacak. Bu image zaten var olan “ubuntu” image ile bire bir aynıdır, çünkü image üzerinde hiçbir değişiklik yapmadık.

## RUN

Temel image’ın çalıştırılacağı işletim sistemi (en minimal hali) üstünde komutlar çalıştırmamızı sağlar.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
{% endhighlight %}

Bir ubuntu hazırlanıyor ve içinde komutlar çalıştırılıyor. Varolan paketler güncelleniyor ve sistem üzerine “telnet” kuruluyor.

Not: Debian temelli Linux dağıtımların da “apt-get” paket yöneticisidir.
Not: Komutlarınızı farklı RUN talimatları ile kullanabileceğiniz gibi tek RUN talimatı içinde de tanımlayabilirsiniz.

{% highlight shell %}
FROM ubuntu
RUN apt-get update && apt-get install -y telnet
{% endhighlight %}

## ENV

Temel image içinde bulunan işletim sistemi (en minimal hali) içinde ortam değişkenleri eklemizi sağlar.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
{% endhighlight %}

Bu Image kullanılarak bir konteyner oluştuğunda ortam değişkenlerinde APP_URL görebilirsiniz.

## COPY

Yolu belirtilen dosyaları, konteyner’in dosya sisteminin belirtilen dizinine kopyalar.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
COPY app/ my_app/
{% endhighlight %}

“app” dizini altında bulunan dosyaları, konteyner’a “my_app” dizini altına kopyalar.

## WORKDIR

Dockerfile’da onu izleyen herhangi bir RUN, CMD, ENTRYPOINT, COPY ve ADD talimatları için çalışma dizinini ayarlar.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
{% endhighlight %}

WORKDIR komutunda sonra çalışacak olan komutların hepsi “my_workplace” dizini altında çalışacaklar. “my_app” dizini “my_workplace” altında oluşacaktır.

## ADD

COPY ve ADD, benzer amaçlara hizmet eden Dockerfile talimatlarıdır. COPY yalnızca ana bilgisayarınızdan konteyner dizinine kopyalamanıza izin verir. ADD talimatı ile bunu yapabileceğiniz gibi bir URL üzerinden de dosya kopyalama yapabilirsiniz.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
ADD https://cdn.pixabay.com/photo/2020/05/12/13/38/common-blue-butterfly-5163066_1280.jpg images/
{% endhighlight %}

Belirtilen URL’den jpg formatında ki dosyayı images dizini altına kopyalar.

## ARG

Docker build komutuyla derleme zamanında konteyner’a iletebilecekleri bir değişkeni tanımlar.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
ADD https://cdn.pixabay.com/photo/2020/05/12/13/38/common-blue-butterfly-5163066_1280.jpg images/
ARG MY_CONNECTION_STRING
ENV CONNECTION_STRING=$MY_CONNECTION_STRING
{% endhighlight %}

“build-arg” argümanını kullanarak “MY_CONNECTION_STRING=blabla” build aşamasında değerleri geçiyoruz. Daha sonra ENV talimatı ile ortam değişkeni olarak ekliyoruz.

{% highlight shell %}
docker build -t anilatalay/example:1.0.1 --build-arg MY_CONNECTION_STRING=blabla .
{% endhighlight %}

## VOLUME

VOLUME komutu, belirtilen adda bir bağlama noktası oluşturur ve bunu yerel ana bilgisayardan veya diğer kapsayıcılardan harici olarak monte edilmiş birimleri tutuyor olarak işaretler.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
ADD https://cdn.pixabay.com/photo/2020/05/12/13/38/common-blue-butterfly-5163066_1280.jpg images/
ARG MY_CONNECTION_STRING
ENV CONNECTION_STRING=$MY_CONNECTION_STRING
VOLUME /images
{% endhighlight %}

## EXPOSE

Docker’a, konteynerin çalışma zamanında belirtilen portu (ağ bağlantı noktaları) dinlediğini bildirir.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
ADD https://cdn.pixabay.com/photo/2020/05/12/13/38/common-blue-butterfly-5163066_1280.jpg images/
ARG MY_CONNECTION_STRING
ENV CONNECTION_STRING=$MY_CONNECTION_STRING
VOLUME /images
EXPOSE 80
{% endhighlight %}

Bu senaryo için çok anlamlı olmasada kullanımını göstermek istedim.

## CMD

Konteyner çalıştığı anda CMD komutu ile belirtilen varsayılan komutu çalıştırır.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
ADD https://cdn.pixabay.com/photo/2020/05/12/13/38/common-blue-butterfly-5163066_1280.jpg images/
ARG MY_CONNECTION_STRING
ENV CONNECTION_STRING=$MY_CONNECTION_STRING
VOLUME /images
EXPOSE 80
CMD [ "ls" ]
{% endhighlight %}

CMD talimatı verilen “ls” komutu ile konteyner çalıştığı anda WORKDIR talimatın da belirtilen dizinin içindeki dosya ve klasörleri listeyecek.

{% highlight shell %}
docker run --rm anilatalay/example:1.0.1
{% endhighlight %}

Not: Çalıştır komutuna bir argüman geçerseniz, CMD talimatı geçersiz sayılır. Çıktı olarak “ls” yerine “echo merhaba” komutu çalıştırılacak.

{% highlight shell %}
docker run --rm anilatalay/example:1.0.1 echo merhaba
{% endhighlight %}

## ENTRYPOINT

Konteyner’ın nasıl çalışacağını yapılandırmak için kullanılan diğer talimattır. CMD’de olduğu gibi, bir komut ve parametreler belirlemeniz gerekir.

{% highlight shell %}
FROM ubuntu
RUN apt-get update
RUN apt-get install -y telnet
ENV APP_URL=https://medium.com/@anilatalay
WORKDIR /my_workplace
COPY app/ my_app/
ADD https://cdn.pixabay.com/photo/2020/05/12/13/38/common-blue-butterfly-5163066_1280.jpg images/
ARG MY_CONNECTION_STRING
ENV CONNECTION_STRING=$MY_CONNECTION_STRING
VOLUME /images
EXPOSE 80
ENTRYPOINT [ "echo", "anilatalay" ]
{% endhighlight %}

ENTRYPOINT talimatı verilen “echo anilatalay” komutu ile konteyner çalıştığı anda ekrana çıktı olarak “anilatalay” yazacaktır.

Not: Kullanılan komutların bazıları windows konteyner’lar da sorun çıkartabilir. Detaylı bilgi için dokümantasyon sayfasına göz atabilirsiniz.

Öğrenilecek çok şey, gezilecek çok yer var. Her bir yazımda gerçek dünyaya bir adım daha yaklaşacağız. Bir sonraki postumda görüşmek üzere.
