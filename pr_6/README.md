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

# Практика: Исследование вредоносной активности в домене Windows

## Установка и загрузка необходимых библиотек

``` r
library(dplyr)     
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
library(tidyr)      
library(jsonlite)  
library(lubridate)
```


    Attaching package: 'lubridate'

    The following objects are masked from 'package:base':

        date, intersect, setdiff, union

``` r
library(stringr)    
library(ggplot2)
```

    Warning: package 'ggplot2' was built under R version 4.5.2

## Подготовка данных

``` r
dataset_url <- "https://storage.yandexcloud.net/iamcth-data/dataset.tar.gz"
temp_archive <- "security_logs.tar.gz"
extract_dir <- "datasets"
target_file <- "caldera_attack_evals_round1_day1_2019-10-20201108.json"
```

## Загрузка архива

``` r
if (!file.exists(temp_archive)) {
  download_result <- tryCatch({
    download.file(dataset_url, temp_archive, mode = "wb", quiet = TRUE)
    cat(" Архив успешно загружен\n")
    TRUE
  }, error = function(e) {
    cat(" Ошибка при загрузке:", e$message, "\n")
    stop("Не удалось загрузить данные")
  })
}
```

## Распаковка архива (только если файла еще нет)

``` r
if (!file.exists(file.path(extract_dir, target_file))) {
  tryCatch({
    if (!dir.exists(extract_dir)) {
      dir.create(extract_dir, showWarnings = FALSE)
    }
    untar(temp_archive, exdir = extract_dir)
    cat(" Архив успешно распакован в", extract_dir, "\n")
  }, error = function(e) {
    cat("Ошибка при распаковке:", e$message, "\n")
    stop("Не удалось распаковать архив")
  })
}
```

## Загрузка JSON файла

``` r
json_file_path <- file.path(extract_dir, target_file)

if (!file.exists(json_file_path)) {
  stop("Файл не найден: ", json_file_path)
}

cat("Загружаем файл:", json_file_path, "\n")
```

    Загружаем файл: datasets/caldera_attack_evals_round1_day1_2019-10-20201108.json 

``` r
log_data <- tryCatch({
  con <- file(json_file_path, "r")
  data <- stream_in(con, verbose = FALSE)
  close(con)
  cat("Данные успешно импортированы через stream_in\n")
  cat("Размер данных:", nrow(data), "строк,", ncol(data), "колонок\n")
  data
}, error = function(e) {
  cat("Ошибка при чтении через stream_in:", e$message, "\n")
  cat("Попытка альтернативного метода загрузки...\n")
  
  tryCatch({
    data <- readLines(json_file_path, warn = FALSE) %>%
      paste(collapse = "") %>%
      fromJSON()
    cat(" Данные успешно импортированы через readLines + fromJSON\n")
    data
  }, error = function(e2) {
    cat(" Ошибка при альтернативном чтении:", e2$message, "\n")
    stop("Не удалось прочитать JSON файл")
  })
})
```

    Данные успешно импортированы через stream_in
    Размер данных: 101904 строк, 9 колонок

## Предварительный анализ структуры данных

``` r
cat("Структура данных:\n")
```

    Структура данных:

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
cat("\nКолонки в датафрейме:\n")
```


    Колонки в датафрейме:

``` r
print(names(log_data))
```

    [1] "@timestamp" "@metadata"  "event"      "log"        "message"   
    [6] "winlog"     "ecs"        "host"       "agent"     

## 1. Раскройте датафрейм избавившись от вложенных датафреймов

### Раскрываем вложенные датафреймы

``` r
logs_clean <- log_data %>%
  unnest(cols = c(event, winlog, host, agent), names_sep = "_")

cat("После первого раскрытия:\n")
```

    После первого раскрытия:

``` r
cat("Строк:", nrow(logs_clean), "\n")
```

    Строк: 101904 

``` r
cat("Колонок:", ncol(logs_clean), "\n")
```

    Колонок: 31 

### Убираем колонки с одним уникальным значением

``` r
logs_clean <- logs_clean %>%
  select(where(~n_distinct(.) > 1))

cat("После удаления колонок с одним значением:\n")
```

    После удаления колонок с одним значением:

``` r
cat("Строк:", nrow(logs_clean), "\n")
```

    Строк: 101904 

``` r
cat("Колонок:", ncol(logs_clean), "\n")
```

    Колонок: 21 

### Дополнительно раскрываем вложенные датафреймы с names_sep

``` r
logs_fully_unnested <- logs_clean %>%
  # Раскрываем небольшие датафреймы
  unnest_wider(winlog_process, names_sep = "_") %>%
  unnest_wider(winlog_user, names_sep = "_") %>%
  unnest_wider(log, names_sep = "_") %>%
  # Преобразуем список keywords в строку (используем sapply вместо map_chr)
  mutate(
    winlog_keywords = sapply(winlog_keywords, function(x) {
      if (is.null(x) || length(x) == 0) {
        return(NA_character_)
      } else {
        return(paste(x, collapse = ", "))
      }
    })
  ) %>%
  # Добавляем ключевые колонки из winlog_event_data
  mutate(
    event_data_ProcessId = winlog_event_data$ProcessId,
    event_data_Image = winlog_event_data$Image,
    event_data_IpAddress = winlog_event_data$IpAddress,
    event_data_TargetUserName = winlog_event_data$TargetUserName,
    event_data_LogonType = winlog_event_data$LogonType,
    event_data_SubjectUserName = winlog_event_data$SubjectUserName,
    event_data_CommandLine = winlog_event_data$CommandLine
  ) %>%
  # Удаляем исходные большие вложенные датафреймы
  select(-winlog_event_data, -winlog_user_data)

cat("После полного раскрытия:\n")
```

    После полного раскрытия:

``` r
cat("Строк:", nrow(logs_fully_unnested), "\n")
```

    Строк: 101904 

``` r
cat("Колонок:", ncol(logs_fully_unnested), "\n")
```

    Колонок: 30 

## 2. Минимизируйте количество колонок в датафрейме – уберите колонки с единственным значением параметра

``` r
logs_minimized <- logs_fully_unnested %>%
  select(where(~n_distinct(., na.rm = TRUE) > 1))

cat("После минимизации колонок:\n")
```

    После минимизации колонок:

``` r
cat("Было колонок:", ncol(logs_fully_unnested), "\n")
```

    Было колонок: 30 

``` r
cat("Стало колонок:", ncol(logs_minimized), "\n")
```

    Стало колонок: 30 

``` r
cat("Удалено колонок:", ncol(logs_fully_unnested) - ncol(logs_minimized), "\n")
```

    Удалено колонок: 0 

``` r
cat("\nОставшиеся колонки:\n")
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

## 3. Какое количество хостов представлено в данном датасете?

``` r
# Ищем колонку с именами компьютеров
if ("winlog_computer_name" %in% names(logs_minimized)) {
  unique_hosts <- unique(na.omit(logs_minimized$winlog_computer_name))
  
  cat("Количество хостов в датасете:", length(unique_hosts), "\n")
  
  cat("\nПервые 10 хостов:\n")
  print(head(unique_hosts, 10))
  
  # Распределение количества событий по хостам
  cat("\nРаспределение событий по хостам (топ-10):\n")
  host_counts <- logs_minimized %>%
    count(winlog_computer_name, sort = TRUE) %>%
    head(10)
  print(host_counts)
  
} else {
  cat("Колонка 'winlog_computer_name' не найдена.\n")
  
  # Ищем другие колонки с информацией о хостах
  possible_host_cols <- names(logs_minimized)[
    grepl("host|computer|machine", names(logs_minimized), ignore.case = TRUE)
  ]
  
  if (length(possible_host_cols) > 0) {
    cat("\nВозможные колонки с информацией о хостах:\n")
    for(col in possible_host_cols) {
      unique_count <- length(unique(na.omit(logs_minimized[[col]])))
      cat(col, ":", unique_count, "уникальных значений\n")
    }
  }
}
```

    Количество хостов в датасете: 5 

    Первые 10 хостов:
    [1] "HR001.shire.com"   "HFDC01.shire.com"  "IT001.shire.com"  
    [4] "ACCT001.shire.com" "FILE001.shire.com"

    Распределение событий по хостам (топ-10):
    # A tibble: 5 × 2
      winlog_computer_name     n
      <chr>                <int>
    1 IT001.shire.com      96296
    2 HFDC01.shire.com      2063
    3 HR001.shire.com       1730
    4 ACCT001.shire.com     1504
    5 FILE001.shire.com      311

## 4. Подготовьте датафрейм с расшифровкой Windows Event_ID, приведите типы данных к типу их значений

### Создаем таблицу расшифровки Event_ID

``` r
event_id_decoder <- tibble(
  event_id = c(4624, 4625, 4634, 4648, 4672, 4688, 4689, 4697, 4698, 4702, 4703, 4719, 4732, 4738, 4740, 4768, 4769, 4776, 5140, 5142, 5143, 5144, 5145, 5156, 5157, 5158, 10, 11, 7, 5, 4700, 4701, 4673, 4704, 4705),
  event_type = c("Успешный вход", "Неудачный вход", "Выход из системы", "Явные учетные данные использованы для входа", "Особые привилегии назначены новому входу", "Создан новый процесс", "Завершен процесс", "Установлена служба", "Создана запись планировщика заданий", "Изменен запуск задачи", "Изменение прав токена", "Изменена политика системного аудита", "Член добавлен в группу с включенной безопасностью", "Изменена учетная запись пользователя", "Заблокирована учетная запись пользователя", "Запрос билета Kerberos (TGT)", "Запрос билета службы Kerberos", "Контроллер домена попытался проверить учетные данные учетной записи", "Доступ к сетевому ресурсу", "Добавлен общий сетевой ресурс", "Удален общий сетевой ресурс", "Открыт сетевой объект", "Закрыт сетевой объект", "Брандмауэр Windows разрешил подключение", "Брандмауэр Windows заблокировал подключение", "Брандмауэр Windows разрешил привязку к порту", "Процесс создан", "Процесс завершен", "Изменение службы", "Событие аудита", "Изменение политики безопасности", "Изменение политики безопасности", "Важные привилегии назначены", "Изменение политики аудита", "Изменение политики аудита"),
  category = c("Аутентификация", "Аутентификация", "Аутентификация", "Аутентификация", "Аутентификация", "Создание процесса", "Создание процесса", "Изменение служб", "Изменение задач", "Изменение задач", "Изменение токена", "Изменение политики", "Изменение группы", "Изменение учетной записи", "Изменение учетной записи", "Аутентификация Kerberos", "Аутентификация Kerberos", "Аутентификация", "Доступ к сети", "Доступ к сети", "Доступ к сети", "Доступ к сети", "Доступ к сети", "Брандмауэр", "Брандмауэр", "Брандмауэр", "Создание процесса", "Создание процесса", "Изменение службы", "Аудит", "Политика безопасности", "Политика безопасности", "Аутентификация", "Политика безопасности", "Политика безопасности"),
  severity = c("Информация", "Предупреждение", "Информация", "Предупреждение", "Информация", "Информация", "Информация", "Предупреждение", "Предупреждение", "Предупреждение", "Предупреждение", "Информация", "Предупреждение", "Предупреждение", "Предупреждение", "Информация", "Информация", "Предупреждение", "Информация", "Информация", "Информация", "Информация", "Информация", "Информация", "Предупреждение", "Информация", "Информация", "Информация", "Предупреждение", "Информация", "Предупреждение", "Предупреждение", "Предупреждение", "Предупреждение", "Предупреждение")
)

cat("Создан датафрейм с расшифровкой для", nrow(event_id_decoder), "Event ID\n")
```

    Создан датафрейм с расшифровкой для 35 Event ID

### Объединяем с нашим датафреймом и преобразуем типы данных

``` r
# Объединяем с расшифровкой Event ID
if ("winlog_event_id" %in% names(logs_minimized)) {
  logs_decoded <- logs_minimized %>%
    rename(event_id = winlog_event_id) %>%
    left_join(event_id_decoder, by = "event_id") %>%
    mutate(
      event_type = ifelse(is.na(event_type), "Неизвестный тип события", event_type),
      category = ifelse(is.na(category), "Неизвестная категория", category),
      severity = ifelse(is.na(severity), "Неизвестно", severity)
    )
  
  cat("Добавлена расшифровка Event ID\n")
} else {
  cat("Колонка 'winlog_event_id' не найдена для расшифровки\n")
}
```

    Добавлена расшифровка Event ID

### Преобразуем типы данных

``` r
if (exists("logs_decoded")) {
  logs_decoded_typed <- logs_decoded %>%
    mutate(
      # Преобразуем даты
      timestamp = as_datetime(`@timestamp`),
      
      # Числовые поля
      across(any_of(c("event_id", "event_code", "winlog_record_id", 
                     "winlog_version", "event_data_LogonType")),
             ~as.integer(.)),
      
      # Текстовые поля
      across(any_of(c("event_action", "message", "winlog_provider_name",
                     "winlog_computer_name", "winlog_channel", "winlog_task",
                     "event_data_Image", "event_data_IpAddress",
                     "event_data_TargetUserName", "event_data_SubjectUserName",
                     "event_data_CommandLine", "winlog_keywords")),
             ~as.character(.)),
      
      # Категориальные поля
      across(any_of(c("event_type", "category", "severity", "winlog_opcode")),
             ~as.factor(.))
    )
  
  cat("Типы данных преобразованы\n")
  
  # Удаляем временную колонку @timestamp
  logs_decoded_typed <- logs_decoded_typed %>%
    select(-`@timestamp`)
  
  cat("Финальный датафрейм с расшифровкой Event ID готов\n")
  cat("Строк:", nrow(logs_decoded_typed), "\n")
  cat("Колонок:", ncol(logs_decoded_typed), "\n")
}
```

    Типы данных преобразованы
    Финальный датафрейм с расшифровкой Event ID готов
    Строк: 101904 
    Колонок: 33 

## 5. Есть ли в логе события с высоким и средним уровнем значимости? Сколько их?

``` r
if (exists("logs_decoded_typed")) {
  # Определяем высокий и средний уровень значимости на основе Event ID
  high_severity_events <- c(4625, 4673, 4697, 4702, 4719, 4740, 4768, 4776, 5157)
  medium_severity_events <- c(4624, 4634, 4648, 4672, 4688, 4689, 4698, 4703, 4732, 4738, 4769, 5140, 5142, 5143, 5144, 5145, 5156, 5158)
  
  logs_with_custom_severity <- logs_decoded_typed %>%
    mutate(
      custom_severity = case_when(
        event_id %in% high_severity_events ~ "Высокий",
        event_id %in% medium_severity_events ~ "Средний",
        TRUE ~ "Низкий"
      )
    )
  
  # Подсчитываем события по уровням значимости
  severity_counts <- logs_with_custom_severity %>%
    count(custom_severity)
  
  cat("Распределение событий по уровням значимости:\n")
  print(severity_counts)
  
  total_events <- nrow(logs_with_custom_severity)
  high_count <- severity_counts %>% 
    filter(custom_severity == "Высокий") %>% 
    pull(n) %>% 
    ifelse(length(.) > 0, ., 0)
  
  medium_count <- severity_counts %>% 
    filter(custom_severity == "Средний") %>% 
    pull(n) %>% 
    ifelse(length(.) > 0, ., 0)
  
  cat("\n=== РЕЗУЛЬТАТЫ ===\n")
  cat("Всего событий в логах:", total_events, "\n")
  cat("Событий с ВЫСОКИМ уровнем значимости:", high_count, "\n")
  cat("Событий со СРЕДНИМ уровнем значимости:", medium_count, "\n")
  cat("Событий с НИЗКИМ уровнем значимости:", total_events - high_count - medium_count, "\n")
  cat("Событий с ВЫСОКИМ и СРЕДНИМ уровнем вместе:", high_count + medium_count, "\n")
  cat("Процент событий высокого/среднего уровня:", 
      round((high_count + medium_count) / total_events * 100, 2), "%\n")
  
  # Анализируем события высокого уровня
  if (high_count > 0) {
    cat("\n=== СОБЫТИЯ ВЫСОКОГО УРОВНЯ ===\n")
    high_events <- logs_with_custom_severity %>%
      filter(custom_severity == "Высокий") %>%
      count(event_id, event_type, sort = TRUE)
    
    cat("Типы событий высокого уровня:\n")
    print(high_events)
  }
  
} else {
  cat("Датафрейм с расшифрованными событиями не найден\n")
}
```

    Распределение событий по уровням значимости:
    # A tibble: 3 × 2
      custom_severity     n
      <chr>           <int>
    1 Высокий           157
    2 Низкий          99003
    3 Средний          2744

    === РЕЗУЛЬТАТЫ ===
    Всего событий в логах: 101904 
    Событий с ВЫСОКИМ уровнем значимости: 157 
    Событий со СРЕДНИМ уровнем значимости: 2744 
    Событий с НИЗКИМ уровнем значимости: 99003 
    Событий с ВЫСОКИМ и СРЕДНИМ уровнем вместе: 2901 
    Процент событий высокого/среднего уровня: 2.85 %

    === СОБЫТИЯ ВЫСОКОГО УРОВНЯ ===
    Типы событий высокого уровня:
    # A tibble: 4 × 3
      event_id event_type                                                          n
         <int> <fct>                                                           <int>
    1     4673 Важные привилегии назначены                                       143
    2     4702 Изменен запуск задачи                                               8
    3     4768 Запрос билета Kerberos (TGT)                                        5
    4     4776 Контроллер домена попытался проверить учетные данные учетной з…     1

## Сохранение результатов

``` r
# Сохраняем датафрейм с расшифрованными событиями
if (exists("logs_decoded_typed")) {
  saveRDS(logs_decoded_typed, "windows_events_decoded.rds")
  write.csv(logs_decoded_typed, "windows_events_decoded.csv", row.names = FALSE)
  cat("\nРезультаты сохранены в файлы:\n")
  cat("- windows_events_decoded.rds\n")
  cat("- windows_events_decoded.csv\n")
}
```


    Результаты сохранены в файлы:
    - windows_events_decoded.rds
    - windows_events_decoded.csv

## Визуализация результатов

``` r
if (exists("severity_counts") && nrow(severity_counts) > 0) {
  library(ggplot2)
  
  # График распределения событий по уровням значимости
  plot1 <- ggplot(severity_counts, aes(x = custom_severity, y = n, fill = custom_severity)) +
    geom_col() +
    geom_text(aes(label = n), vjust = -0.5) +
    scale_fill_manual(values = c("Высокий" = "red", "Средний" = "orange", "Низкий" = "green")) +
    labs(
      title = "Распределение событий по уровням значимости",
      x = "Уровень значимости",
      y = "Количество событий"
    ) +
    theme_minimal()
  
  print(plot1)
  
  # График топ-10 типов событий
  if (exists("logs_decoded_typed")) {
    top_events <- logs_decoded_typed %>%
      count(event_type, sort = TRUE) %>%
      head(10)
    
    plot2 <- ggplot(top_events, aes(x = reorder(event_type, n), y = n)) +
      geom_col(fill = "steelblue") +
      coord_flip() +
      labs(
        title = "Топ-10 типов событий",
        x = "Тип события",
        y = "Количество"
      ) +
      theme_minimal()
    
    print(plot2)
  }
}
```

![](lab_6.markdown_strict_files/figure-markdown_strict/unnamed-chunk-17-1.png)

![](lab_6.markdown_strict_files/figure-markdown_strict/unnamed-chunk-17-2.png)

## Выводы

1.  Закрепили навыки исследования данных журнала Windows Active
    Directory
2.  Изучили структуру журнала системы Windows Active Directory
3.  Зекрепили практические навыки использования языка программирования R
    для обработки данных
4.  Закрепили знания основных функций обработки данных экосистемы
    tidyverse языка R
