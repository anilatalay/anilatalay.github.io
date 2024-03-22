---
title: "ASP.NET Core: Distributed Caching (Redis)"
layout: post
---

Merhaba sevgili okur,

ASP.NET Core uygulamaları için redis’de temel okuma-yazma-silme işlemlerini nasıl gerçekleştirebileceğinizi açıklamaya çalıştım.

Hemen bir ASP.NET Core Web API projesi oluşturalım.

{% highlight shell %}
dotnet new webapi --name "Example"
{% endhighlight %}

![docker](/assets/images/1_yA0QEz4nzxs5q06lkuErOg.png)

Projemiz de redis’in temel özelliklerini kullanmak için gerekli nuget paketini projeye dahil ediyoruz.

Not: Daha geniş kapsam da redis API’sini kullanmamıza olanak sağlayan paketler bulunuyor. İkinci bölümde detaylı olarak yer vereceğim.

{% highlight shell %}
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis --version 3.1.10
{% endhighlight %}

İlgili paketi projeye ekledikten sonra “Startup.cs” dosyasında bulunan servis tanımlarından “DistributedCache” servisini aktif hale getiriyoruz. Tanımlama yaparken yapılandırma ayarları bölümünde kullanacağamız redis’e ait host ve port bilgilerini yazıyoruz.

{% highlight c# %}
services.AddStackExchangeRedisCache(option =>
{
option.Configuration = "localhost:6379";
});
{% endhighlight %}

Not: Host, port, connection string veya benzer bilgileri code içerisinde kullanmamaya özen gösterelim.

{% highlight c# %}
// appsettings.json
{
"Redis": {
"Host": "localhost",
"Port": "6379"
}
}
{% endhighlight %}

Bağımlılık enjeksiyonu (Dependency Injection) yardımıyla IConfiguration arayüzü (interface) ile gelen methodları kullanarak redis bilgilerini artık “appsettings.json” dosyasından okuyoruz.

{% highlight c# %}
services.AddStackExchangeRedisCache(option =>
{
option.Configuration = $”{Configuration[“Redis:Host”]}:{Configuration[“Redis:Port”]}”;
});
{% endhighlight %}

{% highlight c# %}
using System; using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
namespace ProjectRedis {
public class Startup {
public Startup(IConfiguration configuration) { Configuration = configuration; }
public IConfiguration Configuration { get; }
public void ConfigureServices(IServiceCollection services) {
services.AddStackExchangeRedisCache(option => { option.Configuration = $"{Configuration["Redis:Host"]}:{Configuration["Redis:Port"]}"; }); Console.WriteLine($"{Configuration["Redis:Host"]}:{Configuration["Redis:Port"]}");
services.AddControllers(); } public void Configure(IApplicationBuilder app, IWebHostEnvironment env) {
if (env.IsDevelopment()) {
app.UseDeveloperExceptionPage();
}
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthorization();
app.UseEndpoints(endpoints => { endpoints.MapControllers(); });
}
}
}
{% endhighlight %}

Redis üzerinde örnek işlemleri gerçekleştirmek adına user endpoint’lerini tanımlıyoruz. İlgili class’ın constructor’ın da IDistributedCache arayüzünü (interface) bağımlılık enjeksiyonu (Dependency Injection) yardımıyla geçiyoruz.

{% highlight c# %}
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Caching.Distributed;
namespace Example.Controllers { [ApiController] [Route("[controller]")]
public class UsersController : ControllerBase {
private readonly IDistributedCache \_distributedCache;
public UsersController(IDistributedCache distributedCache) { \_distributedCache = distributedCache; }
[HttpGet] public IActionResult Get() { return Ok(); }
[HttpPost] public IActionResult Post() { return Ok(); }
[HttpDelete] public IActionResult Delete() { return Ok(); }
}
}
{% endhighlight %}

IDistributedCache arayüzü ile gelen temel redis methodları şu an için ihtiyacımızı karşılıyor. Eklemiş olduğumuz nuget paketi Microsoft tarafından redis’in temel özelliklerini kullanmamız için geliştirilmiş olup ileri seviye özellikler için kullanılmamaktadır.

![docker](/assets/images/1_AB5YYxG0XSUdMzwW06I8Wg.png)

Evet artık yazma-okuma gibi işlemleri yapabiliriz. “SetString” method’unu kullanarak o an ki tarih-saat bilgisini string olarak redis’e yazıyoruz. Kullanılan method’ların async versiyonları da bulunuyor.

{% highlight c# %}
\_distributedCache.SetString(“DATE_NOW”, DateTime.Now.ToString());
await \_distributedCache.SetStringAsync(“DATE_NOW”, DateTime.Now.ToString());
{% endhighlight %}

Redis de bulunan “DATE_NOW” key’e ait değeri “GetString” medhod’unu kullanarak okuyoruz.

{% highlight c# %}
var DATE_NOW = \_distributedCache.GetString(“DATE_NOW”);
{% endhighlight %}

“DATE_NOW” key’ini siliyoruz.

{% highlight c# %}
\_distributedCache.Remove("DATE_NOW");
{% endhighlight %}

Veri oluşturulduktan sonra verilen süre sonunda otomatik olarak redis’den silinir.

{% highlight c# %}
var options = new DistributedCacheEntryOptions
{
AbsoluteExpiration = DateTime.Now.AddSeconds(10)
};

\_distributedCache.SetString("DATE_NOW", DateTime.Now.ToString(), options);
{% endhighlight %}

Bir complex type’ı redis’e yazmak için “Json” veya “Binary” serialize işlemlerininden birini tercih edebilirsiniz. Fakat genelde “Json” serialize yöntemi kullanılır. “Binary” serialize genelde dosyalarla ilgili işlemler de tercih edilir.

Hemen bir user model class’ı ekliyoruz.

{% highlight c# %}
// User.cs
public class User
{
public int Id { get; set; }
public string FirstName { get; set; }
public string LastName { get; set; }
}
{% endhighlight %}

Class’ımızdan bir nesne örneği alıyoruz.

{% highlight c# %}
var user = new User {
Id = 1,
FirstName = "Anıl",
LastName = "Atalay"
};
{% endhighlight %}

User model class’ımızı json serialize işlemi yapmak için projemize bir nuget paketi ekliyoruz. İlgili paket Asp.Net Core API projelerinde varsayılan olarak gelmiyor!

{% highlight shell %}
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson --version 3.1.10
{% endhighlight %}

Json serialize işlemini gerçekleştiriyoruz.

{% highlight c# %}
var userData = JsonConvert.SerializeObject(user);
{% endhighlight %}

Artık elimizde bulunan string user datasını redis’e yazabiliriz. Birden fazla user yazılabilir düşüncesi ile benzersiz bir key oluşturuyoruz.

{% highlight c# %}
await _distributedCache.SetStringAsync($”USER_{user.Id}”, userData);
{% endhighlight %}

Redis’e yamış olduğumuz user nesnesini okuyalım.

{% highlight c# %}
var userData = await \_distributedCache.GetStringAsync("USER_1");
{% endhighlight %}

Gelen user string datasını User class’ına deserialize işlemi gerçekleştireceğiz.

{% highlight c# %}
var user = JsonConvert.DeserializeObject<User>(userData);
{% endhighlight %}

Benzer işlemleri binary içinde yapalım. User nesnemizin json string halini byte dizisine çeviriyoruz ve “SetAsync” method’unu kullanarak redis’e yazıyoruz.

{% highlight c# %}
var user = new User
{
Id = 1,
FirstName = "Anıl",
LastName = "Atalay"
};

var userData = JsonConvert.SerializeObject(user);

var userByte = Encoding.UTF8.GetBytes(userData);

await _distributedCache.SetAsync($"USER_{user.Id}", userByte);
{% endhighlight %}

Redis’den okurken ise, yazarken yaptığımız işlemlerin tersini yapıyoruz.

{% highlight c# %}
var userByte = await \_distributedCache.GetAsync("USER_1");

var userData = Encoding.UTF8.GetString(userByte);

var user = JsonConvert.DeserializeObject<User>(userData);
{% endhighlight %}

Redis üzerinde temel işlemleri gerçekleştirdik. Artık oluşturmuş olduğumuz projeye devam edelim. En son user endpoint’lerini eklemiştik. GET, POST ve DELETE işlemlerine başlayalım.

{% highlight c# %}
using System.Threading.Tasks;
using Example.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Caching.Distributed;
using Newtonsoft.Json; namespace Example.Controllers { [ApiController] [Route("[controller]")]
public class UsersController : ControllerBase {
private readonly IDistributedCache _distributedCache;
public UsersController(IDistributedCache distributedCache) { \_distributedCache = distributedCache; }
[HttpGet("{id}")] public async Task Get([FromRoute] int id) {
if (id == default) return NotFound();
var userData = await \_distributedCache.GetStringAsync($"USER_{id}");
if (userData == null) return NotFound("User not found."); var user = JsonConvert.DeserializeObject(userData); return Ok(user); }
[HttpPost] public async Task Post([FromBody] User user) { if (user == null) return BadRequest();
var userKey = $"USER_{user.Id}"; 
var userData = JsonConvert.SerializeObject(user); await _distributedCache.SetStringAsync(userKey, userData); return Ok(user); } 
[HttpDelete("{id}")] public async Task Delete([FromRoute] int id) { 
  if (id == default) return NotFound(); 
  await _distributedCache.RemoveAsync($"USER\_{id}"); return Ok(id);
}
}
}
{% endhighlight %}

Evet artık user endpoint’leri üzerinden redis’e veri yazabiliyoruz. Gerçek dünya da bu işlemler veritabanı ile gerçekleştirilir. Uygulamamızın yapısına bağlı olarak süreç değişebilir. İlk isteği veritabanı üzerinden çekip dönebilir, daha sonra veriyi redis’e yazıp diğer isteklere redis’de ki veriler ile cevap verilebilir.

Öğrenilecek çok şey, gezilecek çok yer var. Bir sonraki postumda görüşmek üzere.
