## SIGNALR

Client-Server arasında klasik şekilde request-response türünde olan haberleşmenin tam aksine eş zamanlı olarak yapılan haberleşme yöntemidir.
Yerine göre klasik yöntem kullanılarak yapılan (request-response) işlemler hem zaman hemde performans açısından maliyetli olmaktadır.

İhtiyaç : Sayfaların dinamik olarak yenilenmesi.

Günümüz imkanlarını değerlendirecek olursak klasik haberleşme yaklaşımının pek yeterli olmadığı görülmektedir buna çözüm olarak farklı kütüphanelere ve hatta protokollere olan ihtiyaç kaçınılmazdır. Anlık veriler ile çalışılacak projelerde Real-Time hizmet verecek bir teknolojiye ihtiyaç olduğu ve HTTP'den farklı olarak TCP protokolünü benimseyen Web Socket alt yapılı sistemlerin kullanılması gerekliliği ortadadır.

TCP bağlantısı Client-Server arasında çift yönlü mesajlaşmayı sağlayan bir protokoldür(bi-directional messages). HTTP'de Client Server'i tetikler ama Server Client'ı tetikleyemez.

SignalR'ın altında yatan teknoloji Web Socket'tir. Özünde RPC (Remote Precedure Call) mekanizmasını benimsemektedir. RPC sayesinde, server, client'ta bulunan herhangi bir metodun tetiklenmesini sağlayabilmektedir. Böylece uygulamalar sayfa yenilemesi yapmadan veri transferini sağlamış olur.

SignalR 2011 yılında Microsoft tarafından geliştirilmiştir. 2013 yılında ise Asp.NET' e entegre edilmiştir(.Net içerisinde kullanmak için harici bir paket kurulumuna ihtiyaç yoktur Microsoft.AspNetCore.SignalR kütüphanesini doğrudan çağırabilirsiniz).

'Hub' adı verilen merkezi bir yapı üzerinden proje şekillenmektedir. Hub aslında bir class'tır ve içerisinde tanımlanan bir metoda abone olan/çağıran Client'lar ilgili hub üzerinden iletilen veriyi alacaktır.
Oluşturulan class isimlerinin 'Hub' ile bitmesi zorunluluğu yoktur fakat genel kullanım o şekildedir ve Hub sınıfından miras almak zorundadır.

```cs
 public class MyHub : Hub
    {
        public async Task SendMessageAsync(string message)
        {
            await Clients.All.SendAsync("receiveMessage", message);
        }
    }
```

Client ile Server arasında eş zamanlı bir etkileşim sağlanmaya çalışılırken ilk etapta bir bağlantı olmaması durumunda yada varolan bağlantı süreç içerisinde koptuysa böyle durumlarda bağlantının yeniden kurulması gerekmektedir.

Bağlantı sağlantıkdan sonra kopma durumu söz konusu olduğunda tanımlı olarak gelen withAutomaticReconnect fonksiyonunu kullanabiliriz, bu fonksiyonu .build() fonksiyonundan önce kullanmamız gereklidir.

```cs
   const connection = new signalR.HubConnectionBuilder()
          .withUrl("https://localhost:5001/myhub")
          .withAutomaticReconnect()
          .build();
```

Default olarak 4 periyotta kendisi otomatik olarak tekrar bağlanmayı deneyecektir. Bu periyotlar 0sn - 2sn - 10sn - 30sn'dir. Eğer bu 4 periyottada bağlantı sağlanamazsa tekrar bağlanmaya çalışmayacaktır.Bu periyodik sürelere müdahale edebilmemiz mümkün, fonksiyon içerisinde dizi olarak ve milisaniye cinsinden verdiğimiz değerlere göre tekrar bağlanmayı deneyecektir.

```cs
          .withAutomaticReconnect([1000,1000,5000,10000,30000])
          .build();
```

Bağlantının hiç kurulamadığı durumlarda tanımlı olarak gelen fonksiyon olmadığından bunu kendimiz yazabiliriz. Fonksiyon başlangıçta bağlantını kurulup kurulmadığını recursive şekilde deneyeceğinden dolayı, bu sürecin asenkron gerçekleşmesi gereklidir.

```cs
async function start() {
          try {
            await connection.start();
          } catch (error) {
            setTimeout(() => start(), 2000);
          }
        }
```

Bağlantı kurmaya çalışacak ve kurulamadığı durumda hata fırlatacağından catch bloğu çalışacaktır, buarada setTimeout fonksiyonu ile her 2 saniyede bir recursive olarak bağlantı denemesini ta ki bağlantı sağlanana kadar yapacaktır.

Client ile Server arasındaki bağlantının gerçekleşme durumuna göre fırlatılan eventlerden 3 tanesini ele alalım.

- onreconnecting() : Yeniden bağlanma girişimlerini başlatmadan önce çalıştırılan fonksiyon. Client bağlantısı koptuktan sonra yeniden bağlanma girişiminde bulunmadan önce çalışır. Öncelikle yapılacak işler varsa (temizlenmesi gereken cookie'ler vb.) yada kullanıcı bilgilendirilmesi yapılacaksa burada yapılır.
- onreconnected() : Yeniden bağlantı sağlandığında çalışır.
- onclose() : Yeniden bağlantı denemelerinin başarısız olduğu durumlarda çalışır. Bağlantılı başka servisler varsa gerekli bilgilendirmeler burada çalıştırılabilir.

Durum fonksiyonlarına connection üzerinden erişim sağlarız.

```cs
  connection.onreconnecting((error) => { } );
  connection.onreconnected((connectionId) => {
      // Client'ları biribirinden ayırmak için her bir bağlantıya bir onnectionId ataması yapılır, bize geriye bu connectionId'yi döndürecektir.
   } );
  connection.onclose((connectionId) => { } );
```
