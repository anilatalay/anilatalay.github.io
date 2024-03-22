---
title: "Docker: ASP.NET Core Uygulamalarını Dockerize Etmek"
layout: post
---

Merhaba sevgili okur,

ASP.NET Core’da yazılmış bir uygulama için nasıl docker image’ı oluşturabileceğinizi anlatmaya çalışacağım. Bir önce ki yazım da Docker: Dockerfile ile Image Oluşturmak’ı paylaşmıştım. Keyifli okumalar.

Hemen başlayalım; “dotnet cli”yı kullanarak bir web api projesi oluşturalım.

{% highlight shell %}
dotnet new webapi --name "MyApp"
{% endhighlight %}

![docker](/assets/images/1_VKnUhQB2waisYKpRyor_-g.png)

## Uygulamayı çalıştıralım

Uygulamayı test amaçlı çalıştıralım ve devam edelim.

{% highlight shell %}
dotnet run
{% endhighlight %}

## Dockerfile

Uygulamamızın ana dizinine Dockerfile dosyasını ekleyelim. Temel (base) image tanımımızı yapacağız.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
{% endhighlight %}

Not: Microsoft resmi image’larını Dockerhub üzerinden paylaştığı gibi aynı zaman da kendi container registry’si üzerinden de paylaşıyor.

Not: Temel (base) image’mıza dikkatli baktığımızda bir ASP.NET Core SDK’sı olduğu anlaşılıyor. Fakat bizim uygulamamızın çalışması için ASP.NET Core Runtime’ı yeterli iken neden SDK image’ını kullanıyoruz. Aslında yerine değil, birlikte. SDK image’ını uygulamayı derlemek için, Runtime image’ını ise uygulamayı çalıştırmak için kullanacağız. Detaylı bilgi için dokümantasyon sayfasına göz atabilirsiniz. (keyword: multi-stage)

Not: Uygulamanızı kendi bilgisayarınızda belirli işletim sistemleri için kişileştirilmiş ayarlar ile derleyebilir ve bu çıktılar üzerinden de image oluşturabilirsiniz. Fakat bu yöntem tavsiye edilmiyor!

Not: ASP.NET Core uygulamalarını Runtime’ı ile birlikte paketlemek ve yayınlamak mümkündür. Fakat dosya boyutunu ciddi oranda artıracaktır.

Konteyner’da ki çalışma dizinini “/source” olarak ayarlayalım. İsimlendirme tamamen sizin insiyatifinize kalmış. Bu dizini uygulama kaynak kodlarını kopyalamak için kullanacağız.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
{% endhighlight %}

Uygulama kaynak kodlarını “/source” dizinine kopyalayalım.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
{% endhighlight %}

Not: Geliştirme (development) ortamında kopyalama işlemi yapacağımız için, uygulama dizini içerisinde çalışma zamanından kalan dosyalar, geliştirme (development) ortamına ait ayar dosyaları veya code editörüne ait istenmeyen dosyalar olabilir. Bunlara benzer istenmeyen veya ihtiyaç olmayan dosyaları konteyner içine kopyalamak istemeyebilirsiniz. Problemi çözmek adına uygulamamızın ana dizinine “.dockerignore” dosyasını ekleyelim ve konteyner’a aktarmak istemediğimiz dosyaları belirleyelim.

{% highlight shell %}
// .dockerignore
.git
.gitignore
.vscode
Dockerfile
README.md
\*\*/.DS_Store
{% endhighlight %}

Image’ın temelini oluşturan işletim sistemi (minimal) üzerinde bulunan paketleri güncelleyelim. Bu adım uygulamaların ihtiyaç duyabileceği paketlerin yüklenebilmesi veya güncellenmesi için örnek olarak eklenmiştir.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
{% endhighlight %}

Kaynak kodlarımız artık konteyner içinde olduğuna göre derleme aşamasına geçebiliriz. Konteyner’da ki çalışma dizinini “/source/MyApp” olarak değiştirelim. Burada ki dizin sizin uygulamızın ana dizinidir.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
{% endhighlight %}

Uygulamamızı restore edelim.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
RUN dotnet restore MyApp.csproj
{% endhighlight %}

Uygulamayı “/app” dizini altına publish alalım.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
RUN dotnet restore MyApp.csproj
RUN dotnet publish MyApp.csproj --output /app/ --configuration Release
{% endhighlight %}

Evet uygulamamız artık yayınlamaya hazır hale geldi. Şimdi asıl image’ı oluşturacak dockerfile talimatlarına geçelim. Temel (base) image tanımımızı yapalım.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
RUN dotnet restore MyApp.csproj
RUN dotnet publish MyApp.csproj --output /app/ --configuration Release
// ------------------------------------------------------------
// Runtime
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
{% endhighlight %}

Konteyner’da ki çalışma dizinini “/app” olarak ayarlayalım.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
RUN dotnet restore MyApp.csproj
RUN dotnet publish MyApp.csproj --output /app/ --configuration Release
// ------------------------------------------------------------
// Runtime
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR /app
{% endhighlight %}

Önceki katmanlar da publish işlemini halletmiştik. Şimdi bu işlemlerin sonucunda oluşan derlenmiş uygulamamızı “/app” dizinine kopyalayalım.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
RUN dotnet restore MyApp.csproj
RUN dotnet publish MyApp.csproj --output /app/ --configuration Release
// ------------------------------------------------------------
// Runtime
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR /app
COPY --from=builder /app .
{% endhighlight %}

Konteyner başlatılırken çalışacak olan komutları ekliyoruz.

{% highlight shell %}
// Dockerfile
// Builder
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS builder
WORKDIR /source
COPY src/ .
RUN apt update
WORKDIR /source/MyApp
RUN dotnet restore MyApp.csproj
RUN dotnet publish MyApp.csproj --output /app/ --configuration Release
// ------------------------------------------------------------
// Runtime
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR /app
COPY --from=builder /app .
{% endhighlight %}

## Dockerize Edelim

Dockerfile’mız hazır olduğuna göre artık dockerize etme kısmına geçebiliriz. “docker build” komutunu çalıştırarak image oluşturmakla başlayalım.

{% highlight shell %}
docker build -t anilatalay/my_dotnetcore_app:1.0.1 .
{% endhighlight %}

Artık uygulamamızın bir image’ı mevcut, bu image’ı kullanarak bir konteyner oluştuyoruz.

![docker](/assets/images/1_KnlKNrgaUBlu0W2Yl2xBVA.png)

{% highlight shell %}
docker run --rm -p 8080:80 anilatalay/my_dotnetcore_app:1.0.1
{% endhighlight %}

![docker](/assets/images/1_pPVW63dZohAKFhz0B2Wcrg.png)

Uygulama konteyner içinde “8080” portunu kullanırken dışarıya “80” portu üzerinden yayınlanıyor. (http://localhost:8080/WeatherForecast)

![docker](/assets/images/1_iCx3cS46in5UfUGVtALkew.png)

## Uygulama Image’nı Paylaşalım

Docker cli üzerinden dockerhub’a giriş yapıyoruz.

{% highlight shell %}
docker login
{% endhighlight %}

![docker](/assets/images/1_fz7iIIF5a_S5609i2S9-Fw.png)

Dockerhub’a image’mızı gönderiyoruz.

{% highlight shell %}
docker push anilatalay/my_dotnetcore_app
{% endhighlight %}

![docker](/assets/images/1_LMQBGzspeEUiwqur6CPGbA.png)

Hayırlı olsun. Dockerhub üzerinde nur topu gibi image’mız var. (hub.docker.com) Artık isteyen herkes image’mızı çekebilir :)

![docker](/assets/images/1_Do5ilf7JVaMjmXOCdWQnrQ.png)

Öğrenilecek çok şey, gezilecek çok yer var. Bir sonraki postumda görüşmek üzere.
