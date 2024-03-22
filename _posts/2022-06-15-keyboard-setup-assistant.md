---
title: "macOS harici klavye ayarlarını sıfırlama"
layout: post
---

Merhaba sevgili okur,

Mac'e harici bir klavye taktığımızda sistem otomatik olarak "Keyboard Setup Assistant" uygulamasını çalıştırır ve bu şekilde kurulum işlemini gerçekleştilebiliriz. Maalesef klavyemiz için bu kurulum işlemini bir kez yaptıktan sonra klavye ayar bölümünden ilgili seçenek kalkıyor veya ulaşılmaz oluyor.

![docker](/assets/images/macos-keyboard.png)

Klavye ayarlarını sıfırlamak istediğimizde klavye bölümünde bulunan ilgili ayarı veya direk "Keyboard Setup Assistant" uygulamasını açamıyoruz.

"Keyboard Setup Assistant" uygulamasının dizini;

{% highlight shell %}
/System/Library/CoreServices/KeyboardSetupAssistant.app
{% endhighlight %}

Klavye ayarlarını sıfırlamak için "Library/Preferences" altında bulunan "com.apple.keyboardtype.plist" dosyasını silmemiz gerekiyor. Terminal üzerinde aşağıda bulunan komut ile silme işlemini gerçekleştirebilirsiniz.

{% highlight shell %}
sudo rm -rf /Library/Preferences/com.apple.keyboardtype.plist
{% endhighlight %}

Komutu çalıştırdıktan sonra bilgisayarı yeniden başlattığınızda "Keyboard Setup Assistant" uygulaması otomatik olarak başlatılacaktır.

![docker](/assets/images/keyboard-setup-assistant-2.png)

"Keyboard Setup Assistant" uygulaması otomatik olarak başlamazsa klavye ayarları bölümünden manuel olarak başlatabilirsiniz.

![docker](/assets/images/keyboard-setup-assistant-1-1.png)

Bir sonraki postumda görüşmek üzere.
