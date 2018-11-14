

Uygulama içinde trace logları atarken dikkat edilecekler.

Uygulama geliştirirken console'a yazılan loglar daha sonra canlı ortam da hata bulmak için de açık bırakılabilir.

Doğru ayar ve doğru araç kullanılmalı.

-Örneğin uygulama başlangıcında logger.info()
-Uygulama kapanırken logger.warning()
-Hata esnasında logger.Error()
-Geliştirme ortamında faydalı olan bazı işlemler için kullanılan logger.debug()

Canlı ortam için alt seviye information bırakılabilir. Böylece canlı ortam loglarında debug seviyesi görünmez.
Gerektiğinde config den açılarak canlı ortamda da debug logları alınabilir.

Tüm uygulama bazında debug seviyesini etkinleştirmemek için tanımlarda belirtilebilir.
XClass = DEBUG