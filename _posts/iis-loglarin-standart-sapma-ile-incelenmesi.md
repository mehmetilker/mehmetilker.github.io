# Web logları (istekler) url bazında süreyi ortalamaya ek olarak standart sapma ile hesaplamak

Örneğin sabir bir URL'e gelen 1440 isteğin ortalaması (time-taken by average) 16 hesaplanmakta.
İstklerin çoğu (%99) 10sn ile 25. saniye arasında sürmüş.
Bir istek 255 saniye sürdüğü görülüyor.
Sadece bu isteği çıkarınca ortalama süre 14sn'ye düşüyor.



Tüm istekler için standart sapma hesaplandığında 70284 çıkıyor.
255sn süren isteği çıkarınca standart sapma 5 olarak bulunuyor.

Çıkarım:
Standart sapma, volatilite (düzensizlik, oynaklık, uçuculuk, gelgeçlik, döneklik) olarak alınabilir.
Volatilite yüksek olması iseklerin yanıtlanma süreleri 
Doğru bir ortalama süre almak için önce standart sapmayı belirleyip, onun üzerindeki değerler dışarda bırakılmalı.
Standart sapma üzerindeki istek sürelerinin sayısın toplam istek sayısına oranı volatility verir.

Çan eğrisi ile isteklerin sürelerinin sağlıklı dağılıp dağılmadığına bakılabilir
https://www.statisticshowto.datasciencecentral.com/probability-and-statistics/standard-deviation/
A normal distribution of data means that most of the examples in a set of data are close to the "average," while relatively few examples tend to one extreme or the other.
https://www.robertniles.com/stats/stdev.shtml


diğer bakılabilecek değişken: variance
variance: değişiklik, ayrılı, dağılım
Using the standard deviation, statisticians may determine if the data has a normal curve or other mathematical relationship. 
If the data behaves in a normal curve, then 68 percent of the data points will fall within one standard deviation of the average, or mean data point. 
Bigger variances cause more data points to fall outside the standard deviation. 
maller variances result in more data that is close to average.


https://github.com/gabrielweyer/HorseSpeed
HorseSpeed will compute the mean, median, standard deviation, and percentile(s) of the time-taken field in an IIS log.

LogParser Studio ile Standart sapmalı sorgu çekmek için

```sql

/* Returns the number of times a particular page (in this case .as* files) was hit, with the average, minimum, and maximum time taken, along with the standard deviation. */

SELECT 
TO_LOWERCASE(cs-uri-stem) AS csUriStem, COUNT(*) AS Hits
, DIV ( MUL(1.0, SUM(time-taken)), Hits ) AS AvgTime
, SQRROOT ( SUB ( DIV ( MUL(1.0, SUM(SQR(time-taken)) ), Hits ) 
, SQR(AvgTime) ) ) AS StDev, Max(time-taken) AS Max, Min(time-taken) AS Min
, TO_REAL(STRCAT(TO_STRING(sc-status)
, STRCAT('.', TO_STRING(sc-substatus)))) AS Status
, Min(TO_LOCALTIME(date)) AS LastUpdate
FROM '[LOGFILEPATH]'
GROUP BY TO_LOWERCASE(cs-uri-stem), TO_REAL(STRCAT(TO_STRING(sc-status), STRCAT('.', TO_STRING(sc-substatus)))) 
HAVING COUNT(*) > 2
ORDER BY Hits Desc

```