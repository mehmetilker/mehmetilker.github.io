İzleme çözümü 3 adımdan oluşur.

- [Monitoring - Veri kaydetme + Görüntüleme çözümleri](#monitoring---veri-kaydetme--g%C3%B6r%C3%BCnt%C3%BCleme-%C3%A7%C3%B6z%C3%BCmleri)
    - [Bütünleşik hazır çözüm olarak ELK:](#b%C3%BCt%C3%BCnle%C5%9Fik-haz%C4%B1r-%C3%A7%C3%B6z%C3%BCm-olarak-elk)
- [Event Kaydetme - Okuma](#event-kaydetme---okuma)
        - [Sorgulama](#sorgulama)
- [Event Görüntüleme](#event-g%C3%B6r%C3%BCnt%C3%BCleme)


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
Detaylar ETW-Detaylar yayınında


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



### Sorgulama 
Query elasticsearch

https://logz.io/blog/elasticsearch-queries/

https://www.compose.com/articles/using-query-string-queries-in-elasticsearch/

https://www.timroes.de/2016/05/29/elasticsearch-kibana-queries-in-depth-tutorial/

https://dzone.com/articles/23-useful-elasticsearch-example-queries


!status:(200 OR 206 OR 404 OR 302 OR 304 OR 301) && host:"$host"


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


!!! ElasticSearch Monitoring and Maintenance tools research
https://medium.com/@dionnis/elasticsearch-monitoring-and-maintenance-tools-research-18c5fb45a747





Grafana on Elasticsearch

https://grafana.com/blog/2016/03/09/how-to-effectively-use-the-elasticsearch-data-source-in-grafana-and-solutions-to-common-pitfalls/



Diğer Notlar


Docker ile çözüm geliştirildiğinde:

Docker ile iki ayrı container da (biri ES biri Grafana) olunca birbirlerini görmüyorlar

compose da network alanı ile bu sağlanır.











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




Performance Monitor üzerinde tanım yapılarak

How to display URL Rewrite ETW Events in the Event Viewer

https://blogs.iis.net/danielvl/how-to-display-url-rewrite-etw-events-in-the-event-viewer

Provider: IIS: WWW Server

“Keywords(Any)” For URL Rewrite events: 0x400







Tanıtım ve IE örneği

https://blogs.msdn.microsoft.com/ntdebugging/2009/08/27/part-1-etw-introduction-and-overview/
https://blogs.msdn.microsoft.com/ntdebugging/2009/09/08/part-2-exploring-and-decoding-etw-providers-using-event-log-channels/



google ETW (Tüm sistemi profile edip session keydedilince Windows Performance Analyzer da açıyor.)

https://github.com/google/UIforETW

https://randomascii.wordpress.com/2015/04/14/uiforetw-windows-performance-made-easier/



örnekler

http://book2s.com/csharp/api/microsoft/microsoft.diagnostics.tracing.session/traceeventsession/enableprovider-4.html


işlem başarılı olarak tamamlanmış ama client timeout süresi dolduğu için hataya düşmüş:

http 200 / winhttp 64

https://docs.microsoft.com/en-us/windows/desktop/api/winhttp/nf-winhttp-winhttpsettimeouts




