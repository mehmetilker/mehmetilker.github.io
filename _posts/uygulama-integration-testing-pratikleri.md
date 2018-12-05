#Aternatif test yöntemleri

https://docs.microsoft.com/en-us/dotnet/standard/modern-web-apps-azure-architecture/test-asp-net-core-mvc-apps
Unit
Integration > are written from the perspective of the developer
	to verify that some components of the system work correctly together. 
Functional
	are written from the perspective of the user, and verify the correctness of the system based on its requirements.
UI tests, load tests, stress tests, and smoke tests



# Integration Test pratikleri

Unit test'lerin aksine Integration test'ler sıralı çalışmalı.

-Önce hesap oluşturma, giriş, çıkış, şifremi unuttum vs.  
-Moduller mantıklı bir sıraya göre çalışıp veri oluşturmalı.
	işe yaramadı: <http://hamidmosalla.com/2018/08/16/xunit-control-the-test-execution-order/>
	<https://github.com/xunit/samples.xunit/tree/master/TestOrderExamples/TestCaseOrdering>
	nuget olarak çalıştı. Sadece ITestCaseOrderer yetmiyor, collection olanı da uygulamalı (galiba) 
	<https://github.com/fulls1z3/xunit-orderer> > https://www.nuget.org/packages/XunitOrdererStandard/2.0.0 (NetStandard destekli nuget)
    <http://www.tomdupont.net/2016/04/how-to-order-xunit-tests-and-collections.html>	
-Oluşturulan veriler uygulama ana sayfasına gösterimi ile tutarlılığı kontrol edilmeli.

## Diğer ihtiyaç olabilecek şeyler

Test sonuçlarını için bilgi (ekran/dosya)
	<https://xunit.github.io/docs/capturing-output>  
Uygulamadaki Trace işlemleri (debug, info) test output ile birlikte alınabilmeli.
	<https://www.neovolve.com/2018/06/01/ilogger-for-xunit/>
    <https://github.com/trbenning/serilog-sinks-xunit>

Gruplama  
Bağlam paylaşımı (Context sharing) <https://xunit.github.io/docs/shared-context>
	Farklı testler arasında (Test Class) paylaşım yapılmak isteniyorsa: Collection Fixtures
		You can use the collection fixture feature of xUnit.net to share a single object instance among tests in several test class. 
Testler InMemory veritabanı ile çalıştırılmalı.
Dış servisler Mock'lanabilir.
Test case'lerini farklı verilerle çalıştırmak için
	<http://hamidmosalla.com/2017/02/25/xunit-theory-working-with-inlinedata-memberdata-classdata/>


Karşılaştırma
https://raygun.com/blog/unit-testing-frameworks-c/

XUnit <https://xunit.github.io/>

NUnit
http://nunit.org/documentation/
https://github.com/nunit/docs/wiki/NUnit-Documentation



Integration test örnekleri
https://holsson.wordpress.com/2018/06/25/integration-testing-asp-net-core-2-1/
https://medium.com/@satish1v/asp-net-core-integration-test-e70ab4315f2d
https://github.com/aspnet/Docs/blob/master/aspnetcore/test/integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs
https://github.com/aspnet/Docs/blob/master/aspnetcore/test/integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs



Improved end to end testing support for MVC applications
https://github.com/aspnet/Announcements/issues/275


https://github.com/JonPSmith/EfCore.TestSupport
unit test & EF Core

-WebApplicationFactory
Yaptığı şey test client üzerinde bir web host ayağa lakdırmakkaldırmak ve HttpContext i simule etme.
Login çalıştığı zaman SignIn olmuş oluyor ve sonraki isteklerde auth Cookie bilgisi taşındığı için kullanıcı oturum bilgisi her
istekde taşınmaya gerek kalmıyor.


#Testing üzerine notlar

-Unit test için o kadar zaman harcamak doğru mu?
Dumy veri hazırlamak için geçen zaman mekbul mu?
Bunun için planlamaya zaman ayrılmalı.
Planlama için çok zaman harcamak doğru mu?
En baştan isterler yanlışsa ya da yanlış anlaşıldıysa planlamaya harcaman zaman boş gitmiş olmuyor mu?
Kod stabil olduktan sonra unit testleri yazmak?