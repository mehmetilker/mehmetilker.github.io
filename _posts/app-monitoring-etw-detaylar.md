

Bir uygulada log tutuluyorsa ve ve bu loglar ETW'ye yönlendirilebiliyorsa, aşağıdaki kütüphaneler ile ETW'ye subscribe olup, Provider+Event filtreleme yöntemiyle bu event'ler toplanabilir.

Örnek: IIS logları, bir uygulamadaki System.Net namespace'i altındaki operasyonlar (HttpWebRequest), Ado.Net, .NET CLR Events

>Bir uygulamadan ETW'ye log yazmak için: EventSource
https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.EventSource/

>Okumak için: TraceEvent
https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent

## Tracing alternatifleri

> CMD logman
> APP Performance Monitor > Data Collector Sets > Event Trace Session > [New Data Collector Set] + Providers 
> APP PerfView > Collect
> CODE Microsoft.Diagnostics.Tracing.TraceEvent library


### Kod ile süreç:
Trace işlemi için önce bir session ve bu session üzerinde istenen Event Provider'lar aktfi edilmeli.
Proviver listesi:
```CMD
logman query providers > plist.txt
```
Session > Realtime, dosya (.etl) veya her ikisi de olabilir.
TraceEventSession ile session oluşturulur.
ETWTraceEventSource ile session'da oluşturulan bilgiler okunur.
Özet:
https://blogs.msdn.microsoft.com/dotnet/2013/08/15/announcing-traceevent-monitoring-and-diagnostics-for-the-cloud/




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

---Bir uygulada birden fazla session başlatabilmek için

```C#
Task.Factory.StartNew(delegate
{
      _traceSource.Process()
});
				
```


# Uygulamada app.config ile tracing

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


### github üzerinde proje bilgileri
https://github.com/Microsoft/perfview/blob/master/documentation/TraceEvent/TraceEventLibrary.md
The Microsoft.Diagnostics.Tracing.TraceEvent Library
https://github.com/Microsoft/Microsoft.Diagnostics.Tracing.Logging
Logging using ETW and EventSource


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

### IIS log events

https://docs.microsoft.com/en-us/windows/desktop/http/types-of-errors-logged-by-the-http-server-api
HTTPERR: Types of Errors Logged by the HTTP Server API


## Event yöntemi için Reactive kullanımı

Bir işlem bir event için gerekli değil ama bir işlem için birden fazla event yayınlanan durumlarda (Farklı provider'lardan birbiriyle  ilişkili Start ... Stop) event'leri gruplayıp (merge işlemi) üzerinde çalışmak gerektiğinde Observable sınıfı kullanılabilir.

ETW-Reactive
https://github.com/tomasr/iis-etw-tracing
https://github.com/tomasr/frebrilator/blob/master/Frebrilator/EventAggregator.cs
https://github.com/neuecc/EtwStream
https://github.com/Microsoft/perfview/tree/master/src/TraceEvent/Samples


Reactive 
https://www.tonytruong.net/starting-reactive-extensions-with-existing-events-in-c/
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
https://docs.microsoft.com/en-us/dotnet/framework/debug-trace-profile/trace-listeners
Interpreting Network Tracing
