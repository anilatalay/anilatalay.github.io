---
title: "Docker: Node.js Uygulamalarını Dockerize Etmek"
layout: post
---

Merhaba sevgili okur,

Node.js’de yazılmış bir uygulama için nasıl docker image’ı oluşturabileceğinizi anlatmaya çalışacağım. Bir önce ki yazım da Docker: Dockerfile ile Image Oluşturmak’ı paylaşmıştım. Keyifli okumalar.

Temel amacımız bir Node.js uygulamasını dockerize etmek olduğu için uygulamanın seviyesinin bizim senaryomuz için yeterli olduğu kanaatindeyim. “package.json” dosyası oluşturduktan sonra ihtiyacımız olan paketleri yüklüyoruz.

{% highlight shell %}
npm init -y
npm install
{% endhighlight %}

{% highlight json %}
{ "name": "app", "version": "1.0.0", "description": "", "main": "server.js", "scripts": { "start": "node server.js" }, "keywords": [], "author": "", "license": "ISC", "dependencies": { "express": "^4.17.1" } }
{% endhighlight %}

“server.js” dosyasını oluşturup hayatımıza devam ediyoruz.

{% highlight js %}
const express = require("express");
const PORT = 2323; const HOST = "0.0.0.0";
const app = express();
app.get("/", (req, res) => { res.send("Hello World"); });
app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
{% endhighlight %}

Sonuç olarak; ekrana “Hello World” yazan bir servisimiz mevcut. :)

## Uygulamayı çalıştıralım

{% highlight shell %}
npm run start
{% endhighlight %}

## Dockerfile

Uygulamamızın ana dizinine Dockerfile dosyasını ekleyelim. Dockerhub üzerinden bulunan resmi Node.js image’nı kullanacağız.

Not: Node.js kullanılabilecek farklı sistem ve versiyonlara ait image’lar bulunuyor. Uygulamanızın ihtiyaç duyduğu versiyon ve sisteme ait etikete sahip image’ı temel (base) image olarak tanımayabilirsiniz.

{% highlight shell %}
FROM node:15
{% endhighlight %}

Not: Bir temel (base) image kullanırken versiyon numarası vermeye özen gösterin. Aksi durumda “node” image’ının “latest” versiyonu çekecektir. Uygulamanızın eski versiyona bağımlılığı var ise bazı problemlere sebebiyet verebilir ve uygulamanız düzgün çalışmayabilir.

Konteyner’da ki çalışma dizinini “/app” olarak ayarlayalım. İsimlendirme tamamen sizin insiyatifinize kalmış. Bu dizini uygulama dosyaları ile işlemleri gerçekleştirmek için kullanacağız.

{% highlight shell %}
FROM node:15
WORKDIR /app
{% endhighlight %}

Uygulama dosyalarını “/app” dizinine kopyalayalım.

{% highlight shell %}
FROM node:15
WORKDIR /app
COPY /src .
{% endhighlight %}

Not: Geliştirme (development) ortamında kopyalama işlemi yapacağımız için, uygulama dizini içerisinde çalışma zamanından kalan dosyalar, geliştirme (development) ortamına ait ayar dosyaları veya code editörüne ait istenmeyen dosyalar olabilir. Bunlara benzer istenmeyen veya ihtiyaç olmayan dosyaları konteyner içine kopyalamak istemeyebilirsiniz. Problemi çözmek adına uygulamamızın ana dizinine “.dockerignore” dosyasını ekleyelim ve konteyner’a aktarmak istemediğimiz dosyaları belirleyelim.

{% highlight shell %}
// .dockerignore
\*/node_modules
npm-debug.log
.idea
.vscode
{% endhighlight %}

Not: Uygulama dosyalarını kopyalama işlemi ADD talimatı kullanılarak bir source-code repository üzerinden çekilebilir.

Uygulama dosyalarımız konteyner içinde olduğuna göre artık bağımlılıkları yükleyebiliriz. “package.json” dosyasında tanımlanmış olan paketleri yüklemeye başlayalım.

{% highlight shell %}
FROM node:15
WORKDIR /app
COPY /src .
RUN yarn install
{% endhighlight %}

Not: Uygulamanızın ihtiyaç duyduğu değişiklik, yükleme vb işlemleri ekleyebilirsiniz.

Hissediyorum sona yaklaştık. Konteyner başlatılırken çalışacak olan komutları ekliyoruz.

{% highlight shell %}
FROM node:15
WORKDIR /app
COPY /src .
RUN yarn install
CMD yarn run start
{% endhighlight %}

Not: Bir “sh” benzeri kurulum veya çalıştırma dosyası oluşturup onu da başlangıç komutu olarak tanımlayabilirsiniz.

Son olarak; uygulamamız bir web application olduğu için hangi port (2323) üzerinden yayın yapacağımızıda belirtmemiz gerekiyor. Aksi takdir de uygulamaya konteyner’ın dışından kimse erişmez.

{% highlight shell %}
FROM node:15
WORKDIR /app
COPY /src .
RUN yarn install
CMD yarn run start
EXPOSE 2323
{% endhighlight %}

## Dockerize Edelim

Dockerfile’mız hazır olduğuna göre artık dockerize etme kısmına geçebiliriz. “docker build” komutunu çalıştırarak image oluşturmakla başlayalım.

{% highlight shell %}
docker build -t anilatalay/my_node_app:1.0.1 .
{% endhighlight %}

Artık uygulamamızın bir image’ı mevcut, bu image’ı kullanarak bir konteyner oluştuyoruz.

{% highlight shell %}
docker run --rm -p 80:2323 anilatalay/my_node_app:1.0.1
{% endhighlight %}

Uygulama konteyner içinde “2323" portunu kullanırken dışarıya “80” portu üzerinden yayınlanıyor.

![docker](/assets/images/1_yMbkEs9sv8fxWF8YyVCwMg.png)

![docker](/assets/images/1_xtzYKYiM9AcSPTqSBi0WYw.png)

## Uygulama Image’nı Paylaşalım

Docker cli üzerinden dockerhub’a giriş yapıyoruz.

{% highlight shell %}
docker login
{% endhighlight %}

![docker](/assets/images/1_F26rXwA-bqlJRsEvePCfsg.png)

Dockerhub’a image’mızı gönderiyoruz.

{% highlight shell %}
docker push anilatalay/my_node_app
{% endhighlight %}

![docker](/assets/images/1_QRXXqd4BsJbs5DmzpyiYJg.png)

Hayırlı olsun. Dockerhub üzerinde nur topu gibi image’mız var. (hub.docker.com) Artık isteyen herkes image’mızı çekebilir :)

![docker](/assets/images/1_4scU8j-IrOjXLbigCfC_tA.png)

Öğrenilecek çok şey, gezilecek çok yer var. Her bir yazım da gerçek dünyaya bir adım daha yaklaşacağız. Bir sonraki postumda görüşmek üzere.
