## SIGNALR ##
Client-Server arasında klasik şekilde request-response türünde olan haberleşmenin tam aksine eş zamanlı olarak yapılan haberleşme yöntemidir.
Yerine göre klasik yöntem kullanılarak yapılan (request-response) işlemler hem zaman hemde performans açısından maliyetli olmaktadır.

İhtiyaç : Sayfaların dinamik olarak yenilenmesi.

Günümüz imkanlarını değerlendirecek olursak klasik haberleşme yaklaşımının pek yeterli olmadığı görülmektedir buna çözüm olarak farklı kütüphanelere ve hatta protokollere olan ihtiyaç kaçınılmazdır. Anlık veriler ile çalışılacak projelerde Real-Time hizmet verecek bir teknolojiye ihtiyaç olduğu ve HTTP'den farklı olarak  TCP protokolünü benimseyen Web Socket alt yapılı sistemlerin kullanılması gerekliliği ortadadır.

TCP bağlantısı Client-Server arasında çift yönlü mesajlaşmayı sağlayan bir protokoldür(bi-directional messages). HTTP'de Client Server'i tetikler ama Server Client'ı tetikleyemez.

SignalR'ın altında yatan teknoloji Web Socket'tir. Özünde RPC (Remote Precedure Call) mekanizmasını benimsemektedir. RPC sayesinde, server, client'ta bulunan herhangi bir metodun tetiklenmesini sağlayabilmektedir. Böylece uygulamalar sayfa yenilemesi yapmadan veri transferini sağlamış olur.

SignalR 2011 yılında Microsoft tarafından geliştirilmiştir. 2013 yılında ise Asp.NET' e entegre edilmiştir(.Net içerisinde kullanmak için harici bir paket kurulumuna ihtiyaç yoktur Microsoft.AspNetCore.SignalR kütüphanesini doğrudan çağırabilirsiniz).

'Hub' adı verilen merkezi bir yapı üzerinden proje şekillenmektedir. Hub aslında bir class'tır ve içerisinde tanımlanan bir metoda abone olan/çağıran Client'lar ilgili hub üzerinden iletilen veriyi alacaktır.
Oluşturulan class isimlerinin 'Hub' ile bitmesi zorunluluğu yoktur fakat genel kullanım o şekildedir ve Hub sınıfından miras almak zorundadır.
