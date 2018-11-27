- [Ön bilgi](#%C3%B6n-bilgi)
- [ETW'ye nasıl yazılır](#etwye-nas%C4%B1l-yaz%C4%B1l%C4%B1r)
- [Nasıl Trace yapılır](#nas%C4%B1l-trace-yap%C4%B1l%C4%B1r)
      - [Tracing alternatifleri](#tracing-alternatifleri)
            - [logman ile session oluşturmak](#logman-ile-session-olu%C5%9Fturmak)
            - [Kod ile süreç](#kod-ile-s%C3%BCre%C3%A7)
            - [Uygulamada app.config ile tracing](#uygulamada-appconfig-ile-tracing)
            - [Github üzerinde ETW proje bilgileri](#github-%C3%BCzerinde-etw-proje-bilgileri)
      - [Bazı Provider'lar için kaynaklar](#baz%C4%B1-providerlar-i%C3%A7in-kaynaklar)
            - [Data / Ado.net trace için](#data--adonet-trace-i%C3%A7in)
            - [CLR Events](#clr-events)
            - [IIS log events](#iis-log-events)
                  - [IIS için alternatif tracing yöntemler](#iis-i%C3%A7in-alternatif-tracing-y%C3%B6ntemler)
                  - [Web / Network işlemleri için etkinleştirilebilecek provider'lar](#web--network-i%C5%9Flemleri-i%C3%A7in-etkinle%C5%9Ftirilebilecek-providerlar)
            - [Uygulama takibi](#uygulama-takibi)
            - [Sistem takibi](#sistem-takibi)
      - [Event yöntemi için Reactive kullanımı](#event-y%C3%B6ntemi-i%C3%A7in-reactive-kullan%C4%B1m%C4%B1)
- [ETW okumanın çalışmadığı durumlarda](#etw-okuman%C4%B1n-%C3%A7al%C4%B1%C5%9Fmad%C4%B1%C4%9F%C4%B1-durumlarda)
- [Diğer bilgiler](#di%C4%9Fer-bilgiler)


# Ön bilgi

Bir uygulada log tutuluyorsa ve ve bu loglar ETW'ye yönlendirilebiliyorsa, aşağıdaki kütüphaneler ile ETW'ye subscribe olup, Provider+Event filtreleme yöntemiyle bu event'ler toplanabilir.

Örnek: IIS logları, bir uygulamadaki System.Net namespace'i altındaki operasyonlar (HttpWebRequest), Ado.Net, .NET CLR Events

>Bir uygulamadan ETW'ye log yazmak için: EventSource
https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.EventSource/

>Okumak için: TraceEvent
https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent

ETW Kavram detayları
<https://docs.microsoft.com/en-us/message-analyzer/common-provider-configuration-settings-summary>
<https://docs.microsoft.com/en-us/message-analyzer/system-etw-provider-event-keyword-level-settings>

Consume ETW events
<https://blogs.msdn.microsoft.com/dotnet/2013/08/15/announcing-traceevent-monitoring-and-diagnostics-for-the-cloud/>
<https://blogs.msdn.microsoft.com/vancem/2013/08/15/traceevent-etw-library-published-as-a-nuget-package/>
https://blogs.msdn.microsoft.com/vancem/2014/03/15/walk-through-getting-started-with-etw-traceevent-nuget-samples-package/
EtwTraceEventSource which lets you read the stream of ETW events
https://blogs.msdn.microsoft.com/vancem/2012/12/20/using-tracesource-to-log-etw-data-to-a-file/

Bir de
https://stackoverflow.com/questions/21012622/get-windows-event-provider-information
System.Diagnostics.Eventing.Reader sınıfı var. Yapılabileceker:
Find out the names of ETW providers installed on a computer
Discover a complete list of ETW log names present on a computer
Enumerate metadata related to ETW providers
Export event log data

# ETW'ye nasıl yazılır

EventSource kayıt açmak için
https://blogs.msdn.microsoft.com/vancem/2014/03/15/walk-through-getting-started-with-etw-traceevent-nuget-samples-package/
In a previous post, I talked about the TraceEvent NuGet Library, which allows you to read and manipulate Event Tracing for Windws (ETW).
There is a companion post about the EventSource  NuGet package which allows you to create your own ETW events (or in fact to send those events to anywhere you choose).



# Nasıl Trace yapılır

Trace işlemi için önce bir session ve bu session üzerinde istenen Event Provider'lar aktfi edilmeli.
Proviver listesi:

```CMD
logman query providers > plist.txt
```

Session > Realtime, dosya (.etl) veya her ikisi de olabilir.

Provider ayarları: Provider detaylarında Level ve diğer seçenekler belirtilir. (Keywords...)
Session ne kadar süre aktif kalacak ve Windows Start up da otomatik başlatılacak mı gibi tanımlar da yapılabilir.

## Tracing alternatifleri

> CMD logman, xperf, sysinternals tools
> APP Performance Monitor > Data Collector Sets > Event Trace Session > [New Data Collector Set] + Providers  
> APP PerfView > Collect  
> CODE Microsoft.Diagnostics.Tracing.TraceEvent library


### logman ile session oluşturmak

session adı ve aktif edilmesi istenen provider belirtilir.
Event'ler oluşturulduktan sonra stop.
Olıuşan .etl dosyasını okuyabilmek için .csv ya da .xml çevrimi.
Event Viewer ile de olkunabilir.

```CMD
WinInet ile sadece internet explorer/edge isteklerini takip
logman start "wininettrace"  -p "microsoft-windows-wininet" -o "wininettrace.etl" -ets
--event'leri bekle
logman stop "wininettrace" -ets
tracerpt "wininettrace.etl" -y -o "wininetracelog.xml" -of xml
```

```CMD
Http.sys is the kernel-mode HTTP listener
The 0xFFFF signifies we want full tracing, or all trace events.
logman start httptrace -p Microsoft-Windows-HttpService 0xFFFF -o httptrace.etl -ets
logman stop httptrace -ets
tracerpt.exe httptrace.etl –of CSV -o httptrace.csv / tracerpt.exe httptrace.etl -of XML -o httptrace.xml
```

```CMD
Application app.config - diagnostic System.Net için tanım ile (initializeData=BDE...)
logman start "xappsession" -p "{BDE5930E-34C9-4E2F-A6EC-89E1F1EA69CC}" -o "xappsystemnet.etl" -ets
--Made some request from the application to trigger some HttpWebRequest operation.
logman stop "xappsession" -ets
tracerpt xappsystemnet.etl -of csv -o xappsystemnet.csv // tracerpt "xappsystemnet.etl" -y -o "xappsystemnet.xml" -of xml
```

Örn: üstteki 3. örnekteki gibi bir izleme .etl olarak keydedilip xml'e çevrildiğinde alttaki gibi kayıtlar oluşur.
      -app/web.config için Shared listener örneği aşağıda.

```XML
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
	<System>
		<Provider Guid="{bde5930e-34c9-4e2f-a6ec-89e1f1ea69cc}" />
		<EventID>0</EventID>
		<Version>0</Version>
		<Level>8</Level>
		<Task>0</Task>
		<Opcode>0</Opcode>
		<Keywords>0x0</Keywords>
		<TimeCreated SystemTime="2018-11-12T16:12:12.586128800+02:59" />
		<Correlation ActivityID="{00000000-0000-0000-0000-000000000000}" />
		<Execution ProcessID="14596" ThreadID="21060" ProcessorID="0" KernelTime="0" UserTime="15" />
		<Channel />
		<Computer />
	</System>
	<Data>[21060] HttpWebRequest#22379747 - Request: GET /2.2/answers?order=desc&amp;sort=activity&amp;site=stackoverflow HTTP/1.1
      </Data>
</Event>
```

logman kullanımı  
https://blogs.technet.microsoft.com/askperf/2008/05/13/two-minute-drill-logman-exe/
https://blogs.msdn.microsoft.com/sudeepg/2009/02/26/capturing-and-analyzing-an-etw-trace-event-tracing-for-windows/  
Powershell den (Admin elev.) event listeleme
Get-WinEvent Microsoft-IIS-Logging/Logs
tüm event log listesi
Get-WinEvent -ListLog * | Format-List -Property LogName  
https://letitknow.wordpress.com/2012/09/02/powershell-and-the-applications-and-services-logs/

Benzer araç xperf (Windows Performance Analyzer (GUI)):  

```CMD
xperf.exe -start <SomeName> -on <NameOfYourRegisteredProvider>
xperf.exe -start FirstETW -on Function-Entry-Exit-Provider
```

Sysinternals tools (dbgview.exe)  
https://www.codeproject.com/articles/802379/collect-net-applications-traces-with-sysinternals

### Kod ile süreç

TraceEventSession ile session oluşturulur.
ETWTraceEventSource ile session'da oluşturulan bilgiler okunur.
Özet:
https://blogs.msdn.microsoft.com/dotnet/2013/08/15/announcing-traceevent-monitoring-and-diagnostics-for-the-cloud/

Provider aktifleştirmek:
session.EnableProvider("Microsoft-Windows-IIS-Logging", TraceEventLevel.Informational, 0x10);

>Bir uygulada birden fazla session başlatabilmek için

```C#
Task.Factory.StartNew(delegate
{
      _traceSource.Process()
});
```

### Uygulamada app.config ile tracing

System.Net namespace kullanan uygulamadan (HttpWebRequest) giden istekleri trace etmek için:
web.config e diagnostic element i eklenir.
Event lerin yazılabileceği alternatifler
text dosya, xml dosya, console, ETW, EventLog

https://docs.microsoft.com/en-us/dotnet/framework/network-programming/interpreting-network-tracing
App.config e eklenmesi gereken:


ETW Tracing ve diğerleri için SharedListeners örneği:

```XML
--Diagnostics node u için tam örnek:
--https://docs.microsoft.com/en-us/dotnet/framework/network-programming/how-to-configure-network-tracing
<sharedListeners>
      <add name="MyConsole" type="System.Diagnostics.ConsoleTraceListener"/>
      <add name="MyTraceFile" type="System.Diagnostics.TextWriterTraceListener" initializeData="System.Net.trace.log" traceOutputOptions="DateTime, ProcessId, ThreadId" />
      <!--traceOutputOptions="Timestamp" traceOutputOptions="LogicalOperationStack, DateTime, Timestamp, Callstack"-->
      <add name="ETWListener" initializeData="{BDE5930E-34C9-4E2F-A6EC-89E1F1EA69CC}"
           type="System.Diagnostics.Eventing.EventProviderTraceListener, System.Core, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
      <add name="MyEventLog" initializeData="XAppTraceListenerLog" type="System.Diagnostics.EventLogTraceListener" />
    </sharedListeners>
```

### Github üzerinde ETW proje bilgileri

<https://github.com/Microsoft/perfview/blob/master/documentation/TraceEvent/TraceEventLibrary.md>
The Microsoft.Diagnostics.Tracing.TraceEvent Library
<https://github.com/Microsoft/Microsoft.Diagnostics.Tracing.Logging>
Logging using ETW and EventSource

## Bazı Provider'lar için kaynaklar

### Data / Ado.net trace için

https://lowleveldesign.org/2012/09/07/diagnosing-ado-net-with-etw-traces/
https://msdn.microsoft.com/en-us/library/ms971550.aspx?f=255&MSPPError=-2147217396
https://www.developer.com/net/csharp/article.php/10918_3723011_2/ADONET-Trace-Logging.htm
https://www.codeguru.com/csharp/.net/net_debugging/logging/article.php/c14769/ADONET-Trace-Logging.htm

### CLR Events

https://docs.microsoft.com/en-us/dotnet/framework/performance/etw-events-in-the-common-language-runtime
https://docs.microsoft.com/en-us/dotnet/framework/performance/clr-etw-providers
https://docs.microsoft.com/en-us/dotnet/framework/performance/clr-etw-events

CLR Event'ler için Performance Counter yerine ETW'yi kullanmayı öneriyor.
http://labs.criteo.com/2018/06/replace-net-performance-counters-by-clr-event-tracing/
http://labs.criteo.com/2018/07/grab-etw-session-providers-and-events/

```C#
--Bazı provider'lar için örnek anahtar tanımları
session.EnableKernelProvider(Microsoft.Diagnostics.Tracing.Parsers.KernelTraceEventParser.Keywords.DiskIO);
session.EnableProvider(Microsoft.Diagnostics.Tracing.Parsers.AspNet.AspNetTraceEventParser.Keywords.Page);session.EnableProvider(Microsoft.Diagnostics.Tracing.Parsers.IisTraceEventParser.Keywords.IISGeneral);
session.EnableProvider(Microsoft.Diagnostics.Tracing.Parsers.IisTraceEventParser.IISGeneralGuid);

// We also turn on CLR events because we need them to decode Stacks and we also get exception events (and their stacks)
ClrTraceEventParser.ProviderGuid, TraceEventLevel.Error, (ulong)ClrTraceEventParser.Keywords.Exception

```

### IIS log events

IIS Log için ETW etkin duruma getirildikten sonra  (Site settings&gt; Logging)
IIS için Log seçeneklerinden ETW aktif edilmelidir. (File + ETW)

Event Viewer'da takip için&gt;  Applications and services logs &gt; Microsoft &gt; Windows &gt; IIS Logging &gt; Logs : Sağ tuş Enable yapmak gerekiyor.

Ya da uygulama ile session oluşturup "Microsoft-Windows-IIS-Logging" provider'i ile loglar takip edilebilir.

-Uygulama içeriğinden (server side code) bağımsız takip edilebilecek 3 şey:

1. Http Logs
2. Failed Trace Logs
3. Http Err Logs

https://docs.microsoft.com/en-us/windows/desktop/http/types-of-errors-logged-by-the-http-server-api  
HTTPERR: Types of Errors Logged by the HTTP Server API


ETW de bir web requesti için Provider a göre farklı event ler üretiliyor.  
Http-log için örneğin tek bir event.  
asp.net / asp için her bir app event a karşılık bir ETW event.  
Begin_Request     
Error 
...   
End_Request       
ContextId="8000007d-0000-fe00-b63f-84710c7967bb" ile Applicaton Request Event'ler eşleştirilebilir.  
Örnek uygulama:  
https://github.com/tomasr/frebrilator

https://github.com/tomasr/iis-etw-tracing
https://github.com/Microsoft/perfview/blob/master/documentation/TraceEvent/TraceEventProgrammersGuide.md
https://github.com/Microsoft/perfview/blob/master/documentation/TraceEvent/TraceEventLibrary.md
!!!
https://github.com/Microsoft/dotnet-samples/blob/master/Microsoft.Diagnostics.Tracing/TraceEvent/docs/TraceEvent.md
Genel kavramlar anlatılıyor:
Saniyede 10 bin event den fazlası kaynak sıkıntısı yaratıyor.
Event yayını buffer'lı çalıştığı için geçikme olabiliyor (3 saniye kadar)

https://github.com/neuecc/EtwStream

Samples     
https://github.com/Microsoft/perfview/tree/master/src/TraceEvent/Samples

?
https://github.com/Azure/diagnostics-eventflow  
IISLog parse

https://github.com/alexnolasco/32120528/
https://github.com/Microsoft/Tx (ETW üzerinden)

Power shell ile performance counter okuma       
https://hodgkins.io/using-powershell-to-send-metrics-graphite

İstek akışı anlatımı    
IIS Request-Based Tracing     
https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc786920(v%3dws.10)


Using Failed Request Tracing to Trace Rewrite Rules   
UrlRewrite trace event detayları adım adım açıklanmış  
https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-failed-request-tracing-to-trace-rewrite-rules
How to display URL Rewrite ETW Events in the Event Viewer
https://blogs.msdn.microsoft.com/danielvl/2009/01/25/how-to-display-url-rewrite-etw-events-in-the-event-viewer/
http://blogs.msdn.com/danielvl/archive/2009/01/25/view-etw-rewrite-events.aspx

#### IIS için alternatif tracing yöntemler

FREB, appcmd

>Failed Request Tracing - cmd  
 %windir%\system32\inetsrv\appcmd configure trace "Default Web Site" /enablesite    
 %windir%\system32\inetsrv\appcmd configure trace "Default Web Site" /enable /path:test.aspx /timeTaken:00:00:30  
 appcmd list traces | findstr "test.aspx"       
 http://mvolo.com/troubleshoot-iis-hanging-requests/


>appcmd list ile
IIS Currently executing request
 elevated command line
 %windir%\system32\inetsrv\appcmd list requests 
 %windir%\system32\inetsrv\appcmd list requests /elapsed:5000
 In a loop (assuming you are in %windir%\system32\inetsrv\
 for /l %x in (,,) do (appcmd list requests /elapsed:5000 &amp; timeout 2)
 https://docs.microsoft.com/en-us/iis/troubleshoot/using-failed-request-tracing/troubleshoot-with-failed-request-tracing




#### Web / Network işlemleri için etkinleştirilebilecek provider'lar

```C#
"Microsoft-Windows-IIS-Logging"
//IIS w3c log > 1 log per req/response
"IIS: Active Server Pages (ASP)", TraceEventLevel.Verbose
//Page events > AspReq/ASP_NEW_SESSION_CREATED, AspReq/ASP_END_REQUEST etc... (not all of them)

//Logs All request steps: GENERAL_REQUEST_END, GENERAL_RESPONSE_HEADERS etc. (htm request count 33 event, asp request 19 count)
"Microsoft-Windows-IIS" 
//ASP.NET
"ASP.NET Events" //AspNetReq/Start ...
//Microsoft-Windows-ASPNET/Request/Start

"IIS: WWW Isapi Extension" //W3Isapi/CALL_ISAPI_EXTENSION
"IIS: WWW Server" //> acinca ASP provider çalışmıyor.
"IIS: WWW Global" //> acinca ASP provider çalışmıyor.

"Microsoft-Windows-IIS-APPHOSTSVC" //Application Host Helper Service                
"Microsoft-Windows-IIS-CentralCertificateProvider"                
"Microsoft-Windows-IIS-Configuration" //read/map web.config etc....                
"Microsoft-Windows-IIS-IisMetabaseAudit"                
"Microsoft-Windows-IIS-IISReset"
"Microsoft-Windows-IIS-W3SVC" //World wide web publishing service
"Microsoft-Windows-IIS-W3SVC-PerfCounters"
"Microsoft-Windows-IIS-W3SVC-WP" //>Worker process ??
"Microsoft-Windows-IIS-WMSVC" //> Web management service

//spotify i kadydetti.
//This event is logged when the WinHTTP Webproxy Auto-Discovery service detected an expected exception.
"Microsoft-Windows-WinHttp"

"Microsoft-Windows-HttpLog"
//?? gelen istekler iin
"Microsoft-Windows-HttpEvent"                

//smartscreen adında bir process in istekleri
"Microsoft-Windows-Runtime-Web-Http"
"Microsoft-Windows-Runtime-WebApi"

//Http.sys is the kernel-mode HTTP listener / HttpErr log
//Yerel sunucuya (IIS) gelen istekler (events: RecvReq, SendComplete...)
//https://blogs.msdn.microsoft.com/wndp/2007/01/18/event-tracing-in-http-sys-part-1-capturing-a-trace/
//https://blogs.msdn.microsoft.com/wndp/2007/02/01/event-tracing-in-http-sys-part-3-typical-request/
//https://blogs.msdn.microsoft.com/wndp/2007/01/25/event-tracing-in-http-sys-part-2-anatomy-of-an-event/                
//https://docs.microsoft.com/en-us/windows/desktop/http/scenario-1--http-timeout-example-using-etw-tracing-and-netsh-commands
"Microsoft-Windows-HttpService" 

//Yerel service, app istekleri (opera/edge vs görünmüyor.)
//WinHTTP
"Microsoft-Windows-WebIO"

//client side istekler
//winnet.dll > https://blogs.technet.microsoft.com/askperf/2007/08/21/under-the-hood-wininet/
//Event steps > https://blogs.msdn.microsoft.com/wndp/2007/02/05/wininet-etw-logs-part-1-reading-logs/
"Microsoft-Windows-WinINet" // IE/Edge requests..

"Microsoft-Windows-Winsock"
"Microsoft-Windows-DNS" //DNS Resolve operation

```


### Uygulama takibi

Uygulama açılış kapanışlarını takip etmek için
https://blogs.msdn.microsoft.com/vancem/2013/03/09/using-traceevent-to-mine-information-in-os-registered-etw-providers/
"EventLog-Application"
"Microsoft-Windows-Kernel-Process" provider kullanıyor.
session.EnableProvider("Microsoft-Windows-Kernel-Process", TraceEventLevel.Informational, 0x10);
https://randomascii.wordpress.com/2016/03/08/power-wastage-on-an-idle-laptop/
https://randomascii.wordpress.com/2015/09/01/xperf-basics-recording-a-trace-the-ultimate-easy-way/

### Sistem takibi

Thermal management - fan sıcaklığı vs.
Kod örneği yok.
https://docs.microsoft.com/en-us/windows-hardware/design/device-experiences/thermal-management-in-windows
örnek uygulama
Open hardware monitor
https://openhardwaremonitor.org/screenshots/

Bandwith tracing için
<https://social.msdn.microsoft.com/Forums/en-US/aa9b1fed-b823-4a9c-973d-1dc0554f994c/understanding-etw?forum=csharpgeneral>


## Event yöntemi için Reactive kullanımı

Bir işlem bir event için gerekli değil ama bir işlem için birden fazla event yayınlanan durumlarda (Farklı provider'lardan birbiriyle  ilişkili Start ... Stop) event'leri gruplayıp (merge işlemi) üzerinde çalışmak gerektiğinde Observable sınıfı kullanılabilir.

ETW-Reactive
https://github.com/tomasr/iis-etw-tracing
https://github.com/tomasr/frebrilator/blob/master/Frebrilator/EventAggregator.cs
https://github.com/neuecc/EtwStream
https://github.com/Microsoft/perfview/tree/master/src/TraceEvent/Samples

Reactive  
(https://www.tonytruong.net/starting-reactive-extensions-with-existing-events-in-c/)
https://blogs.endjin.com/2014/04/event-stream-manipulation-using-rx-part-1/  
https://blogs.endjin.com/2014/05/event-stream-manipulation-using-rx-part-2/  
http://reactivex.io/tutorials.html  
http://reactivex.io/documentation/operators.html#combining
https://github.com/Microsoft/Tx/blob/master/Doc/Readme.md
https://github.com/Microsoft/Tx/blob/master/Samples/LinqPad/Queries/IE_IIS/Readme.md
https://github.com/Microsoft/Tx

# ETW okumanın çalışmadığı durumlarda

Örneğin System.Net HttpWebRequest için (config'de initializeData ile provider Guid belirtip) logman ile event'lar .etl'e kaydedilebiliyor ama kod ile okunamıyor.

Böyle durumlar için alternatif app.config'de diagnostic nodunda EventLog'a yazılır ve kod ile buradan okunabilir.  
Bunun için:       
EventLogWatcher class   
https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.eventing.reader.eventlogwatcher?redirectedfrom=MSDN&view=netframework-4.7.2      
https://msdn.microsoft.com/library/62e006d3-9fab-4fdf-a8f8-e23d05498cd4?f=255&MSPPError=-2147217396   
How to: Subscribe to Events in an Event Log




# Diğer bilgiler

Interpreting Network Tracing  
https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/trace-listeners


Event'leri ilişkilendirmek için ActivityId kullanımı  
https://blogs.msdn.microsoft.com/vancem/2015/09/14/exploring-eventsource-activity-correlation-and-causation-features/
https://blogs.msdn.microsoft.com/vancem/2015/09/15/eventsource-activity-support-demo-code/

ETW Diğer notlar

Araç:

http://lallouslab.net/2016/01/25/windows-events-providers-explorer/

Windows Events Providers Explorer

Sistemde hangi Event Provider'lar var ve hangi event'ler sağlanıyor bilgisi.

UIforETW

https://randomascii.wordpress.com/2015/04/14/uiforetw-windows-performance-made-easier/

https://github.com/google/UIforETW

https://randomascii.wordpress.com/2015/09/24/etw-central/


Application Analysis with Event Tracing for Windows (ETW)
xperf üzerinden ETW anlatımı
https://www.codeproject.com/articles/570690/application-analysis-with-event-tracing-for-window
Masaüstü uygulaması
https://www.codeproject.com/Articles/632390/EtwDataViewer-Analyze-visualize-and-make-sense-of

muhtemel eski yöntem
https://blogs.msdn.microsoft.com/dmetzgar/2013/01/24/capturing-a-profile-in-production-based-on-etw-events/
nuget: Microsoft.Samples.Eventing library
class: EventTraceWatcher