# Lab6
vov41234567890@yandex.ru
2025-12-13

# Исследование вредоносной активности в домене Windows

## Цель работы

1.  Закрепить навыки исследования данных журнала Windows Active
    Directory
2.  Изучить структуру журнала системы Windows Active Directory
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1\. macbook ОС MacOS

2\. RStudio

3\. Интерпретатор языка R 4.5.1

## Общий план выполнения

1.  Импорт данных
2.  Нормализация данных
3.  Анализ данных

## Загрузка библиотек

``` r
options(repos = c(CRAN = "https://cloud.r-project.org"))

install.packages(c("tidyverse", "jsonlite", "lubridate"))
```


    The downloaded binary packages are in
        /var/folders/lg/2xmw4mvd0zj8v_t4qmtb_lmm0000gn/T//Rtmp9SglEf/downloaded_packages

## Импорт библиотек

``` r
library(tidyverse)
```

    Warning: package 'ggplot2' was built under R version 4.5.2

    Warning: package 'readr' was built under R version 4.5.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(jsonlite)
```


    Attaching package: 'jsonlite'

    The following object is masked from 'package:purrr':

        flatten

``` r
library(lubridate)
```

## Импорт файла

``` r
file_path <- "datasets/caldera_attack_evals_round1_day1_2019-10-20201108.json"
log_data <- stream_in(file(file_path))
```

    opening file input connection.


     Found 500 records...
     Found 1000 records...
     Found 1500 records...
     Found 2000 records...
     Found 2500 records...
     Found 3000 records...
     Found 3500 records...
     Found 4000 records...
     Found 4500 records...
     Found 5000 records...
     Found 5500 records...
     Found 6000 records...
     Found 6500 records...
     Found 7000 records...
     Found 7500 records...
     Found 8000 records...
     Found 8500 records...
     Found 9000 records...
     Found 9500 records...
     Found 10000 records...
     Found 10500 records...
     Found 11000 records...
     Found 11500 records...
     Found 12000 records...
     Found 12500 records...
     Found 13000 records...
     Found 13500 records...
     Found 14000 records...
     Found 14500 records...
     Found 15000 records...
     Found 15500 records...
     Found 16000 records...
     Found 16500 records...
     Found 17000 records...
     Found 17500 records...
     Found 18000 records...
     Found 18500 records...
     Found 19000 records...
     Found 19500 records...
     Found 20000 records...
     Found 20500 records...
     Found 21000 records...
     Found 21500 records...
     Found 22000 records...
     Found 22500 records...
     Found 23000 records...
     Found 23500 records...
     Found 24000 records...
     Found 24500 records...
     Found 25000 records...
     Found 25500 records...
     Found 26000 records...
     Found 26500 records...
     Found 27000 records...
     Found 27500 records...
     Found 28000 records...
     Found 28500 records...
     Found 29000 records...
     Found 29500 records...
     Found 30000 records...
     Found 30500 records...
     Found 31000 records...
     Found 31500 records...
     Found 32000 records...
     Found 32500 records...
     Found 33000 records...
     Found 33500 records...
     Found 34000 records...
     Found 34500 records...
     Found 35000 records...
     Found 35500 records...
     Found 36000 records...
     Found 36500 records...
     Found 37000 records...
     Found 37500 records...
     Found 38000 records...
     Found 38500 records...
     Found 39000 records...
     Found 39500 records...
     Found 40000 records...
     Found 40500 records...
     Found 41000 records...
     Found 41500 records...
     Found 42000 records...
     Found 42500 records...
     Found 43000 records...
     Found 43500 records...
     Found 44000 records...
     Found 44500 records...
     Found 45000 records...
     Found 45500 records...
     Found 46000 records...
     Found 46500 records...
     Found 47000 records...
     Found 47500 records...
     Found 48000 records...
     Found 48500 records...
     Found 49000 records...
     Found 49500 records...
     Found 50000 records...
     Found 50500 records...
     Found 51000 records...
     Found 51500 records...
     Found 52000 records...
     Found 52500 records...
     Found 53000 records...
     Found 53500 records...
     Found 54000 records...
     Found 54500 records...
     Found 55000 records...
     Found 55500 records...
     Found 56000 records...
     Found 56500 records...
     Found 57000 records...
     Found 57500 records...
     Found 58000 records...
     Found 58500 records...
     Found 59000 records...
     Found 59500 records...
     Found 60000 records...
     Found 60500 records...
     Found 61000 records...
     Found 61500 records...
     Found 62000 records...
     Found 62500 records...
     Found 63000 records...
     Found 63500 records...
     Found 64000 records...
     Found 64500 records...
     Found 65000 records...
     Found 65500 records...
     Found 66000 records...
     Found 66500 records...
     Found 67000 records...
     Found 67500 records...
     Found 68000 records...
     Found 68500 records...
     Found 69000 records...
     Found 69500 records...
     Found 70000 records...
     Found 70500 records...
     Found 71000 records...
     Found 71500 records...
     Found 72000 records...
     Found 72500 records...
     Found 73000 records...
     Found 73500 records...
     Found 74000 records...
     Found 74500 records...
     Found 75000 records...
     Found 75500 records...
     Found 76000 records...
     Found 76500 records...
     Found 77000 records...
     Found 77500 records...
     Found 78000 records...
     Found 78500 records...
     Found 79000 records...
     Found 79500 records...
     Found 80000 records...
     Found 80500 records...
     Found 81000 records...
     Found 81500 records...
     Found 82000 records...
     Found 82500 records...
     Found 83000 records...
     Found 83500 records...
     Found 84000 records...
     Found 84500 records...
     Found 85000 records...
     Found 85500 records...
     Found 86000 records...
     Found 86500 records...
     Found 87000 records...
     Found 87500 records...
     Found 88000 records...
     Found 88500 records...
     Found 89000 records...
     Found 89500 records...
     Found 90000 records...
     Found 90500 records...
     Found 91000 records...
     Found 91500 records...
     Found 92000 records...
     Found 92500 records...
     Found 93000 records...
     Found 93500 records...
     Found 94000 records...
     Found 94500 records...
     Found 95000 records...
     Found 95500 records...
     Found 96000 records...
     Found 96500 records...
     Found 97000 records...
     Found 97500 records...
     Found 98000 records...
     Found 98500 records...
     Found 99000 records...
     Found 99500 records...
     Found 1e+05 records...
     Found 100500 records...
     Found 101000 records...
     Found 101500 records...
     Found 101904 records...
     Imported 101904 records. Simplifying...

    closing file input connection.

## Проверка файла

``` r
glimpse(log_data)
```

    Rows: 101,904
    Columns: 9
    $ `@timestamp` <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.101Z", "…
    $ `@metadata`  <df[,4]> <data.frame[26 x 4]>
    $ event        <df[,4]> <data.frame[26 x 4]>
    $ log          <df[,1]> <data.frame[26 x 1]>
    $ message      <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity I…
    $ winlog       <df[,16]> <data.frame[26 x 16]>
    $ ecs          <df[,1]> <data.frame[26 x 1]>
    $ host         <df[,1]> <data.frame[26 x 1]>
    $ agent        <df[,5]> <data.frame[26 x 5]>

``` r
print(names(log_data))
```

    [1] "@timestamp" "@metadata"  "event"      "log"        "message"   
    [6] "winlog"     "ecs"        "host"       "agent"     

## Преобразование структуры файла

``` r
logs_clean <- log_data %>%
  unnest(cols = c(event, winlog, host, agent), names_sep = "_")

logs_clean <- logs_clean %>%
  select(where(~n_distinct(.) > 1))

glimpse(logs_clean)
```

    Rows: 101,904
    Columns: 21
    $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log                  <df[,1]> <data.frame[26 x 1]>
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSe…
    $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Micr…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54849…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sys…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privil…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>

## Раскройте датафрейм избавившись от вложенных датафреймов. Для обнаружения таких можно использовать функцию dplyr::glimpse(), а для раскрытия вложенности – tidyr::unnest(). Обратите внимание, что при раскрытии теряются внешние названия колонок – это можно предотвратить если использовать параметр tidyr::unnest(…, names_sep = ).

``` r
library(tidyverse)
library(lubridate)

# Проверяем структуру logs_clean
cat("Структура logs_clean:\n")
```

    Структура logs_clean:

``` r
glimpse(logs_clean)
```

    Rows: 101,904
    Columns: 21
    $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
    $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
    $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
    $ log                  <df[,1]> <data.frame[26 x 1]>
    $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSe…
    $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
    $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
    $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Micr…
    $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
    $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
    $ winlog_process       <df[,2]> <data.frame[26 x 2]>
    $ winlog_keywords      <list> "Audit Success", "Audit Failure", <NULL>, <NULL>,…
    $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54849…
    $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sys…
    $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privil…
    $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
    $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
    $ winlog_user          <df[,4]> <data.frame[26 x 4]>
    $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>

``` r
# Правильный способ раскрытия вложенных датафреймов
logs_unnested_safe <- logs_clean %>%
  # Раскрываем простые структуры
  unnest_wider(winlog_process, names_sep = "_") %>%
  unnest_wider(winlog_user, names_sep = "_") %>%
  unnest_wider(log, names_sep = "_") %>%
  # Преобразуем ключевые слова
  mutate(
    winlog_keywords = map_chr(winlog_keywords, 
                             ~ifelse(is.null(.) || length(.) == 0, 
                                    NA_character_, 
                                    paste(., collapse = ", ")))
  )

# Работаем с winlog_event_data - извлекаем отдельные колонки
# Сначала посмотрим, какие колонки есть в winlog_event_data
cat("\nКолонки в winlog_event_data (первые 20):\n")
```


    Колонки в winlog_event_data (первые 20):

``` r
event_data_cols <- names(logs_clean$winlog_event_data)
print(head(event_data_cols, 20))
```

     [1] "SubjectDomainName"     "TargetDomainName"      "SubjectUserSid"       
     [4] "SubjectUserName"       "TargetUserName"        "EnabledPrivilegeList" 
     [7] "TargetLogonId"         "ProcessName"           "ProcessId"            
    [10] "SubjectLogonId"        "TargetUserSid"         "DisabledPrivilegeList"
    [13] "ObjectServer"          "Service"               "PrivilegeList"        
    [16] "TargetProcessId"       "SourceProcessId"       "SourceProcessGUID"    
    [19] "SourceThreadId"        "SourceImage"          

``` r
# Извлекаем только ключевые колонки для безопасности
key_columns <- c("ProcessId", "Image", "IpAddress", "TargetUserName", 
                "LogonType", "SubjectUserName", "CommandLine", "ParentProcessId",
                "TargetUserSid", "SubjectUserSid")

# Добавляем выбранные колонки из winlog_event_data
for (col in key_columns) {
  if (col %in% names(logs_clean$winlog_event_data)) {
    logs_unnested_safe <- logs_unnested_safe %>%
      mutate(!!paste0("event_data_", col) := winlog_event_data[[col]])
  }
}

# Теперь удаляем исходные вложенные структуры
logs_unnested_safe <- logs_unnested_safe %>%
  select(-winlog_event_data, -winlog_user_data)

cat("\nБезопасное раскрытие завершено:\n")
```


    Безопасное раскрытие завершено:

``` r
cat("Строк:", nrow(logs_unnested_safe), "\n")
```

    Строк: 101904 

``` r
cat("Колонок:", ncol(logs_unnested_safe), "\n\n")
```

    Колонок: 33 

``` r
# Смотрим на результат
cat("Первые 10 колонок:\n")
```

    Первые 10 колонок:

``` r
print(names(logs_unnested_safe)[1:10])
```

     [1] "@timestamp"           "event_created"        "event_code"          
     [4] "event_action"         "log_level"            "message"             
     [7] "winlog_event_id"      "winlog_provider_name" "winlog_record_id"    
    [10] "winlog_computer_name"

``` r
cat("\nОбразец данных:\n")
```


    Образец данных:

``` r
glimpse(head(logs_unnested_safe, 3))
```

    Rows: 3
    Columns: 33
    $ `@timestamp`               <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:…
    $ event_created              <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:…
    $ event_code                 <int> 4703, 4673, 10
    $ event_action               <chr> "Token Right Adjusted Events", "Sensitive P…
    $ log_level                  <chr> "information", "information", "information"
    $ message                    <chr> "A token right was adjusted.\n\nSubject:\n\…
    $ winlog_event_id            <int> 4703, 4673, 10
    $ winlog_provider_name       <chr> "Microsoft-Windows-Security-Auditing", "Mic…
    $ winlog_record_id           <int> 50588, 104875, 226649
    $ winlog_computer_name       <chr> "HR001.shire.com", "HFDC01.shire.com", "IT0…
    $ winlog_process_pid         <int> 4, 4, 3220
    $ winlog_process_thread      <df[,1]> <data.frame[3 x 1]>
    $ winlog_keywords            <chr> "Audit Success", "Audit Failure", NA
    $ winlog_provider_guid       <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "…
    $ winlog_channel             <chr> "security", "Security", "Microsoft-Windows-…
    $ winlog_task                <chr> "Token Right Adjusted Events", "Sensitive P…
    $ winlog_opcode              <chr> "Info", "Info", "Info"
    $ winlog_version             <int> NA, NA, 3
    $ winlog_user_domain         <chr> NA, NA, "NT AUTHORITY"
    $ winlog_user_type           <chr> NA, NA, "User"
    $ winlog_user_identifier     <chr> NA, NA, "S-1-5-18"
    $ winlog_user_name           <chr> NA, NA, "SYSTEM"
    $ winlog_activity_id         <chr> NA, NA, NA
    $ event_data_ProcessId       <chr> "0x804", "0x494", NA
    $ event_data_Image           <chr> NA, NA, NA
    $ event_data_IpAddress       <chr> NA, NA, NA
    $ event_data_TargetUserName  <chr> "HR001$", NA, NA
    $ event_data_LogonType       <chr> NA, NA, NA
    $ event_data_SubjectUserName <chr> "HR001$", "LOCAL SERVICE", NA
    $ event_data_CommandLine     <chr> NA, NA, NA
    $ event_data_ParentProcessId <chr> NA, NA, NA
    $ event_data_TargetUserSid   <chr> "S-1-5-18", NA, NA
    $ event_data_SubjectUserSid  <chr> "S-1-5-18", "S-1-5-19", NA

``` r
# Проверяем типы данных
cat("\nТипы данных в колонках:\n")
```


    Типы данных в колонках:

``` r
print(table(sapply(logs_unnested_safe, class)))
```


     character data.frame    integer 
            27          1          5 

## Минимизируйте количество колонок в датафрейме – уберите колоки с единственным значением параметра.

``` r
# Считаем количество уникальных значений в каждой колонке (игнорируя NA)
col_unique_counts <- logs_unnested_safe %>%
  summarise(across(everything(), ~n_distinct(., na.rm = TRUE))) %>%
  pivot_longer(everything(), names_to = "column", values_to = "unique_count")

# Покажем колонки с одним уникальным значением
single_value_cols <- col_unique_counts %>%
  filter(unique_count <= 1) %>%
  pull(column)

cat("Колонки с единственным значением (будут удалены):\n")
```

    Колонки с единственным значением (будут удалены):

``` r
print(single_value_cols)
```

    character(0)

``` r
cat("\nВсего таких колонок:", length(single_value_cols), "\n\n")
```


    Всего таких колонок: 0 

``` r
# Удаляем эти колонки
logs_minimized <- logs_unnested_safe %>%
  select(-all_of(single_value_cols))

# Альтернативно, можно сделать все в одном шаге
# logs_minimized <- logs_unnested_safe %>%
#   select(where(~n_distinct(., na.rm = TRUE) > 1))

# Показываем результат
cat("После минимизации:\n")
```

    После минимизации:

``` r
cat("Было колонок:", ncol(logs_unnested_safe), "\n")
```

    Было колонок: 33 

``` r
cat("Стало колонок:", ncol(logs_minimized), "\n\n")
```

    Стало колонок: 33 

``` r
cat("Оставшиеся колонки:\n")
```

    Оставшиеся колонки:

``` r
print(names(logs_minimized))
```

     [1] "@timestamp"                 "event_created"             
     [3] "event_code"                 "event_action"              
     [5] "log_level"                  "message"                   
     [7] "winlog_event_id"            "winlog_provider_name"      
     [9] "winlog_record_id"           "winlog_computer_name"      
    [11] "winlog_process_pid"         "winlog_process_thread"     
    [13] "winlog_keywords"            "winlog_provider_guid"      
    [15] "winlog_channel"             "winlog_task"               
    [17] "winlog_opcode"              "winlog_version"            
    [19] "winlog_user_domain"         "winlog_user_type"          
    [21] "winlog_user_identifier"     "winlog_user_name"          
    [23] "winlog_activity_id"         "event_data_ProcessId"      
    [25] "event_data_Image"           "event_data_IpAddress"      
    [27] "event_data_TargetUserName"  "event_data_LogonType"      
    [29] "event_data_SubjectUserName" "event_data_CommandLine"    
    [31] "event_data_ParentProcessId" "event_data_TargetUserSid"  
    [33] "event_data_SubjectUserSid" 

``` r
# Дополнительная проверка: покажем количество уникальных значений в оставшихся колонках
cat("\nУникальные значения в оставшихся колонках:\n")
```


    Уникальные значения в оставшихся колонках:

``` r
unique_counts_final <- logs_minimized %>%
  summarise(across(everything(), ~n_distinct(., na.rm = TRUE))) %>%
  pivot_longer(everything(), names_to = "column", values_to = "unique_count") %>%
  arrange(unique_count)

print(unique_counts_final, n = 20)
```

    # A tibble: 33 × 2
       column                     unique_count
       <chr>                             <int>
     1 winlog_user_domain                    2
     2 winlog_user_type                      2
     3 event_data_LogonType                  3
     4 log_level                             4
     5 winlog_keywords                       4
     6 winlog_user_name                      4
     7 winlog_computer_name                  5
     8 winlog_version                        5
     9 winlog_user_identifier                5
    10 event_data_IpAddress                  9
    11 winlog_channel                       10
    12 winlog_opcode                        10
    13 event_data_TargetUserSid             13
    14 event_data_SubjectUserName           16
    15 event_data_SubjectUserSid            17
    16 event_data_TargetUserName            21
    17 winlog_provider_guid                 26
    18 winlog_provider_name                 28
    19 event_action                         50
    20 winlog_task                          51
    # ℹ 13 more rows

## Какое количество хостов представлено в данном датасете?

``` r
 library(tidyverse)

# 1. Проверяем наличие объекта logs_clean_final
if(!exists("logs_clean_final")) {
  cat("Объект logs_clean_final не найден.\n")
  
  # Проверяем наличие logs_clean
  if(!exists("logs_clean")) {
    cat("Объект logs_clean не найден. Создаем из исходных данных...\n")
    
    # Проверяем наличие log_data
    if(!exists("log_data")) {
      cat("Импортируем данные из файла...\n")
      file_path <- "datasets/caldera_attack_evals_round1_day1_2019-10-20201108.json"
      log_data <- jsonlite::stream_in(file(file_path))
    }
    
    # Создаем logs_clean
    logs_clean <- log_data %>%
      unnest(cols = c(event, winlog, host, agent), names_sep = "_")
    
    logs_clean <- logs_clean %>%
      select(where(~n_distinct(.) > 1))
  }
  
  # Создаем logs_clean_final из logs_clean
  logs_clean_final <- logs_clean %>%
    unnest_wider(winlog_process, names_sep = "_") %>%
    unnest_wider(winlog_user, names_sep = "_") %>%
    unnest_wider(log, names_sep = "_") %>%
    mutate(
      winlog_keywords = map_chr(winlog_keywords, 
                               ~ifelse(is.null(.) || length(.) == 0, 
                                      NA_character_, 
                                      paste(., collapse = ", ")))
    ) %>%
    mutate(
      event_data_ProcessId = winlog_event_data$ProcessId,
      event_data_Image = winlog_event_data$Image,
      event_data_IpAddress = winlog_event_data$IpAddress,
      event_data_TargetUserName = winlog_event_data$TargetUserName,
      event_data_LogonType = winlog_event_data$LogonType,
      event_data_SubjectUserName = winlog_event_data$SubjectUserName
    ) %>%
    select(-winlog_event_data, -winlog_user_data) %>%
    select(where(~n_distinct(., na.rm = TRUE) > 1))
  
  cat("Объект logs_clean_final создан.\n")
} else {
  cat("Объект logs_clean_final найден.\n")
}
```

    Объект logs_clean_final не найден.
    Объект logs_clean_final создан.

``` r
# 2. Ищем колонки с информацией о хостах
cat("\n=== ПОИСК ИНФОРМАЦИИ О ХОСТАХ ===\n")
```


    === ПОИСК ИНФОРМАЦИИ О ХОСТАХ ===

``` r
# Основная колонка с именами компьютеров
if("winlog_computer_name" %in% names(logs_clean_final)) {
  # Уникальные имена хостов
  unique_hosts <- unique(na.omit(logs_clean_final$winlog_computer_name))
  total_hosts <- length(unique_hosts)
  
  cat("\n✅ Основная колонка: winlog_computer_name\n")
  cat("📊 Количество уникальных хостов:", total_hosts, "\n")
  
  # Выводим первые 10 хостов
  cat("\n🏷️ Первые 10 хостов:\n")
  for(i in 1:min(10, length(unique_hosts))) {
    cat(i, ":", unique_hosts[i], "\n")
  }
  
  # Статистика по количеству событий
  cat("\n📈 Статистика по хостам:\n")
  host_stats <- logs_clean_final %>%
    group_by(winlog_computer_name) %>%
    summarise(
      событий = n(),
      уникальных_событий = n_distinct(winlog_event_id, na.rm = TRUE)
    ) %>%
    arrange(desc(событий))
  
  cat("\nТоп-5 хостов по количеству событий:\n")
  print(head(host_stats, 5))
  
  # Сохраняем список хостов в отдельный файл
  write_lines(unique_hosts, "hosts_list.txt")
  cat("\n📁 Список хостов сохранен в файл: hosts_list.txt\n")
  
} else {
  cat("\n❌ Колонка 'winlog_computer_name' не найдена.\n")
  
  # Ищем другие колонки с информацией о хостах
  cat("\n🔍 Поиск альтернативных колонок...\n")
  
  # Все колонки с упоминанием host, computer, machine
  possible_cols <- names(logs_clean_final)[
    grepl("host|computer|machine|agent", names(logs_clean_final), ignore.case = TRUE)
  ]
  
  if(length(possible_cols) > 0) {
    cat("Найдены следующие колонки:\n")
    for(col in possible_cols) {
      unique_count <- length(unique(na.omit(logs_clean_final[[col]])))
      cat("-", col, ":", unique_count, "уникальных значений\n")
    }
    
    # Используем первую подходящую колонку
    primary_col <- possible_cols[1]
    unique_hosts <- unique(na.omit(logs_clean_final[[primary_col]]))
    total_hosts <- length(unique_hosts)
    
    cat(sprintf("\n✅ Используем колонку: %s\n", primary_col))
    cat("📊 Количество уникальных хостов:", total_hosts, "\n")
    
    # Выводим первые 10 значений
    cat("\n🏷️ Первые 10 значений:\n")
    for(i in 1:min(10, length(unique_hosts))) {
      cat(i, ":", unique_hosts[i], "\n")
    }
  } else {
    cat("❌ Не найдено колонок с информацией о хостах.\n")
    
    # Показываем все доступные колонки
    cat("\nДоступные колонки в датафрейме:\n")
    print(names(logs_clean_final))
    
    # Предлагаем пользователю выбрать колонку вручную
    cat("\n⚠️ Выберите колонку для подсчета хостов вручную:\n")
    cat("Пример: event_data_TargetUserName или event_data_SubjectUserName\n")
  }
}
```


    ✅ Основная колонка: winlog_computer_name
    📊 Количество уникальных хостов: 5 

    🏷️ Первые 10 хостов:
    1 : HR001.shire.com 
    2 : HFDC01.shire.com 
    3 : IT001.shire.com 
    4 : ACCT001.shire.com 
    5 : FILE001.shire.com 

    📈 Статистика по хостам:

    Топ-5 хостов по количеству событий:
    # A tibble: 5 × 3
      winlog_computer_name событий уникальных_событий
      <chr>                  <int>              <int>
    1 IT001.shire.com        96296                100
    2 HFDC01.shire.com        2063                 34
    3 HR001.shire.com         1730                 19
    4 ACCT001.shire.com       1504                 19
    5 FILE001.shire.com        311                 17

    📁 Список хостов сохранен в файл: hosts_list.txt

``` r
# 3. Дополнительный анализ (если есть данные о хостах)
if(exists("total_hosts") && total_hosts > 0) {
  cat("\n=== ДОПОЛНИТЕЛЬНЫЙ АНАЛИЗ ===\n")
  
  # Анализ доменов
  cat("\n🌐 Анализ доменных имен:\n")
  
  # Проверяем, есть ли домены в именах хостов
  has_domains <- any(grepl("\\.", unique_hosts))
  
  if(has_domains) {
    # Извлекаем домены
    domains <- gsub("^.*\\.", "", unique_hosts)
    domain_table <- table(domains)
    
    cat("Распределение по доменам:\n")
    for(domain in names(domain_table)) {
      cat("-", domain, ":", domain_table[domain], "хостов\n")
    }
    
    cat("\nВсего уникальных доменов:", length(unique(domains)), "\n")
  } else {
    cat("Доменные имена не обнаружены в именах хостов.\n")
  }
  
  # Статистика по активности
  if("winlog_computer_name" %in% names(logs_clean_final)) {
    cat("\n📊 Общая статистика активности:\n")
    
    total_events <- nrow(logs_clean_final)
    events_per_host <- total_events / total_hosts
    
    cat("- Всего событий:", total_events, "\n")
    cat("- Всего хостов:", total_hosts, "\n")
    cat("- Среднее событий на хост:", round(events_per_host, 2), "\n")
    
    # Хосты с наибольшей активностью
    cat("\n🏆 Хосты с наибольшим количеством событий:\n")
    top_active <- logs_clean_final %>%
      count(winlog_computer_name, name = "количество_событий") %>%
      arrange(desc(количество_событий)) %>%
      mutate(доля_от_общего = round(количество_событий / total_events * 100, 2)) %>%
      head(5)
    
    print(top_active)
  }
}
```


    === ДОПОЛНИТЕЛЬНЫЙ АНАЛИЗ ===

    🌐 Анализ доменных имен:
    Распределение по доменам:
    - com : 5 хостов

    Всего уникальных доменов: 1 

    📊 Общая статистика активности:
    - Всего событий: 101904 
    - Всего хостов: 5 
    - Среднее событий на хост: 20380.8 

    🏆 Хосты с наибольшим количеством событий:
    # A tibble: 5 × 3
      winlog_computer_name количество_событий доля_от_общего
      <chr>                             <int>          <dbl>
    1 IT001.shire.com                   96296          94.5 
    2 HFDC01.shire.com                   2063           2.02
    3 HR001.shire.com                    1730           1.7 
    4 ACCT001.shire.com                  1504           1.48
    5 FILE001.shire.com                   311           0.31

## Подготовьте датафрейм с расшифровкой Windows Event_ID, приведите типы

данных к типу их значений.

``` r
 # Проверяем структуру logs_clean_final
cat("Доступные колонки в logs_clean_final:\n")
```

    Доступные колонки в logs_clean_final:

``` r
print(names(logs_clean_final))
```

     [1] "@timestamp"                 "event_created"             
     [3] "event_code"                 "event_action"              
     [5] "log_level"                  "message"                   
     [7] "winlog_event_id"            "winlog_provider_name"      
     [9] "winlog_record_id"           "winlog_computer_name"      
    [11] "winlog_process_pid"         "winlog_process_thread"     
    [13] "winlog_keywords"            "winlog_provider_guid"      
    [15] "winlog_channel"             "winlog_task"               
    [17] "winlog_opcode"              "winlog_version"            
    [19] "winlog_user_domain"         "winlog_user_type"          
    [21] "winlog_user_identifier"     "winlog_user_name"          
    [23] "winlog_activity_id"         "event_data_ProcessId"      
    [25] "event_data_Image"           "event_data_IpAddress"      
    [27] "event_data_TargetUserName"  "event_data_LogonType"      
    [29] "event_data_SubjectUserName"

``` r
# Создаем датафрейм с расшифровкой Windows Event ID
event_id_decoder <- tibble(
  event_id = c(4624, 4625, 4634, 4648, 4672, 4688, 4689, 4697, 4698, 4702, 4703, 4719, 4732, 4738, 4740, 4768, 4769, 4776, 5140, 5142, 5143, 5144, 5145, 5156, 5157, 5158, 10, 11, 7, 5, 4700, 4701),
  event_type = c("Успешный вход", "Неудачный вход", "Выход из системы", "Явные учетные данные использованы для входа", "Особые привилегии назначены новому входу", "Создан новый процесс", "Завершен процесс", "Установлена служба", "Создана запись планировщика заданий", "Изменен запуск задачи", "Изменение прав токена", "Изменена политика системного аудита", "Член добавлен в группу с включенной безопасностью", "Изменена учетная запись пользователя", "Заблокирована учетная запись пользователя", "Запрос билета Kerberos (TGT)", "Запрос билета службы Kerberos", "Контроллер домена попытался проверить учетные данные учетной записи", "Доступ к сетевому ресурсу", "Добавлен общий сетевой ресурс", "Удален общий сетевой ресурс", "Открыт сетевой объект", "Закрыт сетевой объект", "Брандмауэр Windows разрешил подключение", "Брандмауэр Windows заблокировал подключение", "Брандмауэр Windows разрешил привязку к порту", "Процесс создан", "Процесс завершен", "Изменение службы", "Событие аудита", "Изменение политики безопасности", "Изменение политики безопасности"),
  category = c("Аутентификация", "Аутентификация", "Аутентификация", "Аутентификация", "Аутентификация", "Создание процесса", "Создание процесса", "Изменение служб", "Изменение задач", "Изменение задач", "Изменение токена", "Изменение политики", "Изменение группы", "Изменение учетной записи", "Изменение учетной записи", "Аутентификация Kerberos", "Аутентификация Kerberos", "Аутентификация", "Доступ к сети", "Доступ к сети", "Доступ к сети", "Доступ к сети", "Доступ к сети", "Брандмауэр", "Брандмауэр", "Брандмауэр", "Создание процесса", "Создание процесса", "Изменение службы", "Аудит", "Политика безопасности", "Политика безопасности"),
  severity = c("Информация", "Предупреждение", "Информация", "Предупреждение", "Информация", "Информация", "Информация", "Предупреждение", "Предупреждение", "Предупреждение", "Предупреждение", "Информация", "Предупреждение", "Предупреждение", "Предупреждение", "Информация", "Информация", "Предупреждение", "Информация", "Информация", "Информация", "Информация", "Информация", "Информация", "Предупреждение", "Информация", "Информация", "Информация", "Предупреждение", "Информация", "Предупреждение", "Предупреждение")
)

cat("\nСоздан датафрейм с расшифровкой для", nrow(event_id_decoder), "Event ID\n")
```


    Создан датафрейм с расшифровкой для 32 Event ID

``` r
# Проверяем, какие колонки есть в logs_clean_final
cat("\nПоиск колонок для объединения...\n")
```


    Поиск колонок для объединения...

``` r
# Ищем колонку с event_id
event_id_cols <- c("winlog_event_id", "event_id", "event_code")
found_event_id_col <- NULL

for(col in event_id_cols) {
  if(col %in% names(logs_clean_final)) {
    found_event_id_col <- col
    cat("Найдена колонка для event_id:", col, "\n")
    break
  }
}
```

    Найдена колонка для event_id: winlog_event_id 

``` r
if(is.null(found_event_id_col)) {
  cat("Не найдена колонка с event_id. Доступные колонки:\n")
  print(names(logs_clean_final))
  stop("Необходима колонка с event_id для расшифровки")
}

# Объединяем с нашим основным датафреймом
logs_decoded <- logs_clean_final %>%
  # Переименовываем event_id колонку для удобства
  mutate(event_id = as.integer(!!sym(found_event_id_col))) %>%
  # Добавляем расшифровку
  left_join(event_id_decoder, by = "event_id") %>%
  # Заменяем NA на "Неизвестный" для событий без расшифровки
  mutate(
    event_type = ifelse(is.na(event_type), "Неизвестный тип события", event_type),
    category = ifelse(is.na(category), "Неизвестная категория", category),
    severity = ifelse(is.na(severity), "Неизвестно", severity)
  )

cat("Добавлена расшифровка Event ID.\n")
```

    Добавлена расшифровка Event ID.

``` r
# Преобразуем типы данных к правильным типам их значений
logs_decoded_typed <- logs_decoded %>%
  mutate(
    # Дата и время - преобразуем к POSIXct
    across(any_of(c("@timestamp", "event_created")), 
           ~as_datetime(.)),
    
    # Числовые ID - к integer
    across(any_of(c("event_id", "event_code", "winlog_record_id", 
                   "winlog_version", "event_data_LogonType")),
           ~as.integer(.)),
    
    # Текстовые поля - к character
    across(any_of(c("event_action", "message", "winlog_provider_name",
                   "winlog_computer_name", "winlog_channel", "winlog_task",
                   "event_data_Image", "event_data_IpAddress",
                   "event_data_TargetUserName", "event_data_SubjectUserName",
                   "winlog_process_name", "winlog_user_name",
                   "winlog_keywords")),
           ~as.character(.)),
    
    # Категориальные поля - к factor
    across(any_of(c("event_type", "category", "severity", "winlog_opcode")),
           ~as.factor(.))
  )

cat("\nТипы данных преобразованы.\n")
```


    Типы данных преобразованы.

``` r
# Проверяем результаты
cat("\n=== РЕЗУЛЬТАТЫ ===\n")
```


    === РЕЗУЛЬТАТЫ ===

``` r
cat("Колонки в финальном датафрейме:\n")
```

    Колонки в финальном датафрейме:

``` r
print(names(logs_decoded_typed))
```

     [1] "@timestamp"                 "event_created"             
     [3] "event_code"                 "event_action"              
     [5] "log_level"                  "message"                   
     [7] "winlog_event_id"            "winlog_provider_name"      
     [9] "winlog_record_id"           "winlog_computer_name"      
    [11] "winlog_process_pid"         "winlog_process_thread"     
    [13] "winlog_keywords"            "winlog_provider_guid"      
    [15] "winlog_channel"             "winlog_task"               
    [17] "winlog_opcode"              "winlog_version"            
    [19] "winlog_user_domain"         "winlog_user_type"          
    [21] "winlog_user_identifier"     "winlog_user_name"          
    [23] "winlog_activity_id"         "event_data_ProcessId"      
    [25] "event_data_Image"           "event_data_IpAddress"      
    [27] "event_data_TargetUserName"  "event_data_LogonType"      
    [29] "event_data_SubjectUserName" "event_id"                  
    [31] "event_type"                 "category"                  
    [33] "severity"                  

``` r
cat("\nТипы данных в финальном датафрейме:\n")
```


    Типы данных в финальном датафрейме:

``` r
type_summary <- tibble(
  колонка = names(logs_decoded_typed),
  тип = sapply(logs_decoded_typed, class)
)
print(type_summary, n = 20)
```

    # A tibble: 33 × 2
       колонка               тип         
       <chr>                 <named list>
     1 @timestamp            <chr [2]>   
     2 event_created         <chr [2]>   
     3 event_code            <chr [1]>   
     4 event_action          <chr [1]>   
     5 log_level             <chr [1]>   
     6 message               <chr [1]>   
     7 winlog_event_id       <chr [1]>   
     8 winlog_provider_name  <chr [1]>   
     9 winlog_record_id      <chr [1]>   
    10 winlog_computer_name  <chr [1]>   
    11 winlog_process_pid    <chr [1]>   
    12 winlog_process_thread <chr [1]>   
    13 winlog_keywords       <chr [1]>   
    14 winlog_provider_guid  <chr [1]>   
    15 winlog_channel        <chr [1]>   
    16 winlog_task           <chr [1]>   
    17 winlog_opcode         <chr [1]>   
    18 winlog_version        <chr [1]>   
    19 winlog_user_domain    <chr [1]>   
    20 winlog_user_type      <chr [1]>   
    # ℹ 13 more rows

``` r
# Анализируем расшифровку событий
cat("\n=== АНАЛИЗ РАСШИФРОВАННЫХ СОБЫТИЙ ===\n")
```


    === АНАЛИЗ РАСШИФРОВАННЫХ СОБЫТИЙ ===

``` r
# 1. Распределение по типам событий
cat("\n1. Топ-10 типов событий по количеству:\n")
```


    1. Топ-10 типов событий по количеству:

``` r
event_type_stats <- logs_decoded_typed %>%
  count(event_type, sort = TRUE) %>%
  head(10)
print(event_type_stats)
```

    # A tibble: 10 × 2
       event_type                                       n
       <fct>                                        <int>
     1 Неизвестный тип события                      56769
     2 Процесс создан                               36074
     3 Изменение службы                              5937
     4 Изменение прав токена                          809
     5 Брандмауэр Windows разрешил подключение        756
     6 Брандмауэр Windows разрешил привязку к порту   476
     7 Процесс завершен                               220
     8 Завершен процесс                               196
     9 Закрыт сетевой объект                          150
    10 Событие аудита                                 146

``` r
# 2. Распределение по категориям
cat("\n2. Распределение событий по категориям:\n")
```


    2. Распределение событий по категориям:

``` r
category_stats <- logs_decoded_typed %>%
  count(category, sort = TRUE)
print(category_stats)
```

    # A tibble: 10 × 2
       category                    n
       <fct>                   <int>
     1 Неизвестная категория   56769
     2 Создание процесса       36598
     3 Изменение службы         5937
     4 Брандмауэр               1232
     5 Изменение токена          809
     6 Аутентификация            209
     7 Доступ к сети             170
     8 Аудит                     146
     9 Аутентификация Kerberos    25
    10 Изменение задач             9

``` r
# 3. Распределение по уровню серьезности
cat("\n3. Распределение событий по уровню серьезности:\n")
```


    3. Распределение событий по уровню серьезности:

``` r
severity_stats <- logs_decoded_typed %>%
  count(severity, sort = TRUE)
print(severity_stats)
```

    # A tibble: 3 × 2
      severity           n
      <fct>          <int>
    1 Неизвестно     56769
    2 Информация     38379
    3 Предупреждение  6756

``` r
# 4. Неизвестные Event ID
unknown_events <- logs_decoded_typed %>%
  filter(event_type == "Неизвестный тип события") %>%
  distinct(event_id)

if(nrow(unknown_events) > 0) {
  cat("\n4. Найдены неизвестные Event ID (без расшифровки):\n")
  print(unknown_events$event_id)
  cat("Всего неизвестных Event ID:", nrow(unknown_events), "\n")
} else {
  cat("\n4. Все Event ID успешно расшифрованы.\n")
}
```


    4. Найдены неизвестные Event ID (без расшифровки):
     [1]  4673  4627    12     3   800     1    18  4106  4103  4104  4105    22
    [13]     9    13  4670  4662    17   600   400 40961 53504 40962  4690  4658
    [25]  4656  5857  7036   403     2  5858  4799  4661  4611  4663     8  6038
    [37]    54  4674  1074    23  4647  4798 10010  7002  5861  5860  1100  5859
    [49]    40    24    32  4616 20523  2010  2002 10149   258  6006   263 50104
    [61]  1136 50105 51047 51057 50106 50037    41  6009    42  6005  6013   109
    [73]   153    20    25    27     6    98   172    55    14 16962 50036 50103
    [85] 51046 10148  7026  8010  7001    37
    Всего неизвестных Event ID: 90 

``` r
# 5. Дополнительная информация
cat("\n5. Общая статистика:\n")
```


    5. Общая статистика:

``` r
cat("- Всего событий:", nrow(logs_decoded_typed), "\n")
```

    - Всего событий: 101904 

``` r
cat("- Уникальных Event ID:", n_distinct(logs_decoded_typed$event_id), "\n")
```

    - Уникальных Event ID: 109 

``` r
cat("- Уникальных типов событий:", n_distinct(logs_decoded_typed$event_type), "\n")
```

    - Уникальных типов событий: 20 

``` r
cat("- Уникальных категорий:", n_distinct(logs_decoded_typed$category), "\n")
```

    - Уникальных категорий: 10 

``` r
# Сохраняем датафрейм для дальнейшего использования
saveRDS(logs_decoded_typed, "windows_events_decoded.rds")
cat("\n📁 Датафрейм сохранен в файл: windows_events_decoded.rds\n")
```


    📁 Датафрейм сохранен в файл: windows_events_decoded.rds

``` r
# Выводим пример данных
cat("\n=== ПРИМЕР ДАННЫХ ===\n")
```


    === ПРИМЕР ДАННЫХ ===

``` r
cat("Первые 5 строк с расшифровкой:\n")
```

    Первые 5 строк с расшифровкой:

``` r
# Выбираем доступные колонки для отображения
available_cols <- c("event_id", "event_type", "category", "severity", 
                   "winlog_computer_name", "@timestamp", "event_action")

cols_to_show <- available_cols[available_cols %in% names(logs_decoded_typed)]

print(head(logs_decoded_typed %>% select(all_of(cols_to_show)), 5))
```

    # A tibble: 5 × 7
      event_id event_type category severity winlog_computer_name `@timestamp`       
         <int> <fct>      <fct>    <fct>    <chr>                <dttm>             
    1     4703 Изменение… Изменен… Предупр… HR001.shire.com      2019-10-20 20:11:06
    2     4673 Неизвестн… Неизвес… Неизвес… HFDC01.shire.com     2019-10-20 20:11:07
    3       10 Процесс с… Создани… Информа… IT001.shire.com      2019-10-20 20:11:09
    4       10 Процесс с… Создани… Информа… HR001.shire.com      2019-10-20 20:11:10
    5       10 Процесс с… Создани… Информа… ACCT001.shire.com    2019-10-20 20:11:11
    # ℹ 1 more variable: event_action <chr>

``` r
# Создаем визуализацию (если есть ggplot2)
if(requireNamespace("ggplot2", quietly = TRUE)) {
  library(ggplot2)
  
  # График топ-10 типов событий
  top_events <- logs_decoded_typed %>%
    count(event_type, sort = TRUE) %>%
    head(10)
  
  if(nrow(top_events) > 0) {
    plot1 <- ggplot(top_events, aes(x = reorder(event_type, n), y = n)) +
      geom_col(fill = "steelblue") +
      coord_flip() +
      labs(
        title = "Топ-10 типов событий Windows",
        x = "Тип события",
        y = "Количество"
      ) +
      theme_minimal()
    
    print(plot1)
  }
  
  cat("\n📊 Визуализация создана.\n")
}
```

![](lab_6.markdown_strict_files/figure-markdown_strict/unnamed-chunk-9-1.png)


    📊 Визуализация создана.

## Есть ли в логе события с высоким и средним уровнем значимости? Сколько их?

``` r
# Проверяем структуру данных и удаляем столбцы-списки
if(exists("logs_decoded_typed")) {
  # Определяем, какие колонки являются списками
  list_columns <- names(logs_decoded_typed)[sapply(logs_decoded_typed, is.list)]
  
  cat("Столбцы-списки в датафрейме:\n")
  print(list_columns)
  
  # Удаляем столбцы-списки для упрощения анализа
  logs_clean_for_analysis <- logs_decoded_typed %>%
    select(-any_of(list_columns))
  
  cat("\nПосле удаления столбцов-списков:\n")
  cat("Было столбцов:", ncol(logs_decoded_typed), "\n")
  cat("Стало столбцов:", ncol(logs_clean_for_analysis), "\n")
}
```

    Столбцы-списки в датафрейме:
    [1] "winlog_process_thread"

    После удаления столбцов-списков:
    Было столбцов: 33 
    Стало столбцов: 32 

``` r
# Теперь выполняем анализ событий по уровню значимости
cat("\n=== АНАЛИЗ СОБЫТИЙ ПО УРОВНЮ ЗНАЧИМОСТИ ===\n")
```


    === АНАЛИЗ СОБЫТИЙ ПО УРОВНЮ ЗНАЧИМОСТИ ===

``` r
# Определяем события высокого и среднего уровня на основе Event ID
high_severity_events <- c(
  4625,  # Неудачный вход
  4673,  # Важные привилегии назначены
  4697,  # Установка службы
  4702,  # Изменение запуска задачи
  4719,  # Изменение политики аудита
  4740,  # Блокировка учетной записи
  4768,  # Запрос билета Kerberos (TGT) - может указывать на атаку
  4776,  # Контроллер домена попытался проверить учетные данные
  5157   # Брандмауэр Windows заблокировал подключение
)

medium_severity_events <- c(
  4624,  # Успешный вход
  4634,  # Выход из системы
  4648,  # Явные учетные данные использованы для входа
  4672,  # Особые привилегии назначены новому входу
  4688,  # Создание процесса
  4689,  # Завершен процесс
  4698,  # Создана запись планировщика заданий
  4703,  # Изменение прав токена
  4732,  # Член добавлен в группу с включенной безопасностью
  4738,  # Изменена учетная запись пользователя
  4769,  # Запрос билета службы Kerberos
  5140,  # Доступ к сетевому ресурсу
  5142,  # Добавлен общий сетевой ресурс
  5143,  # Удален общий сетевой ресурс
  5144,  # Открыт сетевой объект
  5145,  # Закрыт сетевой объект
  5156,  # Брандмауэр Windows разрешил подключение
  5158   # Брандмауэр Windows разрешил привязку к порту
)

# Добавляем классификацию уровней значимости
logs_with_severity <- logs_clean_for_analysis %>%
  mutate(
    custom_severity = case_when(
      event_id %in% high_severity_events ~ "Высокий",
      event_id %in% medium_severity_events ~ "Средний",
      TRUE ~ "Низкий"
    ),
    custom_severity = factor(custom_severity, levels = c("Высокий", "Средний", "Низкий"))
  )

# Подсчитываем события по уровням значимости
severity_counts <- logs_with_severity %>%
  count(custom_severity)

cat("Распределение событий по уровням значимости:\n")
```

    Распределение событий по уровням значимости:

``` r
print(severity_counts)
```

    # A tibble: 3 × 2
      custom_severity     n
      <fct>           <int>
    1 Высокий           157
    2 Средний          2744
    3 Низкий          99003

``` r
# Подсчитываем общее количество событий
total_events <- nrow(logs_with_severity)

# Извлекаем количества
high_count <- severity_counts %>% 
  filter(custom_severity == "Высокий") %>% 
  pull(n) %>% 
  ifelse(length(.) > 0, ., 0)

medium_count <- severity_counts %>% 
  filter(custom_severity == "Средний") %>% 
  pull(n) %>% 
  ifelse(length(.) > 0, ., 0)

low_count <- severity_counts %>% 
  filter(custom_severity == "Низкий") %>% 
  pull(n) %>% 
  ifelse(length(.) > 0, ., 0)

# Выводим результаты
cat("\n=== РЕЗУЛЬТАТЫ ===\n")
```


    === РЕЗУЛЬТАТЫ ===

``` r
cat("Всего событий в логах:", total_events, "\n")
```

    Всего событий в логах: 101904 

``` r
cat("\nСобытия с ВЫСОКИМ уровнем значимости:\n")
```


    События с ВЫСОКИМ уровнем значимости:

``` r
cat("  Количество:", high_count, "\n")
```

      Количество: 157 

``` r
cat("  Процент от общего числа:", round(high_count / total_events * 100, 2), "%\n")
```

      Процент от общего числа: 0.15 %

``` r
cat("\nСобытия со СРЕДНИМ уровнем значимости:\n")
```


    События со СРЕДНИМ уровнем значимости:

``` r
cat("  Количество:", medium_count, "\n")
```

      Количество: 2744 

``` r
cat("  Процент от общего числа:", round(medium_count / total_events * 100, 2), "%\n")
```

      Процент от общего числа: 2.69 %

``` r
cat("\nСобытия с НИЗКИМ уровнем значимости:\n")
```


    События с НИЗКИМ уровнем значимости:

``` r
cat("  Количество:", low_count, "\n")
```

      Количество: 99003 

``` r
cat("  Процент от общего числа:", round(low_count / total_events * 100, 2), "%\n")
```

      Процент от общего числа: 97.15 %

``` r
cat("\nСобытия с ВЫСОКИМ и СРЕДНИМ уровнем вместе:\n")
```


    События с ВЫСОКИМ и СРЕДНИМ уровнем вместе:

``` r
cat("  Количество:", high_count + medium_count, "\n")
```

      Количество: 2901 

``` r
cat("  Процент от общего числа:", round((high_count + medium_count) / total_events * 100, 2), "%\n")
```

      Процент от общего числа: 2.85 %

``` r
# Детальный анализ событий высокого уровня
if(high_count > 0) {
  cat("\n=== ДЕТАЛЬНЫЙ АНАЛИЗ СОБЫТИЙ ВЫСОКОГО УРОВНЯ ===\n")
  
  high_severity_events_df <- logs_with_severity %>%
    filter(custom_severity == "Высокий")
  
  cat("Топ-10 Event ID с высоким уровнем значимости:\n")
  high_event_types <- high_severity_events_df %>%
    count(event_id, event_type, sort = TRUE) %>%
    head(10)
  print(high_event_types)
}
```


    === ДЕТАЛЬНЫЙ АНАЛИЗ СОБЫТИЙ ВЫСОКОГО УРОВНЯ ===
    Топ-10 Event ID с высоким уровнем значимости:
    # A tibble: 4 × 3
      event_id event_type                                                          n
         <int> <fct>                                                           <int>
    1     4673 Неизвестный тип события                                           143
    2     4702 Изменен запуск задачи                                               8
    3     4768 Запрос билета Kerberos (TGT)                                        5
    4     4776 Контроллер домена попытался проверить учетные данные учетной з…     1

``` r
# Детальный анализ событий среднего уровня
if(medium_count > 0) {
  cat("\n=== ДЕТАЛЬНЫЙ АНАЛИЗ СОБЫТИЙ СРЕДНЕГО УРОВНЯ ===\n")
  
  medium_severity_events_df <- logs_with_severity %>%
    filter(custom_severity == "Средний")
  
  cat("Топ-10 Event ID со средним уровнем значимости:\n")
  medium_event_types <- medium_severity_events_df %>%
    count(event_id, event_type, sort = TRUE) %>%
    head(10)
  print(medium_event_types)
}
```


    === ДЕТАЛЬНЫЙ АНАЛИЗ СОБЫТИЙ СРЕДНЕГО УРОВНЯ ===
    Топ-10 Event ID со средним уровнем значимости:
    # A tibble: 10 × 3
       event_id event_type                                       n
          <int> <fct>                                        <int>
     1     4703 Изменение прав токена                          809
     2     5156 Брандмауэр Windows разрешил подключение        756
     3     5158 Брандмауэр Windows разрешил привязку к порту   476
     4     4689 Завершен процесс                               196
     5     5145 Закрыт сетевой объект                          150
     6     4688 Создан новый процесс                           108
     7     4634 Выход из системы                                82
     8     4624 Успешный вход                                   73
     9     4672 Особые привилегии назначены новому входу        53
    10     4769 Запрос билета службы Kerberos                   20

``` r
# Простая текстовая визуализация
cat("\n=== ТЕКСТОВАЯ ВИЗУАЛИЗАЦИЯ ===\n")
```


    === ТЕКСТОВАЯ ВИЗУАЛИЗАЦИЯ ===

``` r
cat("Распределение событий по уровням значимости:\n")
```

    Распределение событий по уровням значимости:

``` r
cat("Высокий уровень : ", strrep("█", round(high_count/total_events * 50)), " ", high_count, " (", round(high_count/total_events * 100, 1), "%)\n")
```

    Высокий уровень :     157  ( 0.2 %)

``` r
cat("Средний уровень: ", strrep("█", round(medium_count/total_events * 50)), " ", medium_count, " (", round(medium_count/total_events * 100, 1), "%)\n")
```

    Средний уровень:  █   2744  ( 2.7 %)

``` r
cat("Низкий уровень : ", strrep("█", round(low_count/total_events * 50)), " ", low_count, " (", round(low_count/total_events * 100, 1), "%)\n")
```

    Низкий уровень :  █████████████████████████████████████████████████   99003  ( 97.2 %)

``` r
# Ответ на вопрос
cat("\n=== ОТВЕТ НА ВОПРОС ===\n")
```


    === ОТВЕТ НА ВОПРОС ===

``` r
if(high_count > 0) {
  cat("✅ ДА, в логах есть события с высоким уровнем значимости. Их количество:", high_count, "\n")
} else {
  cat("❌ НЕТ, в логах нет событий с высоким уровнем значимости.\n")
}
```

    ✅ ДА, в логах есть события с высоким уровнем значимости. Их количество: 157 

``` r
if(medium_count > 0) {
  cat("✅ ДА, в логах есть события со средним уровнем значимости. Их количество:", medium_count, "\n")
} else {
  cat("❌ НЕТ, в логах нет событий со средним уровнем значимости.\n")
}
```

    ✅ ДА, в логах есть события со средним уровнем значимости. Их количество: 2744 

``` r
cat("\nСуммарно событий с высоким и средним уровнем значимости:", high_count + medium_count, "\n")
```


    Суммарно событий с высоким и средним уровнем значимости: 2901 

## Выводы

1.  Закрепили навыки исследования данных журнала Windows Active
    Directory
2.  Изучили структуру журнала системы Windows Active Directory
3.  Зекрепили практические навыки использования языка программирования R
    для обработки данных
4.  Закрепили знания основных функций обработки данных экосистемы
    tidyverse языка R
