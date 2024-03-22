---
title: "ASP.NET Core: Cache’lemek (In Memory Caching)"
layout: post
---

Merhaba sevgili okur,

ASP.NET Core uygulamalarında bellek (memory) üzerinde okuma-yazma gibi işlemleri nasıl gerçekleştirebileceğinizi açıklamaya çalıştım.

Cache’lemek (önbelleğe alma) dediğimiz işlem, sık kullanılan veya nadiren değişen dataların geçici bir depolama alanının da saklamasıdır. Önbelleğe alma, performansı ve ölçeklenebilirliği artırır. Verileri önbelleğe aldığımız da, verilerin kopyası geçici depolama alanında saklanır. Böylelikle bir daha ki sefere aynı veriler talep edildiğinde geçici depolama alanından alınır ve ana veri kaynağından çok daha hızlı yüklenir.

ASP.NET Core uygulamalarında bellek (memory) üzerinde işlemler gerçekleştirebilmek için built-in olarak gelen MemoryCache servisini aktif hale getirmemiz gerekiyor. Bağımlılık enjeksiyonu (Dependency Injection) yardımıyla IMemoryCache arayüzü (interface) ile gelen methodları kullanarak bellek (memory) üzerinde okuma-yazma işlemleri gerçekleştirilir.

Hemen bir ASP.NET Core Web API projesi oluşturalım.

{% highlight shell %}
dotnet new webapi --name "Example"
{% endhighlight %}

![docker](/assets/images/1_EtEeq1Yl_ffDMdspODpqFA.png)

Projeyi çalıştıralım. “https://localhost:5001/WeatherForecast”

{% highlight shell %}
dotnet run
{% endhighlight %}

![docker](/assets/images/1_iTWyqMjmv0WH_lqxzGKyyA.png)

“Startup.cs” dosyasında ihtiyaç duyduğumuz servis tanımını ve yapılandırma ayarlarını ekliyoruz.

{% highlight shell %}
// Startup.cs
services.AddMemoryCache();
{% endhighlight %}

{% highlight c# %}
// Startup.cs
using Microsoft.AspNetCore.Builder; using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration; using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
namespace Example {
public class Startup {
public Startup(IConfiguration configuration) {
Configuration = configuration;
}
public IConfiguration Configuration { get; }
public void ConfigureServices(IServiceCollection services) {
services.AddMemoryCache();
services.AddControllers();
}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env) {
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

Bellek (memory) üzerinde örnek işlemleri gerçekleştirmek adına user endpoint’lerini tanımlıyoruz.

{% highlight c# %}
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Caching.Memory; namespace Example.Controllers {
[ApiController] [Route("[controller]")]
public class UserController : ControllerBase {
private IMemoryCache \_memoryCache;
public UserController(IMemoryCache memoryCache) { \_memoryCache = memoryCache; }

[HttpGet] public IActionResult Get() { return Ok(); }

[HttpPost] public IActionResult Post() { return Ok(); } } }
{% endhighlight %}

İpucu: Bellek (memory) üzerinde veriler key-value şeklinde tutulur. Key benzersiz olmalıdır. Aksi takdir de varolan key’in değeri güncellenir.

“DATE_NOW” key’ine o an ki tarih ve saat bilgisini string olarak belleğe yazıyoruz.

İpucu: Mehodlar generic olduğu için her tipte veri kaydedebilirsiniz. Serialization işlemini IMemoryCache kendisi gerçekleştiriyor.

{% highlight c# %}
\_memoryCache.Set<string>("DATE_NOW", DateTime.Now.ToString());
{% endhighlight %}

Bellekten “DATE_NOW” key’ine ait değeri çekiyoruz.

{% highlight c# %}
var value = \_memoryCache.Get<string>("DATE_NOW");
{% endhighlight %}

Bellekten “DATE_NOW” key’ini siliyoruz.

{% highlight c# %}
\_memoryCache.Remove(“DATE_NOW”);
{% endhighlight %}

Bir key’in bellekte var olup olmadığını kontrol etmek için “TryGetValue” methodu kullanılır. Key bellekte var ise method “true” döner ve value değişkenine değerini atar. Aksi durumda “false” döner.

{% highlight c# %}
if (\_memoryCache.TryGetValue<string>("DATE_NOW", out string value))
{
...
}
{% endhighlight %}

Verilerin bellekte tutulmasıyla ilgili çeşitli yapılandırma ayarları bulunuyor. Veri oluşturulduktan sonra verilen süre sonunda otomatik olarak bellekten silinir.

{% highlight c# %}
var options = new MemoryCacheEntryOptions
{
AbsoluteExpiration = DateTime.Now.AddSeconds(10)
};
\_memoryCache.Set<string>("DATE_NOW", DateTime.Now.ToString(), options);
{% endhighlight %}

Veri oluşturulduktan sonra verilen süre boyunca veriye erişim olmaz ise, otomatik olarak bellekten silinir.

{% highlight c# %}
var options = new MemoryCacheEntryOptions
{
SlidingExpiration = TimeSpan.FromSeconds(10)
};
\_memoryCache.Set<string>("DATE_NOW", DateTime.Now.ToString(), options);
{% endhighlight %}

Bellek dolduğu anda yeni eklenecek olan verilere yer açmak adına hangi verilerin önce silineceğini belirlemek için “Priority” property’si kullanılır. Önem sırasına göre tanımlamalar yapabilirsiniz.

{% highlight c# %}
// CacheItemPriority.Low
// CacheItemPriority.Normal
// CacheItemPriority.High
// CacheItemPriority.NeverRemove
var options = new MemoryCacheEntryOptions
{
Priority = CacheItemPriority.Low
};
\_memoryCache.Set<string>("DATE_NOW", DateTime.Now.ToString(), options);
{% endhighlight %}

Bellekten bir veri silindiği zaman bir event oluşturulur ve “RegisterPostEvictionCallback” method’unda ki delege tetiklenir. Böylelikle silinen verinin neden silindiği gibi bilgilere sahip olabilir veya başka bir takım işlemler gerçekleştirebiliriz.

{% highlight c# %}
var options = new MemoryCacheEntryOptions();
options.RegisterPostEvictionCallback((key, value, reason, state) =>
{
...
});
\_memoryCache.Set<string>("DATE_NOW", DateTime.Now.ToString(), options);
{% endhighlight %}

Bir complex type’ı belleğe yazmak için get-set method’larının generic yapısı kullanılır.

{% highlight c# %}
// User.cs
public class User
{
public int Id { get; set; }
public string FirstName { get; set; }
public string LastName { get; set; }
}
{% endhighlight %}

User sınıfından bir nesne türetip belleğe yazıyoruz. Birden fazla user yazılabilir düşüncesi ile benzersiz bir key oluşturuyoruz.

{% highlight c# %}
var user = new User
{
Id = 1,
FirstName = "Anıl",
LastName = "Atalay"
};
_memoryCache.Set<User>($"USER_{user.Id}", user);
{% endhighlight %}

User’a ait bellekte ki veriyi okurken get generic method’unu kullanıp bir user nesnesi elde ediyoruz.

{% highlight c# %}
User user = \_memoryCache.Get<User>("USER_1");
{% endhighlight %}

Bellek üzerinde temel işlemleri gerçekleştirdik. Artık oluşturmuş olduğumuz projeye devam edelim. En son user endpoint’lerini eklemiştik. GET, POST ve DELETE işlemlerine başlayalım.

{% highlight c# %}
using System; using Example.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Caching.Memory; namespace Example.Controllers {
[ApiController] [Route("[controller]")]
public class UserController : ControllerBase { private IMemoryCache \_memoryCache; public UserController(IMemoryCache memoryCache) { \_memoryCache = memoryCache; }

[HttpGet("{id}")] public IActionResult Get([FromRoute] int id) {
if (id == default) return NotFound();
var user = \_memoryCache.Get($"USER\_{id}");
if (user == null) return NotFound("User not found."); return Ok(user); }

[HttpPost] public IActionResult Post([FromBody] User user) {
if (user == null) return BadRequest();
var userKey = $"USER*{user.Id}";
if (!\_memoryCache.TryGetValue(userKey, out *)) { var options = new MemoryCacheEntryOptions { Priority = CacheItemPriority.Low }; \_memoryCache.Set(userKey, user, options); } return Ok(user); }

[HttpDelete("{id}")]
public IActionResult Delete([FromRoute] int id) { if (id == default) return NotFound(); \_memoryCache.Remove($"USER\_{id}"); return Ok(id); } } }
{% endhighlight %}

Evet artık user endpoint’leri üzerinden belleğe veri yazabiliyoruz. Gerçek dünya da bu işlemler veritabanı ile gerçekleştirilir. Uygulamamızın yapısına bağlı olarak süreç değişebilir. İlk isteği veritabanı üzerinden çekip dönebilir, daha sonra veriyi belleğe yazıp diğer isteklere bellekte ki veriler ile cevap verilebilir.

Öğrenilecek çok şey, gezilecek çok yer var. Bir sonraki postumda görüşmek üzere.
