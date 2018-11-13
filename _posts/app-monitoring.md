İzleme çözümü 3 adımdan oluşur.



Event okuma

Event kaydetme

Görüntüleme



# Monitoring  - Veri kaydetme + Görüntüleme çözümleri
Alternatifler

https://al-hardy.blog/2017/04/08/open-source-apm/



## Bütünleşik hazır çözüm olarak ELK:

ELK - Elastic stack- search, kibana, beats, logstash

https://www.elastic.co/guide/index.html

https://www.elastic.co/guide/en/elastic-stack-overview/6.3/get-started-elastic-stack.html

https://logz.io/learn/complete-guide-elk-stack/#installing-elk

ELK docker

ES + Kibana + Logstash

http://elk-docker.readthedocs.io/


Veya 3 katman için ayrı çözüm geliştirme.

Veri görselleştirme/okuma aracı veritabanını tanıyor olmalı.



Veri tabanına (Time series DB) yazma probleminde veriyi gönderen tekrar yazabilir duruma gelene kadar yerel ortamda kaydetmeli. Bunun için ETL file kullanılabilir. (Realtime session + ETL file).

Belirli aralıklarla bu dosya temizlenmeli.

Event Okuma
IIS için Log seçeneklerinden ETW aktif edilmelidir. (File + ETW)





Performance Monitor de önce session tanımlanır (Event Trace Sessions) ya da mevcut session tanımına Provider eklenir.

Provider ayarları: Provider detaylarında Level ve diğer seçenekler belirtilir.



Session ayarları: Real time ve/veya File seçimi yapılır.

Session ne kadar süre aktif kalacak ve Windows Start up da otomatik başlatılacak mı gibi tanımlar da yapılabilir.



Log takibi için:

1. .etl dosyaları "Event Viewer" ile açılır. Yada LogParser kullanılarak .csv formatına çevrilir.

2. Real time loglar session adı belirtilerek ETWTraceEventSource class'ı (Microsoft.Diagnostics.Tracing.TraceEvent) ile takip edilir.

Eğer istenilen Provider (örn: "Microsoft-Windows-IIS-Logging") tanımlı bir session'da ekli değilse.

var session = new TraceEventSession(sessionName, null) class kullanılarak session oluşturulabilir ve

session.EnableProvider("Microsoft-Windows-IIS-Logging", TraceEventLevel.Informational, 0x10);

ile istenen provider o session'da aktif edilir.



IIS Log için ETW etkin duruma getirildikten sonra  (Site settings&gt; Logging)

Event Viewer da takip için&gt;  Applications and services logs &gt; Microsoft &gt; Windows &gt; IIS Logging &gt; Logs : Sağ tuş Enable demek gerekiyor.

Ya da uygulama ile session oluşturup "Microsoft-Windows-IIS-Logging" provider'i ile loglar takip edilebilir.



-Takip edilecek 3 şey

Http Logs

Failed Trace Logs

Http Err Logs





ETW de bir web requesti için Provider a göre farklı event ler üretiliyor.

Http-log için örneğin tek bir event.

asp.net / asp için her bir app event a karşılık bir ETW event.

Begin_Request

Error

...

End_Request

ContextId="8000007d-0000-fe00-b63f-84710c7967bb" ile Applicaton Request Event'ler eşleştirilebilir.

Örnek uygulama: https://github.com/tomasr/frebrilator





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





Diğer Event okuma alternatifleri

File beats (java tabanlı)

https://www.elastic.co/blog/monitoring-windows-logons-with-winlogbeat

Filebeat iis module

https://www.elastic.co/guide/en/beats/filebeat/master/filebeat-module-iis.html







# Event Kaydetme - Okuma


Alternatiflerden biri: Elastic Search



Docker

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

config/elasticsearch.yml

RUN echo "http.cors.enabled: true" &gt;&gt; /usr/share/elasticsearch/config/elasticsearch.yml

RUN echo "http.cors.allow-origin: '*'" &gt;&gt; /usr/share/elasticsearch/config/elasticsearch.yml



docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2





.net api

https://github.com/elastic/elasticsearch-net





ES için date / timestamp formatı

https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/date-processor.html

ya da index oluştur ve format belirt:

https://discuss.elastic.co/t/how-to-configure-index-pattern-to-use-custom-timestamp-in-kibana/66505

https://support.logz.io/hc/en-us/articles/210206285-How-can-I-make-sure-the-timestamp-is-properly-parsed-





--System.Net namespace kullanan uygulamadan (HttpWebRequest) giden istekleri trace etmek için

web.config e diagnostic element i eklenir.

Event lerin yazılacağı alternatifler

text dosya, xml dosya, console, ETW, EventLog

https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/trace-listeners

Interpreting Network Tracing

https://docs.microsoft.com/en-us/dotnet/framework/network-programming/interpreting-network-tracing

App.config e eklenmesi gereken:

https://docs.microsoft.com/en-us/dotnet/framework/network-programming/how-to-configure-network-tracing

ETW Tracing ve diğerleri için SharedListeners örneği:

```xml
<sharedListeners>

      <add name="MyConsole" type="System.Diagnostics.ConsoleTraceListener"/>

      <add name="MyTraceFile" type="System.Diagnostics.TextWriterTraceListener" initializeData="System.Net.trace.log" traceOutputOptions="DateTime, ProcessId, ThreadId" />

      <!--traceOutputOptions="Timestamp" traceOutputOptions="LogicalOperationStack, DateTime, Timestamp, Callstack"-->

      <add name="ETWListener" initializeData="{BDE5930E-34C9-4E2F-A6EC-89E1F1EA69CC}"

           type="System.Diagnostics.Eventing.EventProviderTraceListener, System.Core, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />

      <add name="MyEventLog" initializeData="XAppTraceListenerLog" type="System.Diagnostics.EventLogTraceListener" />

    </sharedListeners>
```

### Data / Ado.net trace için

https://lowleveldesign.org/2012/09/07/diagnosing-ado-net-with-etw-traces/

https://msdn.microsoft.com/en-us/library/ms971550.aspx?f=255&MSPPError=-2147217396		

https://www.developer.com/net/csharp/article.php/10918_3723011_2/ADONET-Trace-Logging.htm

https://www.codeguru.com/csharp/.net/net_debugging/logging/article.php/c14769/ADONET-Trace-Logging.htm


CLR Events

https://docs.microsoft.com/en-us/dotnet/framework/performance/etw-events-in-the-common-language-runtime

https://docs.microsoft.com/en-us/dotnet/framework/performance/clr-etw-providers

https://docs.microsoft.com/en-us/dotnet/framework/performance/clr-etw-events



CLR Event'ler için Performance Counter yerine ETW'yi kullanmayı öneriyor.

http://labs.criteo.com/2018/06/replace-net-performance-counters-by-clr-event-tracing/

http://labs.criteo.com/2018/07/grab-etw-session-providers-and-events/


### Sorgulama 
Query elasticsearch

https://logz.io/blog/elasticsearch-queries/

https://www.compose.com/articles/using-query-string-queries-in-elasticsearch/

https://www.timroes.de/2016/05/29/elasticsearch-kibana-queries-in-depth-tutorial/

https://dzone.com/articles/23-useful-elasticsearch-example-queries



Query doc

https://logz.io/blog/elasticsearch-queries/


http://localhost:9200/_search?pretty

http://localhost:9200/winlogbeat-*/_search

http://localhost:9200/_cat/indices?v

http://localhost:9200/iislog5/_search?q=Status:404



# Event Görüntüleme


Kibana yada Grafana



Kibana

https://demo.elastic.co/

https://www.timroes.de/2015/02/07/kibana-4-tutorial-part-1-introduction/

https://www.elastic.co/guide/en/kibana/6.3/dashboard.html



http://docs.grafana.org/installation/docker/

http://docs.grafana.org/features/datasources/elasticsearch/





Front end

https://github.com/lmenezes/cerebro

https://github.com/mobz/elasticsearch-head


https://medium.com/@dionnis/elasticsearch-monitoring-and-maintenance-tools-research-18c5fb45a747





Grafana on Elasticsearch

https://grafana.com/blog/2016/03/09/how-to-effectively-use-the-elasticsearch-data-source-in-grafana-and-solutions-to-common-pitfalls/



Diğer Notlar


Docker ile çözüm geliştirildiğinde:

Docker ile iki ayrı container da (biri ES biri Grafana) olunca birbirlerini görmüyorlar

compose da network alanı ile bu sağlanır.





ETW Diğer notlar
ETW Kavram detayları

https://docs.microsoft.com/en-us/message-analyzer/common-provider-configuration-settings-summary

https://docs.microsoft.com/en-us/message-analyzer/system-etw-provider-event-keyword-level-settings





Araç:

http://lallouslab.net/2016/01/25/windows-events-providers-explorer/

Windows Events Providers Explorer

Sistemde hangi Event Provider'lar var ve hangi event'ler sağlanıyor bilgisi.





logman kullanımı

https://blogs.technet.microsoft.com/askperf/2008/05/13/two-minute-drill-logman-exe/

https://blogs.msdn.microsoft.com/sudeepg/2009/02/26/capturing-and-analyzing-an-etw-trace-event-tracing-for-windows/



Powershell den (Admin elev.) event listeleme

Get-WinEvent Microsoft-IIS-Logging/Logs

tüm event log listesi

Get-WinEvent -ListLog * | Format-List -Property LogName

https://letitknow.wordpress.com/2012/09/02/powershell-and-the-applications-and-services-logs/



appcmd list ile

IIS Currently executing request

 elevated command line

 %windir%\system32\inetsrv\appcmd list requests 

 %windir%\system32\inetsrv\appcmd list requests /elapsed:5000

 In a loop (assuming you are in %windir%\system32\inetsrv\

 for /l %x in (,,) do (appcmd list requests /elapsed:5000 &amp; timeout 2)

 https://docs.microsoft.com/en-us/iis/troubleshoot/using-failed-request-tracing/troubleshoot-with-failed-request-tracing

Failed Request Tracing - cmd 

 %windir%\system32\inetsrv\appcmd configure trace "Default Web Site" /enablesite 

 %windir%\system32\inetsrv\appcmd configure trace "Default Web Site" /enable /path:test.aspx /timeTaken:00:00:30

 appcmd list traces | findstr "test.aspx"

 http://mvolo.com/troubleshoot-iis-hanging-requests/



UIforETW

https://randomascii.wordpress.com/2015/04/14/uiforetw-windows-performance-made-easier/

https://github.com/google/UIforETW

https://randomascii.wordpress.com/2015/09/24/etw-central/







Consume ETW events
https://blogs.msdn.microsoft.com/dotnet/2013/08/15/announcing-traceevent-monitoring-and-diagnostics-for-the-cloud/

https://blogs.msdn.microsoft.com/vancem/2013/08/15/traceevent-etw-library-published-as-a-nuget-package/

https://blogs.msdn.microsoft.com/vancem/2014/03/15/walk-through-getting-started-with-etw-traceevent-nuget-samples-package/

https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent/

EtwTraceEventSource which lets you read the stream of ETW events





How to Setup Event Trace Session

&gt;Performance Monitor &gt; Event Trace Session

http://blogs.msdn.com/danielvl/archive/2009/01/25/view-etw-rewrite-events.aspx

https://blogs.msdn.microsoft.com/danielvl/2009/02/02/how-to-consume-etw-events-from-c/





https://blogs.iis.net/richma/winhttp-tracing-options-for-troubleshooting-with-application-request-routing

Winhttp Tracing Options for Troubleshooting with Application Request Routing

netsh trace start scenario=InternetClient capture=yes report=yes





https://blogs.msdn.microsoft.com/vancem/2012/12/20/an-end-to-end-etw-tracing-example-eventsource-and-traceevent/

This is what a 'TraceEventParser' is: something that knows how to parse a PARTICULAR provider's events.   

We will see in later blog entries how we can create TraceEventParsers that are designed to decode just ONE EventSource's events, 

however in this case we are using the 'DynamicTraceEventParser' which knows how to decode ANY EventSource (but can only do so relatively inefficiently and is relatively cumbersome to code against). 





https://blogs.msdn.microsoft.com/vancem/2013/03/09/using-traceevent-to-mine-information-in-os-registered-etw-providers/

Open a TraceEventSession (with a null file name to indicate a 'real time session'.  A session is a 'controller' which can turn ETW providers on and off.   

You can find out all the providers that have been registered in the operating system by running the command

logman query providers &gt; providers.txt





Using Failed Request Tracing to Trace Rewrite Rules

UrlRewrite trace event detayları adım adım açıklanmış

https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-failed-request-tracing-to-trace-rewrite-rules



Performance Monitor üzerinde tanım yapılarak

How to display URL Rewrite ETW Events in the Event Viewer

https://blogs.iis.net/danielvl/how-to-display-url-rewrite-etw-events-in-the-event-viewer

Provider: IIS: WWW Server

“Keywords(Any)” For URL Rewrite events: 0x400



How to display URL Rewrite ETW Events in the Event Viewer

https://blogs.msdn.microsoft.com/danielvl/2009/01/25/how-to-display-url-rewrite-etw-events-in-the-event-viewer/





İstek akışı anlatımı

IIS Request-Based Tracing

https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc786920(v%3dws.10)



Ado.net i etw için ayarlama

https://www.codeguru.com/csharp/.net/net_debugging/logging/article.php/c14769/ADONET-Trace-Logging.htm

https://lowleveldesign.org/2012/09/07/diagnosing-ado-net-with-etw-traces/









EventSource kayıt açmak için

https://blogs.msdn.microsoft.com/vancem/2014/03/15/walk-through-getting-started-with-etw-traceevent-nuget-samples-package/

In a previous post, I talked about the TraceEvent NuGet Library, which allows you to read and manipulate Event Tracing for Windws (ETW).   

There is a companion post about the EventSource  NuGet package which allows you to create your own ETW events (or in fact to send those events to anywhere you choose).





Tanıtım ve IE örneği

https://blogs.msdn.microsoft.com/ntdebugging/2009/08/27/part-1-etw-introduction-and-overview/

https://blogs.msdn.microsoft.com/ntdebugging/2009/09/08/part-2-exploring-and-decoding-etw-providers-using-event-log-channels/





google ETW (Tüm sistemi profile edip session keydedilince Windows Performance Analyzer da açıyor.)

https://github.com/google/UIforETW

https://randomascii.wordpress.com/2015/04/14/uiforetw-windows-performance-made-easier/



Diğer

https://blogs.msdn.microsoft.com/vancem/2012/12/20/using-tracesource-to-log-etw-data-to-a-file/

örnekler

http://book2s.com/csharp/api/microsoft/microsoft.diagnostics.tracing.session/traceeventsession/enableprovider-4.html





!!!

Uygulama açılış kapanışlarını takip etmek için

https://blogs.msdn.microsoft.com/vancem/2013/03/09/using-traceevent-to-mine-information-in-os-registered-etw-providers/

"EventLog-Application"

"Microsoft-Windows-Kernel-Process" provider kullanıyor.

session.EnableProvider("Microsoft-Windows-Kernel-Process", TraceEventLevel.Informational, 0x10);



https://randomascii.wordpress.com/2016/03/08/power-wastage-on-an-idle-laptop/

https://randomascii.wordpress.com/2015/09/01/xperf-basics-recording-a-trace-the-ultimate-easy-way/







işlem başarılı olarak tamamlanmış ama client timeout süresi dolduğu için hataya düşmüş:

http 200 / winhttp 64

https://docs.microsoft.com/en-us/windows/desktop/api/winhttp/nf-winhttp-winhttpsettimeouts



Thermal management - fan sıcaklığı vs.

Kod örneği yok.

https://docs.microsoft.com/en-us/windows-hardware/design/device-experiences/thermal-management-in-windows

örnek uygulama

Open hardware monitor



https://openhardwaremonitor.org/screenshots/
