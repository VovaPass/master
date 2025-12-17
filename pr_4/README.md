# Lab4
vov41234567890@yandex.ru
2025-11-22

# Цель работы

1.Зекрепить практические навыки использования языка программирования R
для обработки данных

2.Закрепить знания основных функций обработки данных экосистемы
tidyverse языка R

3.Закрепить навыки исследования метаданных DNS трафика

# План

1.  Импортируйте данные DNS –
    https://storage.yandexcloud.net/dataset.ctfsec/dns.zip Данные были
    собраны с помощью сетевого анализатора zeek

2.  Добавьте пропущенные данные о структуре данных (назначении столбцов)

3.  Преобразуйте данные в столбцах в нужный формат,просмотрите общую
    структуру данных с помощью функции glimpse()

4.  Сколько участников информационного обмена всети Доброй Организации?

5.  Какое соотношение участников обмена внутрисети и участников
    обращений к внешним ресурсам?

6.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность.

7.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений

8.  Опеределите базовые статистические характеристики (функция summary()
    ) интервала времени между последовательными обращениями к топ-10
    доменам.

9.  Часто вредоносное программное обеспечение использует DNS канал в
    качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?

10. Определите местоположение (страну, город) и организацию-провайдера
    для топ-10 доменов. Для этого можно использовать сторонние
    сервисы,например http://ip-api.com (API-эндпоинт –
    http://ip-api.com/json).

``` r
print(sessionInfo())
```

    R version 4.5.1 (2025-06-13)
    Platform: aarch64-apple-darwin20
    Running under: macOS Tahoe 26.1

    Matrix products: default
    BLAS:   /Library/Frameworks/R.framework/Versions/4.5-arm64/Resources/lib/libRblas.0.dylib 
    LAPACK: /Library/Frameworks/R.framework/Versions/4.5-arm64/Resources/lib/libRlapack.dylib;  LAPACK version 3.12.1

    locale:
    [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8

    time zone: Europe/Moscow
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    loaded via a namespace (and not attached):
     [1] compiler_4.5.1    fastmap_1.2.0     cli_3.6.5         tools_4.5.1      
     [5] htmltools_0.5.8.1 rstudioapi_0.17.1 yaml_2.3.10       rmarkdown_2.30   
     [9] knitr_1.50        jsonlite_2.0.0    xfun_0.54         digest_0.6.37    
    [13] rlang_1.1.6       evaluate_1.0.5   

## Произведем установку и загрузку вспомогательных файлов

``` r
# Устанавливаем зеркало CRAN
options(repos = c(CRAN = "https://cloud.r-project.org"))

# Устанавливаем пакеты
install.packages("dplyr", repos = "https://cloud.r-project.org")
```


    The downloaded binary packages are in
        /var/folders/lg/2xmw4mvd0zj8v_t4qmtb_lmm0000gn/T//RtmpFVL1UA/downloaded_packages

``` r
install.packages("readr")
```


    The downloaded binary packages are in
        /var/folders/lg/2xmw4mvd0zj8v_t4qmtb_lmm0000gn/T//RtmpFVL1UA/downloaded_packages

``` r
install.packages("tidyr") 
```


    The downloaded binary packages are in
        /var/folders/lg/2xmw4mvd0zj8v_t4qmtb_lmm0000gn/T//RtmpFVL1UA/downloaded_packages

``` r
install.packages("stringr")
```


    The downloaded binary packages are in
        /var/folders/lg/2xmw4mvd0zj8v_t4qmtb_lmm0000gn/T//RtmpFVL1UA/downloaded_packages

``` r
library(readr)
```

    Warning: package 'readr' was built under R version 4.5.2

``` r
library(tidyr)
library(stringr)
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

## Импорт данных

``` r
url <- "https://storage.yandexcloud.net/dataset.ctfsec/dns.zip"
 
 download.file(url, destfile = "dns.zip", mode = "wb") 
```

## Распаковка данных из архива

``` r
unzip("dns.zip", exdir = "dns_data")
```

## Проверка работоспособности импортированных данных

``` r
 readLines("dns_data/dns.log", n = 50)
```

     [1] "1331901005.510000\tCWGtK431H9XuaTN4fi\t192.168.202.100\t45658\t192.168.27.203\t137\tudp\t33008\t*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\t1\tC_INTERNET\t33\tSRV\t0\tNOERROR\tF\tF\tF\tF\t1\t-\t-\tF"
     [2] "1331901015.070000\tC36a282Jljz7BsbGH\t192.168.202.76\t137\t192.168.202.255\t137\tudp\t57402\tHPE8AA67\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
     [3] "1331901015.820000\tC36a282Jljz7BsbGH\t192.168.202.76\t137\t192.168.202.255\t137\tudp\t57402\tHPE8AA67\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
     [4] "1331901016.570000\tC36a282Jljz7BsbGH\t192.168.202.76\t137\t192.168.202.255\t137\tudp\t57402\tHPE8AA67\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
     [5] "1331901005.860000\tC36a282Jljz7BsbGH\t192.168.202.76\t137\t192.168.202.255\t137\tudp\t57398\tWPAD\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                             
     [6] "1331901006.610000\tC36a282Jljz7BsbGH\t192.168.202.76\t137\t192.168.202.255\t137\tudp\t57398\tWPAD\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                             
     [7] "1331901007.360000\tC36a282Jljz7BsbGH\t192.168.202.76\t137\t192.168.202.255\t137\tudp\t57398\tWPAD\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                             
     [8] "1331901006.370000\tClEZCt3GLkJdtGGmAa\t192.168.202.89\t137\t192.168.202.255\t137\tudp\t62187\tEWREP1\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                          
     [9] "1331901007.120000\tClEZCt3GLkJdtGGmAa\t192.168.202.89\t137\t192.168.202.255\t137\tudp\t62187\tEWREP1\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                          
    [10] "1331901007.870000\tClEZCt3GLkJdtGGmAa\t192.168.202.89\t137\t192.168.202.255\t137\tudp\t62187\tEWREP1\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                          
    [11] "1331901007.090000\tCpD4i41jyaYqmTBMH3\t192.168.202.89\t137\t10.7.136.159\t137\tudp\t62190\t*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\t1\tC_INTERNET\t33\tSRV\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"           
    [12] "1331901008.590000\tCpD4i41jyaYqmTBMH3\t192.168.202.89\t137\t10.7.136.159\t137\tudp\t62190\t*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\t1\tC_INTERNET\t33\tSRV\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"           
    [13] "1331901010.090000\tCpD4i41jyaYqmTBMH3\t192.168.202.89\t137\t10.7.136.159\t137\tudp\t62190\t*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\t1\tC_INTERNET\t33\tSRV\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"           
    [14] "1331901017.580000\tCNR6Mr4ep4UVh9tSxj\t192.168.202.100\t45658\t192.168.27.103\t5353\tudp\t0\t_services._dns-sd._udp.local\t1\tC_INTERNET\t12\tPTR\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"                                                    
    [15] "1331901018.380000\tCexi8C2x1d9kI3a8Hd\t192.168.202.100\t45659\t192.168.27.103\t5353\tudp\t0\t_services._dns-sd._udp.local\t1\tC_INTERNET\t12\tPTR\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"                                                    
    [16] "1331901024.790000\tCKGSIMayT9QWEqORb\t192.168.202.100\t45658\t192.168.27.202\t137\tudp\t33008\t*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\t1\tC_INTERNET\t33\tSRV\t0\tNOERROR\tF\tF\tF\tF\t1\t-\t-\tF" 
    [17] "1331901025.830000\tCbOoya1FcP9upkiSN5\t192.168.202.85\t137\t192.168.202.255\t137\tudp\t34107\tSDS-MACBOOK-PRO\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                 
    [18] "1331901025.840000\tCiQXPA3Dtwoz9CV0ii\t192.168.202.102\t137\t192.168.202.255\t137\tudp\t32821\tC02GN35UDJWR\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                   
    [19] "1331901025.830000\tCiQXPA3Dtwoz9CV0ii\t192.168.202.102\t137\t192.168.202.255\t137\tudp\t32818\tSDS-MACBOOK-PRO\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                
    [20] "1331901006.800000\tCgrcup1c5uGRx428V7\t192.168.202.93\t60821\t172.19.1.100\t53\tudp\t3550\twww.apple.com\t1\tC_INTERNET\t28\tAAAA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                    
    [21] "1331901007.800000\tCgrcup1c5uGRx428V7\t192.168.202.93\t60821\t172.19.1.100\t53\tudp\t3550\twww.apple.com\t1\tC_INTERNET\t28\tAAAA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                    
    [22] "1331901010.800000\tCgrcup1c5uGRx428V7\t192.168.202.93\t60821\t172.19.1.100\t53\tudp\t35599\twww.apple.com\t1\tC_INTERNET\t28\tAAAA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                   
    [23] "1331901019.820000\tCgrcup1c5uGRx428V7\t192.168.202.93\t60821\t172.19.1.100\t53\tudp\t35599\twww.apple.com\t1\tC_INTERNET\t28\tAAAA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                   
    [24] "1331901010.800000\tCN3iol3Ge5ULjbEFph\t192.168.202.93\t61184\t172.19.1.100\t53\tudp\t40931\twww.apple.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                       
    [25] "1331901019.820000\tCN3iol3Ge5ULjbEFph\t192.168.202.93\t61184\t172.19.1.100\t53\tudp\t40931\twww.apple.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                       
    [26] "1331901006.800000\tCN3iol3Ge5ULjbEFph\t192.168.202.93\t61184\t172.19.1.100\t53\tudp\t25983\twww.apple.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                       
    [27] "1331901007.800000\tCN3iol3Ge5ULjbEFph\t192.168.202.93\t61184\t172.19.1.100\t53\tudp\t25983\twww.apple.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                       
    [28] "1331901024.010000\tCDzHPo17B429xLtaVb\t192.168.202.97\t59011\t156.154.70.22\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                     
    [29] "1331901026.030000\tCDzHPo17B429xLtaVb\t192.168.202.97\t59011\t156.154.70.22\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                     
    [30] "1331901028.040000\tCDzHPo17B429xLtaVb\t192.168.202.97\t59011\t156.154.70.22\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                     
    [31] "1331901032.050000\tCDzHPo17B429xLtaVb\t192.168.202.97\t59011\t156.154.70.22\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                     
    [32] "1331901025.010000\tCOWLuQ2XIWQK3t4VPj\t192.168.202.97\t59011\t8.26.56.26\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                        
    [33] "1331901028.040000\tCOWLuQ2XIWQK3t4VPj\t192.168.202.97\t59011\t8.26.56.26\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                        
    [34] "1331901032.050000\tCOWLuQ2XIWQK3t4VPj\t192.168.202.97\t59011\t8.26.56.26\t53\tudp\t58389\twww.comodo.com\t1\tC_INTERNET\t1\tA\t-\t-\tF\tF\tT\tF\t0\t-\t-\tF"                                                                        
    [35] "1331901025.610000\tCXxpisATPtm5jKlP\tfe80::ba8d:12ff:fe53:a8d8\t5353\tff02::fb\t5353\tudp\t0\t-\t-\t-\t-\t-\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"                                                                                          
    [36] "1331901030.460000\tCXxpisATPtm5jKlP\tfe80::ba8d:12ff:fe53:a8d8\t5353\tff02::fb\t5353\tudp\t0\t-\t-\t-\t-\t-\t-\t-\tF\tF\tF\tF\t0\t-\t-\tF"                                                                                          
    [37] "1331901025.830000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19333\t(empty)\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
    [38] "1331901026.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19333\t(empty)\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
    [39] "1331901027.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19333\t(empty)\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
    [40] "1331901025.830000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19341\tMSHOME\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                          
    [41] "1331901026.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19333\t(empty)\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
    [42] "1331901027.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19333\t(empty)\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
    [43] "1331901025.830000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19317\t\\x01\\x02__MSBROWSE__\\x02\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                     
    [44] "1331901026.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19317\t\\x01\\x02__MSBROWSE__\\x02\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                     
    [45] "1331901027.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19333\t(empty)\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                         
    [46] "1331901025.830000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19325\tCOLLECTIVE\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                      
    [47] "1331901026.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19325\tCOLLECTIVE\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                      
    [48] "1331901027.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19325\tCOLLECTIVE\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                      
    [49] "1331901025.830000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19337\tMSHOME\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                          
    [50] "1331901026.840000\tCTgyMB2GL0FCCKgv04\t192.168.202.71\t137\t192.168.202.255\t137\tudp\t19325\tCOLLECTIVE\t1\tC_INTERNET\t32\tNB\t-\t-\tF\tF\tT\tF\t1\t-\t-\tF"                                                                      

## Преобразование данных в нужный формат

``` r
library(readr)

dns <- read_tsv("dns_data/dns.log", col_names = FALSE, col_types = cols(.default = "c"), quote = "")

colnames(dns) <- c("ts", "uid", "id_orig_h", "id_orig_p", "id_resp_h", "id_resp_p",
                   "proto", "trans_id", "query", "qclass", "qclass_name", "qtype",
                   "qtype_name", "rcode", "rcode_name", "AA", "TC", "RD", "RA",
                   "Z", "answers", "TTLs", "rejected")
```

## Просмотр с помощью функции glimpse()

``` r
glimpse(dns)
```

    Rows: 427,935
    Columns: 23
    $ ts          <chr> "1331901005.510000", "1331901015.070000", "1331901015.8200…
    $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
    $ id_orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
    $ id_orig_p   <chr> "45658", "137", "137", "137", "137", "137", "137", "137", …
    $ id_resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
    $ id_resp_p   <chr> "137", "137", "137", "137", "137", "137", "137", "137", "1…
    $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
    $ trans_id    <chr> "33008", "57402", "57402", "57402", "57398", "57398", "573…
    $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
    $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
    $ qtype       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32"…
    $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
    $ rcode       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rcode_name  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA          <chr> "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F"…
    $ TC          <chr> "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F"…
    $ RD          <chr> "F", "T", "T", "T", "T", "T", "T", "T", "T", "T", "F", "F"…
    $ RA          <chr> "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F"…
    $ Z           <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "0", "0"…
    $ answers     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLs        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected    <chr> "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F", "F"…

## Подсчет кол-ва участников информационного обмена в сети

``` r
length(unique(dns$id_orig_h[str_detect(dns$id_orig_h, "^192\\.168\\.202\\.")]))
```

    [1] 89

## Соотношение участников обмена внутри

сети и участников обращений к внешним ресурсам?

``` r
summarise(dns, ratio = length(unique(c(id_orig_h[grepl("^192\\.168\\.", id_orig_h)], id_resp_h[grepl("^192\\.168\\.", id_resp_h)]))) / length(unique(id_orig_h[grepl("^192\\.168\\.", id_orig_h) & !grepl("^192\\.168\\.", id_resp_h)])))
```

    # A tibble: 1 × 1
      ratio
      <dbl>
    1  59.0

## Топ-10 участников сети, проявляющих наибольшую сетевую активность.

``` r
dns %>% count(id_orig_h, sort = TRUE) %>% top_n(10, n)
```

    # A tibble: 10 × 2
       id_orig_h           n
       <chr>           <int>
     1 10.10.117.210   75943
     2 192.168.202.93  26522
     3 192.168.202.103 18121
     4 192.168.202.76  16978
     5 192.168.202.97  16176
     6 192.168.202.141 14967
     7 10.10.117.209   14222
     8 192.168.202.110 13372
     9 192.168.203.63  12148
    10 192.168.202.106 10784

## Топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
 dns %>% count(query, sort = TRUE) %>% top_n(10, n)
```

    # A tibble: 10 × 2
       query                                                                       n
       <chr>                                                                   <int>
     1 "teredo.ipv6.microsoft.com"                                             39273
     2 "tools.google.com"                                                      14057
     3 "www.apple.com"                                                         13390
     4 "time.apple.com"                                                        13109
     5 "safebrowsing.clients.google.com"                                       11658
     6 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x… 10401
     7 "WPAD"                                                                   9134
     8 "44.206.168.192.in-addr.arpa"                                            7248
     9 "HPE8AA67"                                                               6929
    10 "ISATAP"                                                                 6569

## Опеределите базовые статистические характеристики (функция summary() ) интервала времени между последовательными обращениями к топ-10 доменам.

``` r
top_domains <- dns %>%
  group_by(query) %>%
  summarise(n = n()) %>%
  arrange(desc(n)) %>%
  head(10) %>%
  pull(query)

# Теперь выполните ваш запрос
result <- dns %>%
  filter(query %in% top_domains) %>%
  arrange(query, as.numeric(ts)) %>%
  group_by(query) %>%
  mutate(interval = c(NA, diff(as.numeric(ts)))) %>%
  summarise(interval_stats = list(summary(interval, na.rm = TRUE)))

# Выведите результат
result
```

    # A tibble: 10 × 2
       query                                                          interval_stats
       <chr>                                                          <list>        
     1 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x0… <smmryDfl [7]>
     2 "44.206.168.192.in-addr.arpa"                                  <smmryDfl [7]>
     3 "HPE8AA67"                                                     <smmryDfl [7]>
     4 "ISATAP"                                                       <smmryDfl [7]>
     5 "WPAD"                                                         <smmryDfl [7]>
     6 "safebrowsing.clients.google.com"                              <smmryDfl [7]>
     7 "teredo.ipv6.microsoft.com"                                    <smmryDfl [7]>
     8 "time.apple.com"                                               <smmryDfl [7]>
     9 "tools.google.com"                                             <smmryDfl [7]>
    10 "www.apple.com"                                                <smmryDfl [7]>

## Часто вредоносное программное обеспечение использует DNS канал в качестве канала управления, периодически отправляя запросы на подконтрольный злоумышленникам DNS сервер. По периодическим запросам на один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP адреса в исследуемом датасете?

``` r
dns %>%
  arrange(id_orig_h, query, as.numeric(ts)) %>%
  group_by(id_orig_h, query) %>%
  mutate(int = diff(c(NA, as.numeric(ts)))) %>%
  summarise(sd_int = sd(int, na.rm = TRUE),
            mean_int = mean(int, na.rm = TRUE),
            n = n(),
            .groups="drop") %>%
  filter(n > 5, sd_int < mean_int * 0.1, mean_int > 0) %>%
  arrange(sd_int)
```

    # A tibble: 112 × 5
       id_orig_h       query                      sd_int mean_int     n
       <chr>           <chr>                       <dbl>    <dbl> <int>
     1 192.168.202.122 SA.WINDOWS.COM        0              0.75      6
     2 192.168.100.130 MYGROUP               0.000000107    0.270     6
     3 192.168.202.146 XP_BASE               0.000000658    0.750     8
     4 192.168.202.65  MINIME                0.000000798    0.750     6
     5 192.168.203.45  technet.microsoft.com 0.00414        5.00     16
     6 192.168.203.64  maps.google.com       0.00445        5.00     32
     7 192.168.202.94  www.iana.org          0.00445        5.00     32
     8 192.168.202.115 EIVXGJQRTS            0.00447        0.748     6
     9 192.168.202.115 UWPKDLUJMR            0.00447        0.748     6
    10 192.168.202.115 VOAGETTQEU            0.00447        0.748     6
    # ℹ 102 more rows

## Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов. Для этого можно использовать сторонние сервисы, например http:/ /ip-api.com (API-эндпоинт – http:/ /ip-api.com/json).

``` r
library(dplyr)
library(httr)
library(jsonlite)

## Топ-домены
top_domains <- c(
  "teredo.ipv6.microsoft.com",
  "tools.google.com",
  "www.apple.com",
  "time.apple.com",
  "safebrowsing.clients.google.com",
  "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00",
  "WPAD",
  "44.206.168.192.in-addr.arpa",
  "HPE8AA67",
  "ISATAP"
)

## Функция для получения геоданных по домену
get_geo_info <- function(domain) {
  url <- paste0("http://ip-api.com/json/", domain)
  res <- try(GET(url), silent = TRUE)
  
  if(inherits(res, "try-error")) return(data.frame(domain=domain, country=NA, city=NA, isp=NA))
  
  data <- fromJSON(content(res, as="text", encoding="UTF-8"))
  
  if(data$status == "success") {
    return(data.frame(
      domain = domain,
      country = data$country,
      city    = data$city,
      isp     = data$isp,
      stringsAsFactors = FALSE
    ))
  } else {
    return(data.frame(domain=domain, country=NA, city=NA, isp=NA, stringsAsFactors = FALSE))
  }
}

## Применяем к каждому домену
geo_df <- bind_rows(lapply(top_domains, get_geo_info))

## Результат
print(geo_df)
```

                                                                        domain
    1                                                teredo.ipv6.microsoft.com
    2                                                         tools.google.com
    3                                                            www.apple.com
    4                                                           time.apple.com
    5                                          safebrowsing.clients.google.com
    6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    7                                                                     WPAD
    8                                              44.206.168.192.in-addr.arpa
    9                                                                 HPE8AA67
    10                                                                  ISATAP
             country          city                 isp
    1           <NA>          <NA>                <NA>
    2  United States      New York          Google LLC
    3  United States    El Segundo Akamai Technologies
    4  United States       Atlanta          Apple Inc.
    5  United States Mountain View          Google LLC
    6           <NA>          <NA>                <NA>
    7           <NA>          <NA>                <NA>
    8           <NA>          <NA>                <NA>
    9           <NA>          <NA>                <NA>
    10          <NA>          <NA>                <NA>

# Оценка результата

В рамках практческой работы была исследована подозрительная сетевая
активность во внутренней сети Доброй Организации. Были восстановлены
недостающие метаданные и подготовлены ответы на вопросы.

# Вывод

Таким мобразом в ходе работы мы зекрепили практические навыки
использования языка программирования R для обработки данных, знания
основных функций обработки данных экосистемы tidyverse языка R и навыки
исследования метаданных DNS трафика
