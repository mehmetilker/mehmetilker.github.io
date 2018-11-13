

https://github.com/Microsoft/perfview/blob/master/documentation/TraceEvent/TraceEventLibrary.md
The Microsoft.Diagnostics.Tracing.TraceEvent Library
https://github.com/Microsoft/Microsoft.Diagnostics.Tracing.Logging
Logging using ETW and EventSource


https://docs.microsoft.com/en-us/windows/desktop/http/types-of-errors-logged-by-the-http-server-api
HTTPERR: Types of Errors Logged by the HTTP Server API




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

```XML

<sharedListeners>

      <add name="MyConsole" type="System.Diagnostics.ConsoleTraceListener"/>

      <add name="MyTraceFile" type="System.Diagnostics.TextWriterTraceListener" initializeData="System.Net.trace.log" traceOutputOptions="DateTime, ProcessId, ThreadId" />

      <!--traceOutputOptions="Timestamp" traceOutputOptions="LogicalOperationStack, DateTime, Timestamp, Callstack"-->

      <add name="ETWListener" initializeData="{BDE5930E-34C9-4E2F-A6EC-89E1F1EA69CC}"

           type="System.Diagnostics.Eventing.EventProviderTraceListener, System.Core, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />

      <add name="MyEventLog" initializeData="XAppTraceListenerLog" type="System.Diagnostics.EventLogTraceListener" />

    </sharedListeners>
```

```CMD
WinInet ile sadece internet explorer/edge isteklerini takip
logman start "wininettrace"  -p "microsoft-windows-wininet" -o "wininettrace.etl" -ets
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

