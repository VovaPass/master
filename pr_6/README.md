# Lab6
vov41234567890@yandex.ru
2025-12-13

# Цель работы

1.  Закрепить навыки исследования данных журнала Windows Active
    Directory
2.  Изучить структуру журнала системы Windows Active Directory
3.  Зекрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Установка и скачивание библиотек, импорт файла для анализа, импорт справочника с веб-страницы


    > if (!require("dplyr")) install.packages("dplyr")
    Loading required package: dplyr

    Attaching package: ‘dplyr’

    The following objects are masked from ‘package:stats’:

        filter, lag

    The following objects are masked from ‘package:base’:

        intersect, setdiff, setequal, union
    > if (!require("jsonlite")) install.packages("jsonlite")
    Loading required package: jsonlite
    > if (!require("tidyr")) install.packages("tidyr")
    Loading required package: tidyr
    > file_path <- "/Users/vladimir/Desktop/caldera_attack_evals_round1_day1_2019-10-20201108.json"
    > log_data <- jsonlite::stream_in(file(file_path))
    opening file input connection.
     Imported 101904 records. Simplifying...
    closing file input connection.
    > log_data <- as_tibble(log_data)
    > cat("Структура данных:\n")
    Структура данных:
    > glimpse(log_data)
    Rows: 101,904
    Columns: 9
    $ `@timestamp` <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.101Z", "2019-10-20T20:11:09.052Z", …
    $ `@metadata`  <df[,4]> <data.frame[35 x 4]>
    $ event        <df[,4]> <data.frame[35 x 4]>
    $ log          <df[,1]> <data.frame[35 x 1]>
    $ message      <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity ID:\t\tS-1-5-18\n\tAccount N…
    $ winlog       <df[,16]> <data.frame[35 x 16]>
    $ ecs          <df[,1]> <data.frame[35 x 1]>
    $ host         <df[,1]> <data.frame[35 x 1]>
    $ agent        <df[,5]> <data.frame[35 x 5]>
    > cat("\n")

    > 
    > 
    > log_processed <- log_data %>%
    +     # Извлекаем основные поля
    +     mutate(
    +         timestamp = as.POSIXct(`@timestamp`, format = "%Y-%m-%dT%H:%M:%OSZ", tz = "UTC"),
    +         event_id = winlog$event_id,
    +         computer_name = winlog$computer_name,
    +         channel = winlog$channel,
    +         provider_name = winlog$provider_name,
    +         task = winlog$task,
    +         level = log$level,
    +         message_short = substr(message, 1, 100)  # Сокращенное сообщение для просмотра
    +     ) %>%
    +     # Извлекаем данные из вложенной структуры event_data
    +     mutate(
    +         process_name = winlog$event_data$ProcessName,
    +         process_id = winlog$event_data$ProcessId,
    +         subject_user = winlog$event_data$SubjectUserName,
    +         subject_domain = winlog$event_data$SubjectDomainName,
    +         target_user = winlog$event_data$TargetUserName,
    +         target_domain = winlog$event_data$TargetDomainName
    +     )
    > 
    > data <- jsonlite::stream_in(file(file_path))
    opening file input connection.
     Imported 101904 records. Simplifying...
    closing file input connection.
    > print(dim(data))  # Размерность данных
    [1] 101904      9
    > print(names(data))
    [1] "@timestamp" "@metadata"  "event"      "log"        "message"    "winlog"     "ecs"        "host"      
    [9] "agent"     
    > webpage_url <- "https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor"
    > 
    > webpage <- xml2::read_html(webpage_url)
    > event_df <- rvest::html_table(webpage)[[1]]
    > 
    > # Посмотрим на структуру справочника
    > glimpse(event_df)
    Rows: 381
    Columns: 4
    $ `Current Windows Event ID` <chr> "4618", "4649", "4719", "4765", "4766", "4794", "4897", "4964", "5124"…
    $ `Legacy Windows Event ID`  <chr> "N/A", "N/A", "612", "N/A", "N/A", "N/A", "801", "N/A", "N/A", "550", …
    $ `Potential Criticality`    <chr> "High", "High", "High", "High", "High", "High", "High", "High", "High"…
    $ `Event Summary`            <chr> "A monitored security event pattern has occurred.", "A replay attack w…
    > head(event_df)
    # A tibble: 6 × 4
      `Current Windows Event ID` `Legacy Windows Event ID` `Potential Criticality` `Event Summary`             
      <chr>                      <chr>                     <chr>                   <chr>                       
    1 4618                       N/A                       High                    A monitored security event …
    2 4649                       N/A                       High                    A replay attack was detecte…
    3 4719                       612                       High                    System audit policy was cha…
    4 4765                       N/A                       High                    SID History was added to an…
    5 4766                       N/A                       High                    An attempt to add SID Histo…
    6 4794                       N/A                       High                    An attempt was made to set …

## Преобразование данных в “аккуратный” вид


    > # Загрузим необходимые пакеты
    > library(jsonlite)
    > library(tidyverse)
    > library(rvest)
    > library(dplyr)
    > 
    > # 1. Импорт вашего JSON файла
    > file_path <- "/Users/vladimir/Desktop/caldera_attack_evals_round1_day1_2019-10-20201108.json"
    > data <- jsonlite::stream_in(file(file_path))
    opening file input connection.
     Imported 101904 records. Simplifying...
    closing file input connection.
    > 
    > # Посмотрим на структуру сырых данных
    > cat("Размер данных:", dim(data), "\n")
    Размер данных: 101904 9 
    > cat("Количество строк:", nrow(data), "\n")
    Количество строк: 101904 
    > cat("Количество столбцов:", ncol(data), "\n\n")
    Количество столбцов: 9 

    > 
    > # 2. Посмотрим на имена столбцов
    > cat("Имена столбцов в исходных данных:\n")
    Имена столбцов в исходных данных:
    > print(names(data))
    [1] "@timestamp" "@metadata"  "event"      "log"        "message"    "winlog"     "ecs"        "host"      
    [9] "agent"     
    > cat("\n")

    > 
    > # 3. Упрощенный подход: используем flatten для разворачивания вложенных структур
    > data_clean <- jsonlite::flatten(data, recursive = TRUE)
    > 
    > # 4. Посмотрим на результат
    > cat("Столбцы после flatten:\n")
    Столбцы после flatten:
    > print(names(data_clean))
      [1] "@timestamp"                                  "message"                                    
      [3] "@metadata.beat"                              "@metadata.type"                             
      [5] "@metadata.version"                           "@metadata.topic"                            
      [7] "event.created"                               "event.kind"                                 
      [9] "event.code"                                  "event.action"                               
     [11] "log.level"                                   "winlog.event_id"                            
     [13] "winlog.provider_name"                        "winlog.api"                                 
     [15] "winlog.record_id"                            "winlog.computer_name"                       
     [17] "winlog.keywords"                             "winlog.provider_guid"                       
     [19] "winlog.channel"                              "winlog.task"                                
     [21] "winlog.opcode"                               "winlog.version"                             
     [23] "winlog.activity_id"                          "winlog.event_data.SubjectDomainName"        
     [25] "winlog.event_data.TargetDomainName"          "winlog.event_data.SubjectUserSid"           
     [27] "winlog.event_data.SubjectUserName"           "winlog.event_data.TargetUserName"           
     [29] "winlog.event_data.EnabledPrivilegeList"      "winlog.event_data.TargetLogonId"            
     [31] "winlog.event_data.ProcessName"               "winlog.event_data.ProcessId"                
     [33] "winlog.event_data.SubjectLogonId"            "winlog.event_data.TargetUserSid"            
     [35] "winlog.event_data.DisabledPrivilegeList"     "winlog.event_data.ObjectServer"             
     [37] "winlog.event_data.Service"                   "winlog.event_data.PrivilegeList"            
     [39] "winlog.event_data.TargetProcessId"           "winlog.event_data.SourceProcessId"          
     [41] "winlog.event_data.SourceProcessGUID"         "winlog.event_data.SourceThreadId"           
     [43] "winlog.event_data.SourceImage"               "winlog.event_data.CallTrace"                
     [45] "winlog.event_data.TargetProcessGUID"         "winlog.event_data.TargetImage"              
     [47] "winlog.event_data.GrantedAccess"             "winlog.event_data.UtcTime"                  
     [49] "winlog.event_data.ProcessGuid"               "winlog.event_data.Image"                    
     [51] "winlog.event_data.TargetFilename"            "winlog.event_data.CreationUtcTime"          
     [53] "winlog.event_data.FileVersion"               "winlog.event_data.Company"                  
     [55] "winlog.event_data.Signed"                    "winlog.event_data.Signature"                
     [57] "winlog.event_data.OriginalFileName"          "winlog.event_data.Description"              
     [59] "winlog.event_data.Product"                   "winlog.event_data.ImageLoaded"              
     [61] "winlog.event_data.SignatureStatus"           "winlog.event_data.Hashes"                   
     [63] "winlog.event_data.Status"                    "winlog.event_data.Protocol"                 
     [65] "winlog.event_data.FilterRTID"                "winlog.event_data.LayerName"                
     [67] "winlog.event_data.LayerRTID"                 "winlog.event_data.Application"              
     [69] "winlog.event_data.SourceAddress"             "winlog.event_data.SourcePort"               
     [71] "winlog.event_data.ProcessID"                 "winlog.event_data.DestPort"                 
     [73] "winlog.event_data.RemoteMachineID"           "winlog.event_data.RemoteUserID"             
     [75] "winlog.event_data.Direction"                 "winlog.event_data.DestAddress"              
     [77] "winlog.event_data.RestrictedAdminMode"       "winlog.event_data.TransmittedServices"      
     [79] "winlog.event_data.LogonType"                 "winlog.event_data.WorkstationName"          
     [81] "winlog.event_data.LmPackageName"             "winlog.event_data.KeyLength"                
     [83] "winlog.event_data.LogonGuid"                 "winlog.event_data.ImpersonationLevel"       
     [85] "winlog.event_data.TargetOutboundDomainName"  "winlog.event_data.TargetLinkedLogonId"      
     [87] "winlog.event_data.AuthenticationPackageName" "winlog.event_data.TargetOutboundUserName"   
     [89] "winlog.event_data.VirtualAccount"            "winlog.event_data.IpAddress"                
     [91] "winlog.event_data.ElevatedToken"             "winlog.event_data.LogonProcessName"         
     [93] "winlog.event_data.IpPort"                    "winlog.event_data.GroupMembership"          
     [95] "winlog.event_data.EventIdx"                  "winlog.event_data.EventCountTotal"          
     [97] "winlog.event_data.EventType"                 "winlog.event_data.TargetObject"             
     [99] "winlog.event_data.DestinationPort"           "winlog.event_data.DestinationIsIpv6"        
    [101] "winlog.event_data.SourceIp"                  "winlog.event_data.User"                     
    [103] "winlog.event_data.SourceIsIpv6"              "winlog.event_data.DestinationIp"            
    [105] "winlog.event_data.SourceHostname"            "winlog.event_data.DestinationHostname"      
    [107] "winlog.event_data.DestinationPortName"       "winlog.event_data.Initiated"                
    [109] "winlog.event_data.param3"                    "winlog.event_data.param2"                   
    [111] "winlog.event_data.param1"                    "winlog.event_data.ParentProcessId"          
    [113] "winlog.event_data.ParentProcessGuid"         "winlog.event_data.ParentCommandLine"        
    [115] "winlog.event_data.CommandLine"               "winlog.event_data.CurrentDirectory"         
    [117] "winlog.event_data.TerminalSessionId"         "winlog.event_data.ParentImage"              
    [119] "winlog.event_data.LogonId"                   "winlog.event_data.IntegrityLevel"           
    [121] "winlog.event_data.PipeName"                  "winlog.event_data.ScriptBlockId"            
    [123] "winlog.event_data.RunspaceId"                "winlog.event_data.ContextInfo"              
    [125] "winlog.event_data.Payload"                   "winlog.event_data.MessageNumber"            
    [127] "winlog.event_data.MessageTotal"              "winlog.event_data.ScriptBlockText"          
    [129] "winlog.event_data.NewProcessId"              "winlog.event_data.NewProcessName"           
    [131] "winlog.event_data.MandatoryLabel"            "winlog.event_data.ParentProcessName"        
    [133] "winlog.event_data.TokenElevationType"        "winlog.event_data.ServiceName"              
    [135] "winlog.event_data.ServiceSid"                "winlog.event_data.TicketOptions"            
    [137] "winlog.event_data.TicketEncryptionType"      "winlog.event_data.QueryName"                
    [139] "winlog.event_data.QueryStatus"               "winlog.event_data.QueryResults"             
    [141] "winlog.event_data.SourcePortName"            "winlog.event_data.Device"                   
    [143] "winlog.event_data.Details"                   "winlog.event_data.ObjectName"               
    [145] "winlog.event_data.OldSd"                     "winlog.event_data.NewSd"                    
    [147] "winlog.event_data.ObjectType"                "winlog.event_data.HandleId"                 
    [149] "winlog.event_data.ShareName"                 "winlog.event_data.AccessList"               
    [151] "winlog.event_data.ShareLocalPath"            "winlog.event_data.AccessMask"               
    [153] "winlog.event_data.AccessReason"              "winlog.event_data.RelativeTargetName"       
    [155] "winlog.event_data.Properties"                "winlog.event_data.AdditionalInfo2"          
    [157] "winlog.event_data.OperationType"             "winlog.event_data.AdditionalInfo"           
    [159] "winlog.event_data.Path"                      "winlog.event_data.TargetHandleId"           
    [161] "winlog.event_data.SourceHandleId"            "winlog.event_data.TransactionId"            
    [163] "winlog.event_data.RestrictedSidCount"        "winlog.event_data.ResourceAttributes"       
    [165] "winlog.event_data.Binary"                    "winlog.event_data.PreviousCreationUtcTime"  
    [167] "winlog.event_data.CallerProcessId"           "winlog.event_data.CallerProcessName"        
    [169] "winlog.event_data.TargetSid"                 "winlog.event_data.TaskName"                 
    [171] "winlog.event_data.TaskContentNew"            "winlog.event_data.SourceProcessGuid"        
    [173] "winlog.event_data.TargetProcessGuid"         "winlog.event_data.StartAddress"             
    [175] "winlog.event_data.NewThreadId"               "winlog.event_data.TaskContent"              
    [177] "winlog.event_data.PackageName"               "winlog.event_data.Workstation"              
    [179] "winlog.event_data.param4"                    "winlog.event_data.param5"                   
    [181] "winlog.event_data.param7"                    "winlog.event_data.StartModule"              
    [183] "winlog.event_data.StartFunction"             "winlog.event_data.TSId"                     
    [185] "winlog.event_data.UserSid"                   "winlog.event_data.PreAuthType"              
    [187] "winlog.event_data.PreviousTime"              "winlog.event_data.NewTime"                  
    [189] "winlog.event_data.InterfaceName"             "winlog.event_data.OldProfile"               
    [191] "winlog.event_data.NewProfile"                "winlog.event_data.InterfaceGuid"            
    [193] "winlog.event_data.SettingType"               "winlog.event_data.SettingValueSize"         
    [195] "winlog.event_data.SettingValue"              "winlog.event_data.SettingValueDisplay"      
    [197] "winlog.event_data.Origin"                    "winlog.event_data.ModifyingUser"            
    [199] "winlog.event_data.Reason"                    "winlog.event_data.OldTime"                  
    [201] "winlog.event_data.DwordVal"                  "winlog.event_data.param6"                   
    [203] "winlog.event_data.ShutdownActionType"        "winlog.event_data.ShutdownEventCode"        
    [205] "winlog.event_data.ShutdownReason"            "winlog.event_data.StopTime"                 
    [207] "winlog.event_data.BootMode"                  "winlog.event_data.StartTime"                
    [209] "winlog.event_data.MajorVersion"              "winlog.event_data.MinorVersion"             
    [211] "winlog.event_data.BuildVersion"              "winlog.event_data.QfeVersion"               
    [213] "winlog.event_data.ServiceVersion"            "winlog.event_data.EnableDisableReason"      
    [215] "winlog.event_data.VsmPolicy"                 "winlog.event_data.LastBootGood"             
    [217] "winlog.event_data.LastBootId"                "winlog.event_data.BootStatusPolicy"         
    [219] "winlog.event_data.LastShutdownGood"          "winlog.event_data.BootMenuPolicy"           
    [221] "winlog.event_data.BootType"                  "winlog.event_data.LoadOptions"              
    [223] "winlog.event_data.EntryCount"                "winlog.event_data.BitlockerUserInputTime"   
    [225] "winlog.event_data.CountNew"                  "winlog.event_data.CountOld"                 
    [227] "winlog.event_data.UpdateReason"              "winlog.event_data.EnabledNew"               
    [229] "winlog.event_data.DeviceNameLength"          "winlog.event_data.DeviceName"               
    [231] "winlog.event_data.DeviceTime"                "winlog.event_data.FinalStatus"              
    [233] "winlog.event_data.DeviceVersionMajor"        "winlog.event_data.DeviceVersionMinor"       
    [235] "winlog.event_data.DriveName"                 "winlog.event_data.CorruptionActionState"    
    [237] "winlog.event_data.State"                     "winlog.event_data.MinimumPerformancePercent"
    [239] "winlog.event_data.MaximumPerformancePercent" "winlog.event_data.PerformanceImplementation"
    [241] "winlog.event_data.Group"                     "winlog.event_data.Number"                   
    [243] "winlog.event_data.IdleStateCount"            "winlog.event_data.IdleImplementation"       
    [245] "winlog.event_data.NominalFrequency"          "winlog.event_data.MinimumThrottlePercent"   
    [247] "winlog.event_data.Config"                    "winlog.event_data.IsTestConfig"             
    [249] "winlog.event_data.Default SD String:"        "winlog.event_data.AdapterSuffixName"        
    [251] "winlog.event_data.DnsServerList"             "winlog.event_data.Sent UpdateServer"        
    [253] "winlog.event_data.Ipaddress"                 "winlog.event_data.ErrorCode"                
    [255] "winlog.event_data.AdapterName"               "winlog.event_data.HostName"                 
    [257] "winlog.event_data.TimeSource"                "winlog.process.pid"                         
    [259] "winlog.process.thread.id"                    "winlog.user.domain"                         
    [261] "winlog.user.type"                            "winlog.user.identifier"                     
    [263] "winlog.user.name"                            "winlog.user_data.ProcessID"                 
    [265] "winlog.user_data.ProviderPath"               "winlog.user_data.xml_name"                  
    [267] "winlog.user_data.ProviderName"               "winlog.user_data.Code"                      
    [269] "winlog.user_data.HostProcess"                "winlog.user_data.Id"                        
    [271] "winlog.user_data.PossibleCause"              "winlog.user_data.ClientProcessId"           
    [273] "winlog.user_data.Component"                  "winlog.user_data.Operation"                 
    [275] "winlog.user_data.User"                       "winlog.user_data.ResultCode"                
    [277] "winlog.user_data.ClientMachine"              "winlog.user_data.SessionID"                 
    [279] "winlog.user_data.ESS"                        "winlog.user_data.CONSUMER"                  
    [281] "winlog.user_data.Namespace"                  "winlog.user_data.Processid"                 
    [283] "winlog.user_data.NamespaceName"              "winlog.user_data.Query"                     
    [285] "winlog.user_data.Provider"                   "winlog.user_data.queryid"                   
    [287] "winlog.user_data.Session"                    "winlog.user_data.Reason"                    
    [289] "winlog.user_data.Address"                    "winlog.user_data.messageName"               
    [291] "winlog.user_data.ListenerName"               "winlog.user_data.Class"                     
    [293] "winlog.user_data.listenerName"               "ecs.version"                                
    [295] "host.name"                                   "agent.ephemeral_id"                         
    [297] "agent.hostname"                              "agent.id"                                   
    [299] "agent.version"                               "agent.type"                                 
    > cat("\n")

    > 
    > # 5. Преобразуем типы данных (упрощенный вариант)
    > # Функция для безопасного преобразования hex в numeric
    > safe_hex_to_numeric <- function(x) {
    +     if (is.character(x)) {
    +         # Заменяем пустые строки на NA
    +         x[x == ""] <- NA
    +         
    +         # Пытаемся преобразовать hex значения
    +         result <- sapply(x, function(val) {
    +             if (is.na(val)) return(NA)
    +             
    +             # Если значение начинается с 0x, пробуем преобразовать
    +             if (grepl("^0x", val, ignore.case = TRUE)) {
    +                 # Убираем 0x префикс
    +                 hex_val <- sub("^0x", "", val, ignore.case = TRUE)
    +                 # Пробуем преобразовать
    +                 tryCatch({
    +                     # strtoi конвертирует hex строки в integer
    +                     as.numeric(strtoi(hex_val, base = 16))
    +                 }, error = function(e) {
    +                     # Если ошибка, возвращаем исходное значение
    +                     return(val)
    +                 })
    +             } else {
    +                 # Если не hex, пробуем преобразовать в число
    +                 num_val <- suppressWarnings(as.numeric(val))
    +                 if (is.na(num_val)) {
    +                     return(val)  # Оставляем как есть если не число
    +                 } else {
    +                     return(num_val)
    +                 }
    +             }
    +         })
    +         
    +         return(unlist(result))
    +     }
    +     return(x)
    + }
    > 
    > # 6. Применяем преобразование только к столбцам, которые содержат Id или hex значения
    > # Найдем такие столбцы
    > id_cols <- grep("id|Id|ID", names(data_clean), value = TRUE, ignore.case = TRUE)
    > cat("Столбцы, содержащие 'id':\n")
    Столбцы, содержащие 'id':
    > print(id_cols)
     [1] "winlog.event_id"                       "winlog.provider_name"                 
     [3] "winlog.record_id"                      "winlog.provider_guid"                 
     [5] "winlog.activity_id"                    "winlog.event_data.SubjectUserSid"     
     [7] "winlog.event_data.TargetLogonId"       "winlog.event_data.ProcessId"          
     [9] "winlog.event_data.SubjectLogonId"      "winlog.event_data.TargetUserSid"      
    [11] "winlog.event_data.TargetProcessId"     "winlog.event_data.SourceProcessId"    
    [13] "winlog.event_data.SourceProcessGUID"   "winlog.event_data.SourceThreadId"     
    [15] "winlog.event_data.TargetProcessGUID"   "winlog.event_data.ProcessGuid"        
    [17] "winlog.event_data.FilterRTID"          "winlog.event_data.LayerRTID"          
    [19] "winlog.event_data.ProcessID"           "winlog.event_data.RemoteMachineID"    
    [21] "winlog.event_data.RemoteUserID"        "winlog.event_data.LogonGuid"          
    [23] "winlog.event_data.TargetLinkedLogonId" "winlog.event_data.EventIdx"           
    [25] "winlog.event_data.ParentProcessId"     "winlog.event_data.ParentProcessGuid"  
    [27] "winlog.event_data.TerminalSessionId"   "winlog.event_data.LogonId"            
    [29] "winlog.event_data.ScriptBlockId"       "winlog.event_data.RunspaceId"         
    [31] "winlog.event_data.NewProcessId"        "winlog.event_data.ServiceSid"         
    [33] "winlog.event_data.HandleId"            "winlog.event_data.TargetHandleId"     
    [35] "winlog.event_data.SourceHandleId"      "winlog.event_data.TransactionId"      
    [37] "winlog.event_data.RestrictedSidCount"  "winlog.event_data.CallerProcessId"    
    [39] "winlog.event_data.TargetSid"           "winlog.event_data.SourceProcessGuid"  
    [41] "winlog.event_data.TargetProcessGuid"   "winlog.event_data.NewThreadId"        
    [43] "winlog.event_data.TSId"                "winlog.event_data.UserSid"            
    [45] "winlog.event_data.InterfaceGuid"       "winlog.event_data.LastBootId"         
    [47] "winlog.event_data.IdleStateCount"      "winlog.event_data.IdleImplementation" 
    [49] "winlog.process.pid"                    "winlog.process.thread.id"             
    [51] "winlog.user.identifier"                "winlog.user_data.ProcessID"           
    [53] "winlog.user_data.ProviderPath"         "winlog.user_data.ProviderName"        
    [55] "winlog.user_data.Id"                   "winlog.user_data.ClientProcessId"     
    [57] "winlog.user_data.SessionID"            "winlog.user_data.Processid"           
    [59] "winlog.user_data.Provider"             "winlog.user_data.queryid"             
    [61] "agent.ephemeral_id"                    "agent.id"                             
    > cat("\n")

    > 
    > # 7. Преобразуем временные метки
    > # Найдем все timestamp столбцы
    > timestamp_cols <- grep("timestamp|time|created", names(data_clean), value = TRUE, ignore.case = TRUE)
    > 
    > for (col in timestamp_cols) {
    +     if (is.character(data_clean[[col]])) {
    +         cat("Преобразование столбца:", col, "\n")
    +         # Пробуем разные форматы дат
    +         data_clean[[paste0(col, "_datetime")]] <- tryCatch({
    +             as.POSIXct(data_clean[[col]], format = "%Y-%m-%dT%H:%M:%OSZ", tz = "UTC")
    +         }, error = function(e) {
    +             tryCatch({
    +                 as.POSIXct(data_clean[[col]], format = "%Y-%m-%d %H:%M:%S", tz = "UTC")
    +             }, error = function(e) {
    +                 NA
    +             })
    +         })
    +     }
    + }
    Преобразование столбца: @timestamp 
    Преобразование столбца: event.created 
    Преобразование столбца: winlog.event_data.UtcTime 
    Преобразование столбца: winlog.event_data.CreationUtcTime 
    Преобразование столбца: winlog.event_data.PreviousCreationUtcTime 
    Преобразование столбца: winlog.event_data.PreviousTime 
    Преобразование столбца: winlog.event_data.NewTime 
    Преобразование столбца: winlog.event_data.OldTime 
    Преобразование столбца: winlog.event_data.StopTime 
    Преобразование столбца: winlog.event_data.StartTime 
    Преобразование столбца: winlog.event_data.BitlockerUserInputTime 
    Преобразование столбца: winlog.event_data.DeviceTime 
    Преобразование столбца: winlog.event_data.TimeSource 
    > 
    > # 8. Простой просмотр данных без сложных преобразований
    > cat("\n=== ПРОСМОТР ДАННЫХ ===\n")

    === ПРОСМОТР ДАННЫХ ===
    > 
    > # Способ 1: Используем head для просмотра первых строк
    > cat("\nПервые 3 строки данных:\n")

    Первые 3 строки данных:
    > print(head(data_clean, 3))
                    @timestamp
    1 2019-10-20T20:11:06.937Z
    2 2019-10-20T20:11:07.101Z
    3 2019-10-20T20:11:09.052Z
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             message
    1                                                                                                                                                                                                                                                                                                                                                                    A token right was adjusted.\n\nSubject:\n\tSecurity ID:\t\tS-1-5-18\n\tAccount Name:\t\tHR001$\n\tAccount Domain:\t\tshire\n\tLogon ID:\t\t0x3E7\n\nTarget Account:\n\tSecurity ID:\t\tS-1-5-18\n\tAccount Name:\t\tHR001$\n\tAccount Domain:\t\tshire\n\tLogon ID:\t\t0x3E7\n\nProcess Information:\n\tProcess ID:\t\t0x804\n\tProcess Name:\t\tC:\\Windows\\System32\\svchost.exe\n\nEnabled Privileges:\n\t\t\tSeTakeOwnershipPrivilege\n\nDisabled Privileges:\n\t\t\t-
    2                                                                                                                                                                                                                                                                                                                                                                                                                                              A privileged service was called.\n\nSubject:\n\tSecurity ID:\t\tS-1-5-19\n\tAccount Name:\t\tLOCAL SERVICE\n\tAccount Domain:\t\tNT AUTHORITY\n\tLogon ID:\t\t0x3E5\n\nService:\n\tServer:\tSecurity\n\tService Name:\t-\n\nProcess:\n\tProcess ID:\t0x494\n\tProcess Name:\tC:\\Windows\\System32\\svchost.exe\n\nService Request Information:\n\tPrivileges:\t\tSeProfileSingleProcessPrivilege
    3 Process accessed:\nRuleName: \nUtcTime: 2019-10-20 20:11:09.052\nSourceProcessGUID: {a158f72c-afec-5dac-0000-001030640200}\nSourceProcessId: 3556\nSourceThreadId: 3688\nSourceImage: C:\\Windows\\System32\\svchost.exe\nTargetProcessGUID: {a158f72c-afeb-5dac-0000-001082220200}\nTargetProcessId: 3108\nTargetImage: C:\\Program Files\\Amazon\\Ec2ConfigService\\Ec2Config.exe\nGrantedAccess: 0x1400\nCallTrace: C:\\Windows\\SYSTEM32\\ntdll.dll+9c524|C:\\Windows\\System32\\KERNELBASE.dll+2730e|c:\\windows\\system32\\rasmans.dll+3751b|c:\\windows\\system32\\rasmans.dll+36fad|c:\\windows\\system32\\rasmans.dll+10ed8|c:\\windows\\system32\\rasmans.dll+33cd5|C:\\Windows\\System32\\svchost.exe+314c|C:\\Windows\\System32\\sechost.dll+2de2|C:\\Windows\\System32\\KERNEL32.DLL+17944|C:\\Windows\\SYSTEM32\\ntdll.dll+6ce71
      @metadata.beat @metadata.type @metadata.version @metadata.topic            event.created event.kind
    1     winlogbeat           _doc             7.4.0      winlogbeat 2019-10-20T20:11:09.988Z      event
    2     winlogbeat           _doc             7.4.0      winlogbeat 2019-10-20T20:11:09.988Z      event
    3     winlogbeat           _doc             7.4.0      winlogbeat 2019-10-20T20:11:11.995Z      event
      event.code                           event.action   log.level winlog.event_id
    1       4703            Token Right Adjusted Events information            4703
    2       4673                Sensitive Privilege Use information            4673
    3         10 Process accessed (rule: ProcessAccess) information              10
                     winlog.provider_name  winlog.api winlog.record_id winlog.computer_name winlog.keywords
    1 Microsoft-Windows-Security-Auditing wineventlog            50588      HR001.shire.com   Audit Success
    2 Microsoft-Windows-Security-Auditing wineventlog           104875     HFDC01.shire.com   Audit Failure
    3            Microsoft-Windows-Sysmon wineventlog           226649      IT001.shire.com            NULL
                        winlog.provider_guid                       winlog.channel
    1 {54849625-5478-4994-a5ba-3e3b0328c30d}                             security
    2 {54849625-5478-4994-a5ba-3e3b0328c30d}                             Security
    3 {5770385f-c22a-43e0-bf4c-06f5698ffbd9} Microsoft-Windows-Sysmon/Operational
                                 winlog.task winlog.opcode winlog.version winlog.activity_id
    1            Token Right Adjusted Events          Info             NA               <NA>
    2                Sensitive Privilege Use          Info             NA               <NA>
    3 Process accessed (rule: ProcessAccess)          Info              3               <NA>
      winlog.event_data.SubjectDomainName winlog.event_data.TargetDomainName winlog.event_data.SubjectUserSid
    1                               shire                              shire                         S-1-5-18
    2                        NT AUTHORITY                               <NA>                         S-1-5-19
    3                                <NA>                               <NA>                             <NA>
      winlog.event_data.SubjectUserName winlog.event_data.TargetUserName
    1                            HR001$                           HR001$
    2                     LOCAL SERVICE                             <NA>
    3                              <NA>                             <NA>
      winlog.event_data.EnabledPrivilegeList winlog.event_data.TargetLogonId
    1               SeTakeOwnershipPrivilege                           0x3e7
    2                                   <NA>                            <NA>
    3                                   <NA>                            <NA>
           winlog.event_data.ProcessName winlog.event_data.ProcessId winlog.event_data.SubjectLogonId
    1 C:\\Windows\\System32\\svchost.exe                       0x804                            0x3e7
    2 C:\\Windows\\System32\\svchost.exe                       0x494                            0x3e5
    3                               <NA>                        <NA>                             <NA>
      winlog.event_data.TargetUserSid winlog.event_data.DisabledPrivilegeList winlog.event_data.ObjectServer
    1                        S-1-5-18                                       -                           <NA>
    2                            <NA>                                    <NA>                       Security
    3                            <NA>                                    <NA>                           <NA>
      winlog.event_data.Service winlog.event_data.PrivilegeList winlog.event_data.TargetProcessId
    1                      <NA>                            <NA>                              <NA>
    2                         - SeProfileSingleProcessPrivilege                              <NA>
    3                      <NA>                            <NA>                              3108
      winlog.event_data.SourceProcessId    winlog.event_data.SourceProcessGUID
    1                              <NA>                                   <NA>
    2                              <NA>                                   <NA>
    3                              3556 {a158f72c-afec-5dac-0000-001030640200}
      winlog.event_data.SourceThreadId      winlog.event_data.SourceImage
    1                             <NA>                               <NA>
    2                             <NA>                               <NA>
    3                             3688 C:\\Windows\\System32\\svchost.exe
                                                                                                                                                                                                                                                                                                                                                                                                  winlog.event_data.CallTrace
    1                                                                                                                                                                                                                                                                                                                                                                                                                    <NA>
    2                                                                                                                                                                                                                                                                                                                                                                                                                    <NA>
    3 C:\\Windows\\SYSTEM32\\ntdll.dll+9c524|C:\\Windows\\System32\\KERNELBASE.dll+2730e|c:\\windows\\system32\\rasmans.dll+3751b|c:\\windows\\system32\\rasmans.dll+36fad|c:\\windows\\system32\\rasmans.dll+10ed8|c:\\windows\\system32\\rasmans.dll+33cd5|C:\\Windows\\System32\\svchost.exe+314c|C:\\Windows\\System32\\sechost.dll+2de2|C:\\Windows\\System32\\KERNEL32.DLL+17944|C:\\Windows\\SYSTEM32\\ntdll.dll+6ce71
         winlog.event_data.TargetProcessGUID                              winlog.event_data.TargetImage
    1                                   <NA>                                                       <NA>
    2                                   <NA>                                                       <NA>
    3 {a158f72c-afeb-5dac-0000-001082220200} C:\\Program Files\\Amazon\\Ec2ConfigService\\Ec2Config.exe
      winlog.event_data.GrantedAccess winlog.event_data.UtcTime winlog.event_data.ProcessGuid
    1                            <NA>                      <NA>                          <NA>
    2                            <NA>                      <NA>                          <NA>
    3                          0x1400   2019-10-20 20:11:09.052                          <NA>
      winlog.event_data.Image winlog.event_data.TargetFilename winlog.event_data.CreationUtcTime
    1                    <NA>                             <NA>                              <NA>
    2                    <NA>                             <NA>                              <NA>
    3                    <NA>                             <NA>                              <NA>
      winlog.event_data.FileVersion winlog.event_data.Company winlog.event_data.Signed
    1                          <NA>                      <NA>                     <NA>
    2                          <NA>                      <NA>                     <NA>
    3                          <NA>                      <NA>                     <NA>
      winlog.event_data.Signature winlog.event_data.OriginalFileName winlog.event_data.Description
    1                        <NA>                               <NA>                          <NA>
    2                        <NA>                               <NA>                          <NA>
    3                        <NA>                               <NA>                          <NA>
      winlog.event_data.Product winlog.event_data.ImageLoaded winlog.event_data.SignatureStatus
    1                      <NA>                          <NA>                              <NA>
    2                      <NA>                          <NA>                              <NA>
    3                      <NA>                          <NA>                              <NA>
      winlog.event_data.Hashes winlog.event_data.Status winlog.event_data.Protocol
    1                     <NA>                     <NA>                       <NA>
    2                     <NA>                     <NA>                       <NA>
    3                     <NA>                     <NA>                       <NA>
      winlog.event_data.FilterRTID winlog.event_data.LayerName winlog.event_data.LayerRTID
    1                         <NA>                        <NA>                        <NA>
    2                         <NA>                        <NA>                        <NA>
    3                         <NA>                        <NA>                        <NA>
      winlog.event_data.Application winlog.event_data.SourceAddress winlog.event_data.SourcePort
    1                          <NA>                            <NA>                         <NA>
    2                          <NA>                            <NA>                         <NA>
    3                          <NA>                            <NA>                         <NA>
      winlog.event_data.ProcessID winlog.event_data.DestPort winlog.event_data.RemoteMachineID
    1                        <NA>                       <NA>                              <NA>
    2                        <NA>                       <NA>                              <NA>
    3                        <NA>                       <NA>                              <NA>
      winlog.event_data.RemoteUserID winlog.event_data.Direction winlog.event_data.DestAddress
    1                           <NA>                        <NA>                          <NA>
    2                           <NA>                        <NA>                          <NA>
    3                           <NA>                        <NA>                          <NA>
      winlog.event_data.RestrictedAdminMode winlog.event_data.TransmittedServices winlog.event_data.LogonType
    1                                  <NA>                                  <NA>                        <NA>
    2                                  <NA>                                  <NA>                        <NA>
    3                                  <NA>                                  <NA>                        <NA>
      winlog.event_data.WorkstationName winlog.event_data.LmPackageName winlog.event_data.KeyLength
    1                              <NA>                            <NA>                        <NA>
    2                              <NA>                            <NA>                        <NA>
    3                              <NA>                            <NA>                        <NA>
      winlog.event_data.LogonGuid winlog.event_data.ImpersonationLevel
    1                        <NA>                                 <NA>
    2                        <NA>                                 <NA>
    3                        <NA>                                 <NA>
      winlog.event_data.TargetOutboundDomainName winlog.event_data.TargetLinkedLogonId
    1                                       <NA>                                  <NA>
    2                                       <NA>                                  <NA>
    3                                       <NA>                                  <NA>
      winlog.event_data.AuthenticationPackageName winlog.event_data.TargetOutboundUserName
    1                                        <NA>                                     <NA>
    2                                        <NA>                                     <NA>
    3                                        <NA>                                     <NA>
      winlog.event_data.VirtualAccount winlog.event_data.IpAddress winlog.event_data.ElevatedToken
    1                             <NA>                        <NA>                            <NA>
    2                             <NA>                        <NA>                            <NA>
    3                             <NA>                        <NA>                            <NA>
      winlog.event_data.LogonProcessName winlog.event_data.IpPort winlog.event_data.GroupMembership
    1                               <NA>                     <NA>                              <NA>
    2                               <NA>                     <NA>                              <NA>
    3                               <NA>                     <NA>                              <NA>
      winlog.event_data.EventIdx winlog.event_data.EventCountTotal winlog.event_data.EventType
    1                       <NA>                              <NA>                        <NA>
    2                       <NA>                              <NA>                        <NA>
    3                       <NA>                              <NA>                        <NA>
      winlog.event_data.TargetObject winlog.event_data.DestinationPort winlog.event_data.DestinationIsIpv6
    1                           <NA>                              <NA>                                <NA>
    2                           <NA>                              <NA>                                <NA>
    3                           <NA>                              <NA>                                <NA>
      winlog.event_data.SourceIp winlog.event_data.User winlog.event_data.SourceIsIpv6
    1                       <NA>                   <NA>                           <NA>
    2                       <NA>                   <NA>                           <NA>
    3                       <NA>                   <NA>                           <NA>
      winlog.event_data.DestinationIp winlog.event_data.SourceHostname winlog.event_data.DestinationHostname
    1                            <NA>                             <NA>                                  <NA>
    2                            <NA>                             <NA>                                  <NA>
    3                            <NA>                             <NA>                                  <NA>
      winlog.event_data.DestinationPortName winlog.event_data.Initiated winlog.event_data.param3
    1                                  <NA>                        <NA>                     <NA>
    2                                  <NA>                        <NA>                     <NA>
    3                                  <NA>                        <NA>                     <NA>
      winlog.event_data.param2 winlog.event_data.param1 winlog.event_data.ParentProcessId
    1                     <NA>                     <NA>                              <NA>
    2                     <NA>                     <NA>                              <NA>
    3                     <NA>                     <NA>                              <NA>
      winlog.event_data.ParentProcessGuid winlog.event_data.ParentCommandLine winlog.event_data.CommandLine
    1                                <NA>                                <NA>                          <NA>
    2                                <NA>                                <NA>                          <NA>
    3                                <NA>                                <NA>                          <NA>
      winlog.event_data.CurrentDirectory winlog.event_data.TerminalSessionId winlog.event_data.ParentImage
    1                               <NA>                                <NA>                          <NA>
    2                               <NA>                                <NA>                          <NA>
    3                               <NA>                                <NA>                          <NA>
      winlog.event_data.LogonId winlog.event_data.IntegrityLevel winlog.event_data.PipeName
    1                      <NA>                             <NA>                       <NA>
    2                      <NA>                             <NA>                       <NA>
    3                      <NA>                             <NA>                       <NA>
      winlog.event_data.ScriptBlockId winlog.event_data.RunspaceId winlog.event_data.ContextInfo
    1                            <NA>                         <NA>                          <NA>
    2                            <NA>                         <NA>                          <NA>
    3                            <NA>                         <NA>                          <NA>
      winlog.event_data.Payload winlog.event_data.MessageNumber winlog.event_data.MessageTotal
    1                      <NA>                            <NA>                           <NA>
    2                      <NA>                            <NA>                           <NA>
    3                      <NA>                            <NA>                           <NA>
      winlog.event_data.ScriptBlockText winlog.event_data.NewProcessId winlog.event_data.NewProcessName
    1                              <NA>                           <NA>                             <NA>
    2                              <NA>                           <NA>                             <NA>
    3                              <NA>                           <NA>                             <NA>
      winlog.event_data.MandatoryLabel winlog.event_data.ParentProcessName
    1                             <NA>                                <NA>
    2                             <NA>                                <NA>
    3                             <NA>                                <NA>
      winlog.event_data.TokenElevationType winlog.event_data.ServiceName winlog.event_data.ServiceSid
    1                                 <NA>                          <NA>                         <NA>
    2                                 <NA>                          <NA>                         <NA>
    3                                 <NA>                          <NA>                         <NA>
      winlog.event_data.TicketOptions winlog.event_data.TicketEncryptionType winlog.event_data.QueryName
    1                            <NA>                                   <NA>                        <NA>
    2                            <NA>                                   <NA>                        <NA>
    3                            <NA>                                   <NA>                        <NA>
      winlog.event_data.QueryStatus winlog.event_data.QueryResults winlog.event_data.SourcePortName
    1                          <NA>                           <NA>                             <NA>
    2                          <NA>                           <NA>                             <NA>
    3                          <NA>                           <NA>                             <NA>
      winlog.event_data.Device winlog.event_data.Details winlog.event_data.ObjectName winlog.event_data.OldSd
    1                     <NA>                      <NA>                         <NA>                    <NA>
    2                     <NA>                      <NA>                         <NA>                    <NA>
    3                     <NA>                      <NA>                         <NA>                    <NA>
      winlog.event_data.NewSd winlog.event_data.ObjectType winlog.event_data.HandleId
    1                    <NA>                         <NA>                       <NA>
    2                    <NA>                         <NA>                       <NA>
    3                    <NA>                         <NA>                       <NA>
      winlog.event_data.ShareName winlog.event_data.AccessList winlog.event_data.ShareLocalPath
    1                        <NA>                         <NA>                             <NA>
    2                        <NA>                         <NA>                             <NA>
    3                        <NA>                         <NA>                             <NA>
      winlog.event_data.AccessMask winlog.event_data.AccessReason winlog.event_data.RelativeTargetName
    1                         <NA>                           <NA>                                 <NA>
    2                         <NA>                           <NA>                                 <NA>
    3                         <NA>                           <NA>                                 <NA>
      winlog.event_data.Properties winlog.event_data.AdditionalInfo2 winlog.event_data.OperationType
    1                         <NA>                              <NA>                            <NA>
    2                         <NA>                              <NA>                            <NA>
    3                         <NA>                              <NA>                            <NA>
      winlog.event_data.AdditionalInfo winlog.event_data.Path winlog.event_data.TargetHandleId
    1                             <NA>                   <NA>                             <NA>
    2                             <NA>                   <NA>                             <NA>
    3                             <NA>                   <NA>                             <NA>
      winlog.event_data.SourceHandleId winlog.event_data.TransactionId winlog.event_data.RestrictedSidCount
    1                             <NA>                            <NA>                                 <NA>
    2                             <NA>                            <NA>                                 <NA>
    3                             <NA>                            <NA>                                 <NA>
      winlog.event_data.ResourceAttributes winlog.event_data.Binary winlog.event_data.PreviousCreationUtcTime
    1                                 <NA>                     <NA>                                      <NA>
    2                                 <NA>                     <NA>                                      <NA>
    3                                 <NA>                     <NA>                                      <NA>
      winlog.event_data.CallerProcessId winlog.event_data.CallerProcessName winlog.event_data.TargetSid
    1                              <NA>                                <NA>                        <NA>
    2                              <NA>                                <NA>                        <NA>
    3                              <NA>                                <NA>                        <NA>
      winlog.event_data.TaskName winlog.event_data.TaskContentNew winlog.event_data.SourceProcessGuid
    1                       <NA>                             <NA>                                <NA>
    2                       <NA>                             <NA>                                <NA>
    3                       <NA>                             <NA>                                <NA>
      winlog.event_data.TargetProcessGuid winlog.event_data.StartAddress winlog.event_data.NewThreadId
    1                                <NA>                           <NA>                          <NA>
    2                                <NA>                           <NA>                          <NA>
    3                                <NA>                           <NA>                          <NA>
      winlog.event_data.TaskContent winlog.event_data.PackageName winlog.event_data.Workstation
    1                          <NA>                          <NA>                          <NA>
    2                          <NA>                          <NA>                          <NA>
    3                          <NA>                          <NA>                          <NA>
      winlog.event_data.param4 winlog.event_data.param5 winlog.event_data.param7 winlog.event_data.StartModule
    1                     <NA>                     <NA>                     <NA>                          <NA>
    2                     <NA>                     <NA>                     <NA>                          <NA>
    3                     <NA>                     <NA>                     <NA>                          <NA>
      winlog.event_data.StartFunction winlog.event_data.TSId winlog.event_data.UserSid
    1                            <NA>                   <NA>                      <NA>
    2                            <NA>                   <NA>                      <NA>
    3                            <NA>                   <NA>                      <NA>
      winlog.event_data.PreAuthType winlog.event_data.PreviousTime winlog.event_data.NewTime
    1                          <NA>                           <NA>                      <NA>
    2                          <NA>                           <NA>                      <NA>
    3                          <NA>                           <NA>                      <NA>
      winlog.event_data.InterfaceName winlog.event_data.OldProfile winlog.event_data.NewProfile
    1                            <NA>                         <NA>                         <NA>
    2                            <NA>                         <NA>                         <NA>
    3                            <NA>                         <NA>                         <NA>
      winlog.event_data.InterfaceGuid winlog.event_data.SettingType winlog.event_data.SettingValueSize
    1                            <NA>                          <NA>                               <NA>
    2                            <NA>                          <NA>                               <NA>
    3                            <NA>                          <NA>                               <NA>
      winlog.event_data.SettingValue winlog.event_data.SettingValueDisplay winlog.event_data.Origin
    1                           <NA>                                  <NA>                     <NA>
    2                           <NA>                                  <NA>                     <NA>
    3                           <NA>                                  <NA>                     <NA>
      winlog.event_data.ModifyingUser winlog.event_data.Reason winlog.event_data.OldTime
    1                            <NA>                     <NA>                      <NA>
    2                            <NA>                     <NA>                      <NA>
    3                            <NA>                     <NA>                      <NA>
      winlog.event_data.DwordVal winlog.event_data.param6 winlog.event_data.ShutdownActionType
    1                       <NA>                     <NA>                                 <NA>
    2                       <NA>                     <NA>                                 <NA>
    3                       <NA>                     <NA>                                 <NA>
      winlog.event_data.ShutdownEventCode winlog.event_data.ShutdownReason winlog.event_data.StopTime
    1                                <NA>                             <NA>                       <NA>
    2                                <NA>                             <NA>                       <NA>
    3                                <NA>                             <NA>                       <NA>
      winlog.event_data.BootMode winlog.event_data.StartTime winlog.event_data.MajorVersion
    1                       <NA>                        <NA>                           <NA>
    2                       <NA>                        <NA>                           <NA>
    3                       <NA>                        <NA>                           <NA>
      winlog.event_data.MinorVersion winlog.event_data.BuildVersion winlog.event_data.QfeVersion
    1                           <NA>                           <NA>                         <NA>
    2                           <NA>                           <NA>                         <NA>
    3                           <NA>                           <NA>                         <NA>
      winlog.event_data.ServiceVersion winlog.event_data.EnableDisableReason winlog.event_data.VsmPolicy
    1                             <NA>                                  <NA>                        <NA>
    2                             <NA>                                  <NA>                        <NA>
    3                             <NA>                                  <NA>                        <NA>
      winlog.event_data.LastBootGood winlog.event_data.LastBootId winlog.event_data.BootStatusPolicy
    1                           <NA>                         <NA>                               <NA>
    2                           <NA>                         <NA>                               <NA>
    3                           <NA>                         <NA>                               <NA>
      winlog.event_data.LastShutdownGood winlog.event_data.BootMenuPolicy winlog.event_data.BootType
    1                               <NA>                             <NA>                       <NA>
    2                               <NA>                             <NA>                       <NA>
    3                               <NA>                             <NA>                       <NA>
      winlog.event_data.LoadOptions winlog.event_data.EntryCount winlog.event_data.BitlockerUserInputTime
    1                          <NA>                         <NA>                                     <NA>
    2                          <NA>                         <NA>                                     <NA>
    3                          <NA>                         <NA>                                     <NA>
      winlog.event_data.CountNew winlog.event_data.CountOld winlog.event_data.UpdateReason
    1                       <NA>                       <NA>                           <NA>
    2                       <NA>                       <NA>                           <NA>
    3                       <NA>                       <NA>                           <NA>
      winlog.event_data.EnabledNew winlog.event_data.DeviceNameLength winlog.event_data.DeviceName
    1                         <NA>                               <NA>                         <NA>
    2                         <NA>                               <NA>                         <NA>
    3                         <NA>                               <NA>                         <NA>
      winlog.event_data.DeviceTime winlog.event_data.FinalStatus winlog.event_data.DeviceVersionMajor
    1                         <NA>                          <NA>                                 <NA>
    2                         <NA>                          <NA>                                 <NA>
    3                         <NA>                          <NA>                                 <NA>
      winlog.event_data.DeviceVersionMinor winlog.event_data.DriveName winlog.event_data.CorruptionActionState
    1                                 <NA>                        <NA>                                    <NA>
    2                                 <NA>                        <NA>                                    <NA>
    3                                 <NA>                        <NA>                                    <NA>
      winlog.event_data.State winlog.event_data.MinimumPerformancePercent
    1                    <NA>                                        <NA>
    2                    <NA>                                        <NA>
    3                    <NA>                                        <NA>
      winlog.event_data.MaximumPerformancePercent winlog.event_data.PerformanceImplementation
    1                                        <NA>                                        <NA>
    2                                        <NA>                                        <NA>
    3                                        <NA>                                        <NA>
      winlog.event_data.Group winlog.event_data.Number winlog.event_data.IdleStateCount
    1                    <NA>                     <NA>                             <NA>
    2                    <NA>                     <NA>                             <NA>
    3                    <NA>                     <NA>                             <NA>
      winlog.event_data.IdleImplementation winlog.event_data.NominalFrequency
    1                                 <NA>                               <NA>
    2                                 <NA>                               <NA>
    3                                 <NA>                               <NA>
      winlog.event_data.MinimumThrottlePercent winlog.event_data.Config winlog.event_data.IsTestConfig
    1                                     <NA>                     <NA>                           <NA>
    2                                     <NA>                     <NA>                           <NA>
    3                                     <NA>                     <NA>                           <NA>
      winlog.event_data.Default SD String: winlog.event_data.AdapterSuffixName winlog.event_data.DnsServerList
    1                                 <NA>                                <NA>                            <NA>
    2                                 <NA>                                <NA>                            <NA>
    3                                 <NA>                                <NA>                            <NA>
      winlog.event_data.Sent UpdateServer winlog.event_data.Ipaddress winlog.event_data.ErrorCode
    1                                <NA>                        <NA>                        <NA>
    2                                <NA>                        <NA>                        <NA>
    3                                <NA>                        <NA>                        <NA>
      winlog.event_data.AdapterName winlog.event_data.HostName winlog.event_data.TimeSource winlog.process.pid
    1                          <NA>                       <NA>                         <NA>                  4
    2                          <NA>                       <NA>                         <NA>                  4
    3                          <NA>                       <NA>                         <NA>               3220
      winlog.process.thread.id winlog.user.domain winlog.user.type winlog.user.identifier winlog.user.name
    1                     4108               <NA>             <NA>                   <NA>             <NA>
    2                     5144               <NA>             <NA>                   <NA>             <NA>
    3                     4972       NT AUTHORITY             User               S-1-5-18           SYSTEM
      winlog.user_data.ProcessID winlog.user_data.ProviderPath winlog.user_data.xml_name
    1                       <NA>                          <NA>                      <NA>
    2                       <NA>                          <NA>                      <NA>
    3                       <NA>                          <NA>                      <NA>
      winlog.user_data.ProviderName winlog.user_data.Code winlog.user_data.HostProcess winlog.user_data.Id
    1                          <NA>                  <NA>                         <NA>                <NA>
    2                          <NA>                  <NA>                         <NA>                <NA>
    3                          <NA>                  <NA>                         <NA>                <NA>
      winlog.user_data.PossibleCause winlog.user_data.ClientProcessId winlog.user_data.Component
    1                           <NA>                             <NA>                       <NA>
    2                           <NA>                             <NA>                       <NA>
    3                           <NA>                             <NA>                       <NA>
      winlog.user_data.Operation winlog.user_data.User winlog.user_data.ResultCode
    1                       <NA>                  <NA>                        <NA>
    2                       <NA>                  <NA>                        <NA>
    3                       <NA>                  <NA>                        <NA>
      winlog.user_data.ClientMachine winlog.user_data.SessionID winlog.user_data.ESS winlog.user_data.CONSUMER
    1                           <NA>                       <NA>                 <NA>                      <NA>
    2                           <NA>                       <NA>                 <NA>                      <NA>
    3                           <NA>                       <NA>                 <NA>                      <NA>
      winlog.user_data.Namespace winlog.user_data.Processid winlog.user_data.NamespaceName
    1                       <NA>                       <NA>                           <NA>
    2                       <NA>                       <NA>                           <NA>
    3                       <NA>                       <NA>                           <NA>
      winlog.user_data.Query winlog.user_data.Provider winlog.user_data.queryid winlog.user_data.Session
    1                   <NA>                      <NA>                     <NA>                     <NA>
    2                   <NA>                      <NA>                     <NA>                     <NA>
    3                   <NA>                      <NA>                     <NA>                     <NA>
      winlog.user_data.Reason winlog.user_data.Address winlog.user_data.messageName
    1                    <NA>                     <NA>                         <NA>
    2                    <NA>                     <NA>                         <NA>
    3                    <NA>                     <NA>                         <NA>
      winlog.user_data.ListenerName winlog.user_data.Class winlog.user_data.listenerName ecs.version host.name
    1                          <NA>                   <NA>                          <NA>       1.1.0 WECServer
    2                          <NA>                   <NA>                          <NA>       1.1.0 WECServer
    3                          <NA>                   <NA>                          <NA>       1.1.0 WECServer
                        agent.ephemeral_id agent.hostname                             agent.id agent.version
    1 b372be1f-ba0a-4d7e-b4df-79eac86e1fde      WECServer d347d9a4-bff4-476c-b5a4-d51119f78250         7.4.0
    2 b372be1f-ba0a-4d7e-b4df-79eac86e1fde      WECServer d347d9a4-bff4-476c-b5a4-d51119f78250         7.4.0
    3 b372be1f-ba0a-4d7e-b4df-79eac86e1fde      WECServer d347d9a4-bff4-476c-b5a4-d51119f78250         7.4.0
      agent.type @timestamp_datetime event.created_datetime winlog.event_data.UtcTime_datetime
    1 winlogbeat 2019-10-20 20:11:06    2019-10-20 20:11:09                               <NA>
    2 winlogbeat 2019-10-20 20:11:07    2019-10-20 20:11:09                               <NA>
    3 winlogbeat 2019-10-20 20:11:09    2019-10-20 20:11:11                               <NA>
      winlog.event_data.CreationUtcTime_datetime winlog.event_data.PreviousCreationUtcTime_datetime
    1                                       <NA>                                               <NA>
    2                                       <NA>                                               <NA>
    3                                       <NA>                                               <NA>
      winlog.event_data.PreviousTime_datetime winlog.event_data.NewTime_datetime
    1                                    <NA>                               <NA>
    2                                    <NA>                               <NA>
    3                                    <NA>                               <NA>
      winlog.event_data.OldTime_datetime winlog.event_data.StopTime_datetime
    1                               <NA>                                <NA>
    2                               <NA>                                <NA>
    3                               <NA>                                <NA>
      winlog.event_data.StartTime_datetime winlog.event_data.BitlockerUserInputTime_datetime
    1                                 <NA>                                              <NA>
    2                                 <NA>                                              <NA>
    3                                 <NA>                                              <NA>
      winlog.event_data.DeviceTime_datetime winlog.event_data.TimeSource_datetime
    1                                  <NA>                                  <NA>
    2                                  <NA>                                  <NA>
    3                                  <NA>                                  <NA>
    > 
    > # Способ 2: Создаем свою функцию для просмотра структуры
    > cat("\n=== СТРУКТУРА ДАННЫХ ===\n")

    === СТРУКТУРА ДАННЫХ ===
    > cat("Общее количество строк:", nrow(data_clean), "\n")
    Общее количество строк: 101904 
    > cat("Общее количество столбцов:", ncol(data_clean), "\n\n")
    Общее количество столбцов: 313 

    > 
    > # Посмотрим на типы данных и примеры значений
    > for (i in 1:min(10, ncol(data_clean))) { # Ограничимся первыми 10 столбцами
    +     col_name <- names(data_clean)[i]
    +     col_type <- class(data_clean[[col_name]])[1]
    +     unique_count <- length(unique(data_clean[[col_name]]))
    +     
    +     cat(i, ". ", col_name, " [", col_type, "]\n", sep = "")
    +     cat("   Уникальных значений: ", unique_count, "\n", sep = "")
    +     
    +     # Покажем несколько примеров значений (только для не-NA)
    +     non_na_vals <- na.omit(data_clean[[col_name]])
    +     if (length(non_na_vals) > 0) {
    +         sample_vals <- head(unique(non_na_vals), 3)
    +         cat("   Примеры: ", paste(sample_vals, collapse = ", "), "\n", sep = "")
    +     }
    +     cat("\n")
    + }
    1. @timestamp [character]
       Уникальных значений: 26366
       Примеры: 2019-10-20T20:11:06.937Z, 2019-10-20T20:11:07.101Z, 2019-10-20T20:11:09.052Z

    2. message [character]
       Уникальных значений: 47944
       Примеры: A token right was adjusted.

    Subject:
        Security ID:        S-1-5-18
        Account Name:       HR001$
        Account Domain:     shire
        Logon ID:       0x3E7

    Target Account:
        Security ID:        S-1-5-18
        Account Name:       HR001$
        Account Domain:     shire
        Logon ID:       0x3E7

    Process Information:
        Process ID:     0x804
        Process Name:       C:\Windows\System32\svchost.exe

    Enabled Privileges:
                SeTakeOwnershipPrivilege

    Disabled Privileges:
                -, A privileged service was called.

    Subject:
        Security ID:        S-1-5-19
        Account Name:       LOCAL SERVICE
        Account Domain:     NT AUTHORITY
        Logon ID:       0x3E5

    Service:
        Server: Security
        Service Name:   -

    Process:
        Process ID: 0x494
        Process Name:   C:\Windows\System32\svchost.exe

    Service Request Information:
        Privileges:     SeProfileSingleProcessPrivilege, Process accessed:
    RuleName: 
    UtcTime: 2019-10-20 20:11:09.052
    SourceProcessGUID: {a158f72c-afec-5dac-0000-001030640200}
    SourceProcessId: 3556
    SourceThreadId: 3688
    SourceImage: C:\Windows\System32\svchost.exe
    TargetProcessGUID: {a158f72c-afeb-5dac-0000-001082220200}
    TargetProcessId: 3108
    TargetImage: C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
    GrantedAccess: 0x1400
    CallTrace: C:\Windows\SYSTEM32\ntdll.dll+9c524|C:\Windows\System32\KERNELBASE.dll+2730e|c:\windows\system32\rasmans.dll+3751b|c:\windows\system32\rasmans.dll+36fad|c:\windows\system32\rasmans.dll+10ed8|c:\windows\system32\rasmans.dll+33cd5|C:\Windows\System32\svchost.exe+314c|C:\Windows\System32\sechost.dll+2de2|C:\Windows\System32\KERNEL32.DLL+17944|C:\Windows\SYSTEM32\ntdll.dll+6ce71

    3. @metadata.beat [character]
       Уникальных значений: 1
       Примеры: winlogbeat

    4. @metadata.type [character]
       Уникальных значений: 1
       Примеры: _doc

    5. @metadata.version [character]
       Уникальных значений: 1
       Примеры: 7.4.0

    6. @metadata.topic [character]
       Уникальных значений: 1
       Примеры: winlogbeat

    7. event.created [character]
       Уникальных значений: 3761
       Примеры: 2019-10-20T20:11:09.988Z, 2019-10-20T20:11:11.995Z, 2019-10-20T20:11:14.013Z

    8. event.kind [character]
       Уникальных значений: 1
       Примеры: event

    9. event.code [integer]
       Уникальных значений: 109
       Примеры: 4703, 4673, 10

    10. event.action [character]
       Уникальных значений: 51
       Примеры: Token Right Adjusted Events, Sensitive Privilege Use, Process accessed (rule: ProcessAccess)

    > 
    > # 9. Проверим наличие важных столбцов событий Windows
    > important_cols <- c("event.code", "winlog.event_id", "message", "log.level", 
    +                     "winlog.channel", "winlog.computer_name", "host.name")
    > 
    > cat("\n=== ВАЖНЫЕ СТОЛБЦЫ ДЛЯ АНАЛИЗА СОБЫТИЙ WINDOWS ===\n")

    === ВАЖНЫЕ СТОЛБЦЫ ДЛЯ АНАЛИЗА СОБЫТИЙ WINDOWS ===
    > for (col in important_cols) {
    +     exists <- col %in% names(data_clean)
    +     cat(col, ": ", ifelse(exists, "ЕСТЬ", "ОТСУТСТВУЕТ"), "\n", sep = "")
    + }
    event.code: ЕСТЬ
    winlog.event_id: ЕСТЬ
    message: ЕСТЬ
    log.level: ЕСТЬ
    winlog.channel: ЕСТЬ
    winlog.computer_name: ЕСТЬ
    host.name: ЕСТЬ
    > 
    > # 10. Если есть столбец event.code, посмотрим на распределение событий
    > if ("event.code" %in% names(data_clean)) {
    +     cat("\n=== РАСПРЕДЕЛЕНИЕ СОБЫТИЙ ПО КОДАМ ===\n")
    +     event_counts <- table(data_clean$event.code)
    +     print(head(event_counts[order(-event_counts)], 10)) # Топ-10 событий
    + }

    === РАСПРЕДЕЛЕНИЕ СОБЫТИЙ ПО КОДАМ ===

       10   800  4103  4105  4106    12     7  4703  5156     3 
    36074 12342 12166 11456 11456  6354  5937   809   756   582 
    > 
    > # 11. Альтернативный способ просмотра через tibble
    > cat("\n=== ПРОСМОТР ЧЕРЕЗ TIBBLE ===\n")

    === ПРОСМОТР ЧЕРЕЗ TIBBLE ===
    > if (require(tibble)) {
    +     data_tibble <- as_tibble(data_clean)
    +     print(data_tibble)
    +     
    +     # Используем glimpse если он работает
    +     tryCatch({
    +         cat("\nИспользуем glimpse():\n")
    +         glimpse(data_tibble)
    +     }, error = function(e) {
    +         cat("glimpse() не работает, используем альтернативы\n")
    +     })
    + }
    # A tibble: 101,904 × 313
       `@timestamp`             message `@metadata.beat` `@metadata.type` `@metadata.version` `@metadata.topic`
       <chr>                    <chr>   <chr>            <chr>            <chr>               <chr>            
     1 2019-10-20T20:11:06.937Z "A tok… winlogbeat       _doc             7.4.0               winlogbeat       
     2 2019-10-20T20:11:07.101Z "A pri… winlogbeat       _doc             7.4.0               winlogbeat       
     3 2019-10-20T20:11:09.052Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
     4 2019-10-20T20:11:10.985Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
     5 2019-10-20T20:11:11.249Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
     6 2019-10-20T20:11:15.017Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
     7 2019-10-20T20:11:15.438Z "File … winlogbeat       _doc             7.4.0               winlogbeat       
     8 2019-10-20T20:11:15.522Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
     9 2019-10-20T20:11:15.522Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
    10 2019-10-20T20:11:16.460Z "Proce… winlogbeat       _doc             7.4.0               winlogbeat       
    # ℹ 101,894 more rows
    # ℹ 307 more variables: event.created <chr>, event.kind <chr>, event.code <int>, event.action <chr>,
    #   log.level <chr>, winlog.event_id <int>, winlog.provider_name <chr>, winlog.api <chr>,
    #   winlog.record_id <int>, winlog.computer_name <chr>, winlog.keywords <list>,
    #   winlog.provider_guid <chr>, winlog.channel <chr>, winlog.task <chr>, winlog.opcode <chr>,
    #   winlog.version <int>, winlog.activity_id <chr>, winlog.event_data.SubjectDomainName <chr>,
    #   winlog.event_data.TargetDomainName <chr>, winlog.event_data.SubjectUserSid <chr>, …
    # ℹ Use `print(n = ...)` to see more rows, and `colnames()` to see all variable names

    Используем glimpse():
    Rows: 101,904
    Columns: 313
    $ `@timestamp`                                       <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:…
    $ message                                            <chr> "A token right was adjusted.\n\nSubject:\n\tSe…
    $ `@metadata.beat`                                   <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…
    $ `@metadata.type`                                   <chr> "_doc", "_doc", "_doc", "_doc", "_doc", "_doc"…
    $ `@metadata.version`                                <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "…
    $ `@metadata.topic`                                  <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…
    $ event.created                                      <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:…
    $ event.kind                                         <chr> "event", "event", "event", "event", "event", "…
    $ event.code                                         <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10…
    $ event.action                                       <chr> "Token Right Adjusted Events", "Sensitive Priv…
    $ log.level                                          <chr> "information", "information", "information", "…
    $ winlog.event_id                                    <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10…
    $ winlog.provider_name                               <chr> "Microsoft-Windows-Security-Auditing", "Micros…
    $ winlog.api                                         <chr> "wineventlog", "wineventlog", "wineventlog", "…
    $ winlog.record_id                                   <int> 50588, 104875, 226649, 153525, 163488, 153526,…
    $ winlog.computer_name                               <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.…
    $ winlog.keywords                                    <list> "Audit Success", "Audit Failure", <NULL>, <NU…
    $ winlog.provider_guid                               <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog.channel                                     <chr> "security", "Security", "Microsoft-Windows-Sys…
    $ winlog.task                                        <chr> "Token Right Adjusted Events", "Sensitive Priv…
    $ winlog.opcode                                      <chr> "Info", "Info", "Info", "Info", "Info", "Info"…
    $ winlog.version                                     <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA…
    $ winlog.activity_id                                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SubjectDomainName                <chr> "shire", "NT AUTHORITY", NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TargetDomainName                 <chr> "shire", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.SubjectUserSid                   <chr> "S-1-5-18", "S-1-5-19", NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SubjectUserName                  <chr> "HR001$", "LOCAL SERVICE", NA, NA, NA, NA, NA,…
    $ winlog.event_data.TargetUserName                   <chr> "HR001$", NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.EnabledPrivilegeList             <chr> "SeTakeOwnershipPrivilege", NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetLogonId                    <chr> "0x3e7", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ProcessName                      <chr> "C:\\Windows\\System32\\svchost.exe", "C:\\Win…
    $ winlog.event_data.ProcessId                        <chr> "0x804", "0x494", NA, NA, NA, NA, "1464", NA, …
    $ winlog.event_data.SubjectLogonId                   <chr> "0x3e7", "0x3e5", NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.TargetUserSid                    <chr> "S-1-5-18", NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DisabledPrivilegeList            <chr> "-", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ObjectServer                     <chr> NA, "Security", NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Service                          <chr> NA, "-", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PrivilegeList                    <chr> NA, "SeProfileSingleProcessPrivilege", NA, NA,…
    $ winlog.event_data.TargetProcessId                  <chr> NA, NA, "3108", "1968", "2032", "7592", NA, "6…
    $ winlog.event_data.SourceProcessId                  <chr> NA, NA, "3556", "3548", "3632", "7112", NA, "9…
    $ winlog.event_data.SourceProcessGUID                <chr> NA, NA, "{a158f72c-afec-5dac-0000-001030640200…
    $ winlog.event_data.SourceThreadId                   <chr> NA, NA, "3688", "3660", "3772", "7004", NA, "8…
    $ winlog.event_data.SourceImage                      <chr> NA, NA, "C:\\Windows\\System32\\svchost.exe", …
    $ winlog.event_data.CallTrace                        <chr> NA, NA, "C:\\Windows\\SYSTEM32\\ntdll.dll+9c52…
    $ winlog.event_data.TargetProcessGUID                <chr> NA, NA, "{a158f72c-afeb-5dac-0000-001082220200…
    $ winlog.event_data.TargetImage                      <chr> NA, NA, "C:\\Program Files\\Amazon\\Ec2ConfigS…
    $ winlog.event_data.GrantedAccess                    <chr> NA, NA, "0x1400", "0x1400", "0x1400", "0x1400"…
    $ winlog.event_data.UtcTime                          <chr> NA, NA, "2019-10-20 20:11:09.052", "2019-10-20…
    $ winlog.event_data.ProcessGuid                      <chr> NA, NA, NA, NA, NA, NA, "{b2887b82-a64e-5dac-0…
    $ winlog.event_data.Image                            <chr> NA, NA, NA, NA, NA, NA, "C:\\Windows\\System32…
    $ winlog.event_data.TargetFilename                   <chr> NA, NA, NA, NA, NA, NA, "C:\\Windows\\ServiceS…
    $ winlog.event_data.CreationUtcTime                  <chr> NA, NA, NA, NA, NA, NA, "2019-10-20 18:25:15.1…
    $ winlog.event_data.FileVersion                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "1…
    $ winlog.event_data.Company                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "M…
    $ winlog.event_data.Signed                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "t…
    $ winlog.event_data.Signature                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "M…
    $ winlog.event_data.OriginalFileName                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "P…
    $ winlog.event_data.Description                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "P…
    $ winlog.event_data.Product                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "M…
    $ winlog.event_data.ImageLoaded                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "C…
    $ winlog.event_data.SignatureStatus                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "V…
    $ winlog.event_data.Hashes                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "S…
    $ winlog.event_data.Status                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Protocol                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.FilterRTID                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LayerName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LayerRTID                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Application                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceAddress                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourcePort                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ProcessID                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestPort                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RemoteMachineID                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RemoteUserID                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Direction                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestAddress                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RestrictedAdminMode              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TransmittedServices              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonType                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.WorkstationName                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LmPackageName                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.KeyLength                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonGuid                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ImpersonationLevel               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetOutboundDomainName         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetLinkedLogonId              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AuthenticationPackageName        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetOutboundUserName           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.VirtualAccount                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IpAddress                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ElevatedToken                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonProcessName                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IpPort                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.GroupMembership                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EventIdx                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EventCountTotal                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EventType                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetObject                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationPort                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationIsIpv6                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceIp                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.User                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceIsIpv6                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationIp                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceHostname                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationHostname              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationPortName              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Initiated                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param3                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param2                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param1                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentProcessId                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentProcessGuid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentCommandLine                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CommandLine                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CurrentDirectory                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TerminalSessionId                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentImage                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonId                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IntegrityLevel                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PipeName                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ScriptBlockId                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RunspaceId                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ContextInfo                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Payload                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MessageNumber                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MessageTotal                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ScriptBlockText                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewProcessId                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewProcessName                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MandatoryLabel                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentProcessName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TokenElevationType               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ServiceName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ServiceSid                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TicketOptions                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TicketEncryptionType             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QueryName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QueryStatus                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QueryResults                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourcePortName                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Device                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Details                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ObjectName                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OldSd                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewSd                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ObjectType                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.HandleId                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShareName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AccessList                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShareLocalPath                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AccessMask                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AccessReason                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RelativeTargetName               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Properties                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdditionalInfo2                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OperationType                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdditionalInfo                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Path                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetHandleId                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceHandleId                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TransactionId                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RestrictedSidCount               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ResourceAttributes               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Binary                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PreviousCreationUtcTime          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CallerProcessId                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CallerProcessName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetSid                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TaskName                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TaskContentNew                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceProcessGuid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetProcessGuid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartAddress                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewThreadId                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TaskContent                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PackageName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Workstation                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param4                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param5                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param7                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartModule                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartFunction                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TSId                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.UserSid                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PreAuthType                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PreviousTime                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewTime                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.InterfaceName                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OldProfile                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewProfile                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.InterfaceGuid                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingType                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingValueSize                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingValue                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingValueDisplay              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Origin                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ModifyingUser                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Reason                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OldTime                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DwordVal                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param6                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShutdownActionType               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShutdownEventCode                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShutdownReason                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StopTime                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootMode                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartTime                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MajorVersion                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MinorVersion                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BuildVersion                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QfeVersion                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ServiceVersion                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EnableDisableReason              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.VsmPolicy                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LastBootGood                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LastBootId                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootStatusPolicy                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LastShutdownGood                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootMenuPolicy                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootType                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LoadOptions                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EntryCount                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BitlockerUserInputTime           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CountNew                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CountOld                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.UpdateReason                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EnabledNew                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceNameLength                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceName                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceTime                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.FinalStatus                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceVersionMajor               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceVersionMinor               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DriveName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CorruptionActionState            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.State                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MinimumPerformancePercent        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MaximumPerformancePercent        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PerformanceImplementation        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Group                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Number                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IdleStateCount                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IdleImplementation               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NominalFrequency                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MinimumThrottlePercent           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Config                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IsTestConfig                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ `winlog.event_data.Default SD String:`             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdapterSuffixName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DnsServerList                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ `winlog.event_data.Sent UpdateServer`              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Ipaddress                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ErrorCode                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdapterName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.HostName                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TimeSource                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.process.pid                                 <int> 4, 4, 3220, 3136, 3184, 3136, 2728, 3220, 3220…
    $ winlog.process.thread.id                           <int> 4108, 5144, 4972, 4672, 5028, 4672, 3368, 4972…
    $ winlog.user.domain                                 <chr> NA, NA, "NT AUTHORITY", "NT AUTHORITY", "NT AU…
    $ winlog.user.type                                   <chr> NA, NA, "User", "User", "User", "User", "User"…
    $ winlog.user.identifier                             <chr> NA, NA, "S-1-5-18", "S-1-5-18", "S-1-5-18", "S…
    $ winlog.user.name                                   <chr> NA, NA, "SYSTEM", "SYSTEM", "SYSTEM", "SYSTEM"…
    $ winlog.user_data.ProcessID                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ProviderPath                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.xml_name                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ProviderName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Code                              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.HostProcess                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Id                                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.PossibleCause                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ClientProcessId                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Component                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Operation                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.User                              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ResultCode                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ClientMachine                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.SessionID                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ESS                               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.CONSUMER                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Namespace                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Processid                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.NamespaceName                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Query                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Provider                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.queryid                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Session                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Reason                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Address                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.messageName                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ListenerName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Class                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.listenerName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ ecs.version                                        <chr> "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1.0", "…
    $ host.name                                          <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent.ephemeral_id                                 <chr> "b372be1f-ba0a-4d7e-b4df-79eac86e1fde", "b372b…
    $ agent.hostname                                     <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent.id                                           <chr> "d347d9a4-bff4-476c-b5a4-d51119f78250", "d347d…
    $ agent.version                                      <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "…
    $ agent.type                                         <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…
    $ `@timestamp_datetime`                              <dttm> 2019-10-20 20:11:06, 2019-10-20 20:11:07, 201…
    $ event.created_datetime                             <dttm> 2019-10-20 20:11:09, 2019-10-20 20:11:09, 201…
    $ winlog.event_data.UtcTime_datetime                 <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CreationUtcTime_datetime         <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PreviousCreationUtcTime_datetime <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PreviousTime_datetime            <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.NewTime_datetime                 <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.OldTime_datetime                 <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.StopTime_datetime                <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.StartTime_datetime               <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.BitlockerUserInputTime_datetime  <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.DeviceTime_datetime              <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TimeSource_datetime              <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    > 
    > # 12. Сохраним данные для дальнейшего анализа
    > save_path <- "/Users/vladimir/Desktop/windows_events_clean.rds"
    > saveRDS(data_clean, save_path)
    > cat("\nДанные сохранены в:", save_path, "\n")

    Данные сохранены в: /Users/vladimir/Desktop/windows_events_clean.rds 
    > 
    > # 13. Выведем сводную информацию
    > cat("\n=== СВОДНАЯ ИНФОРМАЦИЯ ===\n")

    === СВОДНАЯ ИНФОРМАЦИЯ ===
    > cat("1. Файл содержит записи событий Windows\n")
    1. Файл содержит записи событий Windows
    > cat("2. Формат: Windows Event Log в JSON\n")
    2. Формат: Windows Event Log в JSON
    > cat("3. Источник: вероятно, Windows Security Auditing\n")
    3. Источник: вероятно, Windows Security Auditing
    > cat("4. Типичные события: аудит безопасности, изменения токенов доступа\n")
    4. Типичные события: аудит безопасности, изменения токенов доступа
    > cat("5. Структура: вложенный JSON развернут в плоскую таблицу\n")
    5. Структура: вложенный JSON развернут в плоскую таблицу
    > 

## Общая структура данных с помощью функции glimpse()

    glimpse(data_clean)
    Rows: 101,904
    Columns: 313
    $ `@timestamp`                                       <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:…
    $ message                                            <chr> "A token right was adjusted.\n\nSubject:\n\tSe…
    $ `@metadata.beat`                                   <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…
    $ `@metadata.type`                                   <chr> "_doc", "_doc", "_doc", "_doc", "_doc", "_doc"…
    $ `@metadata.version`                                <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "…
    $ `@metadata.topic`                                  <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…
    $ event.created                                      <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:…
    $ event.kind                                         <chr> "event", "event", "event", "event", "event", "…
    $ event.code                                         <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10…
    $ event.action                                       <chr> "Token Right Adjusted Events", "Sensitive Priv…
    $ log.level                                          <chr> "information", "information", "information", "…
    $ winlog.event_id                                    <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10…
    $ winlog.provider_name                               <chr> "Microsoft-Windows-Security-Auditing", "Micros…
    $ winlog.api                                         <chr> "wineventlog", "wineventlog", "wineventlog", "…
    $ winlog.record_id                                   <int> 50588, 104875, 226649, 153525, 163488, 153526,…
    $ winlog.computer_name                               <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.…
    $ winlog.keywords                                    <list> "Audit Success", "Audit Failure", <NULL>, <NU…
    $ winlog.provider_guid                               <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
    $ winlog.channel                                     <chr> "security", "Security", "Microsoft-Windows-Sys…
    $ winlog.task                                        <chr> "Token Right Adjusted Events", "Sensitive Priv…
    $ winlog.opcode                                      <chr> "Info", "Info", "Info", "Info", "Info", "Info"…
    $ winlog.version                                     <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA…
    $ winlog.activity_id                                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SubjectDomainName                <chr> "shire", "NT AUTHORITY", NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TargetDomainName                 <chr> "shire", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.SubjectUserSid                   <chr> "S-1-5-18", "S-1-5-19", NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SubjectUserName                  <chr> "HR001$", "LOCAL SERVICE", NA, NA, NA, NA, NA,…
    $ winlog.event_data.TargetUserName                   <chr> "HR001$", NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.EnabledPrivilegeList             <chr> "SeTakeOwnershipPrivilege", NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetLogonId                    <chr> "0x3e7", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ProcessName                      <chr> "C:\\Windows\\System32\\svchost.exe", "C:\\Win…
    $ winlog.event_data.ProcessId                        <chr> "0x804", "0x494", NA, NA, NA, NA, "1464", NA, …
    $ winlog.event_data.SubjectLogonId                   <chr> "0x3e7", "0x3e5", NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.TargetUserSid                    <chr> "S-1-5-18", NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DisabledPrivilegeList            <chr> "-", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ObjectServer                     <chr> NA, "Security", NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Service                          <chr> NA, "-", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PrivilegeList                    <chr> NA, "SeProfileSingleProcessPrivilege", NA, NA,…
    $ winlog.event_data.TargetProcessId                  <chr> NA, NA, "3108", "1968", "2032", "7592", NA, "6…
    $ winlog.event_data.SourceProcessId                  <chr> NA, NA, "3556", "3548", "3632", "7112", NA, "9…
    $ winlog.event_data.SourceProcessGUID                <chr> NA, NA, "{a158f72c-afec-5dac-0000-001030640200…
    $ winlog.event_data.SourceThreadId                   <chr> NA, NA, "3688", "3660", "3772", "7004", NA, "8…
    $ winlog.event_data.SourceImage                      <chr> NA, NA, "C:\\Windows\\System32\\svchost.exe", …
    $ winlog.event_data.CallTrace                        <chr> NA, NA, "C:\\Windows\\SYSTEM32\\ntdll.dll+9c52…
    $ winlog.event_data.TargetProcessGUID                <chr> NA, NA, "{a158f72c-afeb-5dac-0000-001082220200…
    $ winlog.event_data.TargetImage                      <chr> NA, NA, "C:\\Program Files\\Amazon\\Ec2ConfigS…
    $ winlog.event_data.GrantedAccess                    <chr> NA, NA, "0x1400", "0x1400", "0x1400", "0x1400"…
    $ winlog.event_data.UtcTime                          <chr> NA, NA, "2019-10-20 20:11:09.052", "2019-10-20…
    $ winlog.event_data.ProcessGuid                      <chr> NA, NA, NA, NA, NA, NA, "{b2887b82-a64e-5dac-0…
    $ winlog.event_data.Image                            <chr> NA, NA, NA, NA, NA, NA, "C:\\Windows\\System32…
    $ winlog.event_data.TargetFilename                   <chr> NA, NA, NA, NA, NA, NA, "C:\\Windows\\ServiceS…
    $ winlog.event_data.CreationUtcTime                  <chr> NA, NA, NA, NA, NA, NA, "2019-10-20 18:25:15.1…
    $ winlog.event_data.FileVersion                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "1…
    $ winlog.event_data.Company                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "M…
    $ winlog.event_data.Signed                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "t…
    $ winlog.event_data.Signature                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "M…
    $ winlog.event_data.OriginalFileName                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "P…
    $ winlog.event_data.Description                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "P…
    $ winlog.event_data.Product                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "M…
    $ winlog.event_data.ImageLoaded                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "C…
    $ winlog.event_data.SignatureStatus                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "V…
    $ winlog.event_data.Hashes                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, "S…
    $ winlog.event_data.Status                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Protocol                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.FilterRTID                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LayerName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LayerRTID                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Application                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceAddress                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourcePort                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ProcessID                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestPort                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RemoteMachineID                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RemoteUserID                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Direction                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestAddress                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RestrictedAdminMode              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TransmittedServices              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonType                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.WorkstationName                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LmPackageName                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.KeyLength                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonGuid                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ImpersonationLevel               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetOutboundDomainName         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetLinkedLogonId              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AuthenticationPackageName        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetOutboundUserName           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.VirtualAccount                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IpAddress                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ElevatedToken                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonProcessName                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IpPort                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.GroupMembership                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EventIdx                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EventCountTotal                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EventType                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetObject                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationPort                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationIsIpv6                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceIp                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.User                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceIsIpv6                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationIp                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceHostname                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationHostname              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DestinationPortName              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Initiated                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param3                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param2                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param1                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentProcessId                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentProcessGuid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentCommandLine                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CommandLine                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CurrentDirectory                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TerminalSessionId                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentImage                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LogonId                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IntegrityLevel                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PipeName                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ScriptBlockId                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RunspaceId                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ContextInfo                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Payload                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MessageNumber                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MessageTotal                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ScriptBlockText                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewProcessId                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewProcessName                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MandatoryLabel                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ParentProcessName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TokenElevationType               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ServiceName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ServiceSid                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TicketOptions                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TicketEncryptionType             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QueryName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QueryStatus                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QueryResults                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourcePortName                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Device                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Details                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ObjectName                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OldSd                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewSd                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ObjectType                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.HandleId                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShareName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AccessList                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShareLocalPath                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AccessMask                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AccessReason                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RelativeTargetName               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Properties                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdditionalInfo2                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OperationType                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdditionalInfo                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Path                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetHandleId                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceHandleId                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TransactionId                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.RestrictedSidCount               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ResourceAttributes               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Binary                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PreviousCreationUtcTime          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CallerProcessId                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CallerProcessName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetSid                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TaskName                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TaskContentNew                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SourceProcessGuid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TargetProcessGuid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartAddress                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewThreadId                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TaskContent                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PackageName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Workstation                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param4                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param5                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param7                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartModule                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartFunction                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TSId                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.UserSid                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PreAuthType                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PreviousTime                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewTime                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.InterfaceName                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OldProfile                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NewProfile                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.InterfaceGuid                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingType                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingValueSize                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingValue                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.SettingValueDisplay              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Origin                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ModifyingUser                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Reason                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.OldTime                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DwordVal                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.param6                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShutdownActionType               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShutdownEventCode                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ShutdownReason                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StopTime                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootMode                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.StartTime                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MajorVersion                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MinorVersion                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BuildVersion                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.QfeVersion                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ServiceVersion                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EnableDisableReason              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.VsmPolicy                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LastBootGood                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LastBootId                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootStatusPolicy                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LastShutdownGood                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootMenuPolicy                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BootType                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.LoadOptions                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EntryCount                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.BitlockerUserInputTime           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CountNew                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CountOld                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.UpdateReason                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.EnabledNew                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceNameLength                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceName                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceTime                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.FinalStatus                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceVersionMajor               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DeviceVersionMinor               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DriveName                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.CorruptionActionState            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.State                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MinimumPerformancePercent        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MaximumPerformancePercent        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.PerformanceImplementation        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Group                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Number                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IdleStateCount                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IdleImplementation               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.NominalFrequency                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.MinimumThrottlePercent           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Config                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.IsTestConfig                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ `winlog.event_data.Default SD String:`             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdapterSuffixName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.DnsServerList                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ `winlog.event_data.Sent UpdateServer`              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Ipaddress                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.ErrorCode                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.AdapterName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.HostName                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.TimeSource                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.process.pid                                 <int> 4, 4, 3220, 3136, 3184, 3136, 2728, 3220, 3220…
    $ winlog.process.thread.id                           <int> 4108, 5144, 4972, 4672, 5028, 4672, 3368, 4972…
    $ winlog.user.domain                                 <chr> NA, NA, "NT AUTHORITY", "NT AUTHORITY", "NT AU…
    $ winlog.user.type                                   <chr> NA, NA, "User", "User", "User", "User", "User"…
    $ winlog.user.identifier                             <chr> NA, NA, "S-1-5-18", "S-1-5-18", "S-1-5-18", "S…
    $ winlog.user.name                                   <chr> NA, NA, "SYSTEM", "SYSTEM", "SYSTEM", "SYSTEM"…
    $ winlog.user_data.ProcessID                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ProviderPath                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.xml_name                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ProviderName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Code                              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.HostProcess                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Id                                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.PossibleCause                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ClientProcessId                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Component                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Operation                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.User                              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ResultCode                        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ClientMachine                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.SessionID                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ESS                               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.CONSUMER                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Namespace                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Processid                         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.NamespaceName                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Query                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Provider                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.queryid                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Session                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Reason                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Address                           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.messageName                       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.ListenerName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.Class                             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.user_data.listenerName                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ ecs.version                                        <chr> "1.1.0", "1.1.0", "1.1.0", "1.1.0", "1.1.0", "…
    $ host.name                                          <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent.ephemeral_id                                 <chr> "b372be1f-ba0a-4d7e-b4df-79eac86e1fde", "b372b…
    $ agent.hostname                                     <chr> "WECServer", "WECServer", "WECServer", "WECSer…
    $ agent.id                                           <chr> "d347d9a4-bff4-476c-b5a4-d51119f78250", "d347d…
    $ agent.version                                      <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", "…
    $ agent.type                                         <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…
    $ `@timestamp_datetime`                              <dttm> 2019-10-20 20:11:06, 2019-10-20 20:11:07, 201…
    $ event.created_datetime                             <dttm> 2019-10-20 20:11:09, 2019-10-20 20:11:09, 201…
    $ winlog.event_data.UtcTime_datetime                 <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CreationUtcTime_datetime         <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PreviousCreationUtcTime_datetime <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PreviousTime_datetime            <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.NewTime_datetime                 <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.OldTime_datetime                 <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.StopTime_datetime                <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.StartTime_datetime               <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.BitlockerUserInputTime_datetime  <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.DeviceTime_datetime              <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TimeSource_datetime              <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    > 

## 1. Раскройте датафрейм избавившись от вложенных датафреймов. Для обнаружения таких можно использовать функцию dplyr::glimpse(), а для раскрытия вложенности – tidyr::unnest(). Обратите внимание, что при раскрытии теряются внешние названия колонок – это можно предотвратить если использовать параметр tidyr::unnest(…, names_sep = ).

    # === МИНИМИЗАЦИЯ КОЛИЧЕСТВА КОЛОНОК ===
    > # Удаление колонок с единственным значением параметра
    > 
    > # 1. Посмотрим текущее количество колонок
    > cat("=== МИНИМИЗАЦИЯ КОЛИЧЕСТВА КОЛОНОК ===\n")
    === МИНИМИЗАЦИЯ КОЛИЧЕСТВА КОЛОНОК ===
    > cat("Текущее количество колонок:", ncol(data_clean), "\n")
    Текущее количество колонок: 313 
    > cat("Текущее количество строк:", nrow(data_clean), "\n\n")
    Текущее количество строк: 15965 

    > 
    > # 2. Функция для определения, имеет ли колонка только одно уникальное значение
    > # (учитывая NA как возможное значение)
    > count_unique_values <- function(x) {
    +     # Учитываем NA как отдельное значение, если они есть
    +     if (any(is.na(x))) {
    +         # Считаем уникальные значения, включая NA
    +         unique_vals <- unique(x)
    +         return(length(unique_vals))
    +     } else {
    +         # Если нет NA, просто считаем уникальные значения
    +         return(length(unique(x)))
    +     }
    + }
    > 
    > # 3. Создаем таблицу с информацией об уникальных значениях в каждой колонке
    > col_stats <- data.frame(
    +     column_name = names(data_clean),
    +     unique_values = sapply(data_clean, count_unique_values),
    +     all_missing = sapply(data_clean, function(x) all(is.na(x))),
    +     row.names = NULL
    + )
    > 
    > # 4. Определяем колонки для удаления
    > # Удаляем колонки с:
    > # а) только одним уникальным значением (включая NA)
    > # б) полностью пустые (все значения NA)
    > columns_to_remove <- col_stats$column_name[col_stats$unique_values <= 1]
    > 
    > cat("=== КОЛОНКИ ДЛЯ УДАЛЕНИЯ ===\n")
    === КОЛОНКИ ДЛЯ УДАЛЕНИЯ ===
    > if (length(columns_to_remove) > 0) {
    +     cat("Найдено колонок с единственным значением:", length(columns_to_remove), "\n\n")
    +     
    +     # Выводим информацию об удаляемых колонках
    +     removal_info <- col_stats[col_stats$column_name %in% columns_to_remove, ]
    +     print(removal_info)
    +     
    +     # Покажем примеры значений из этих колонок
    +     cat("\nПримеры значений из удаляемых колонок:\n")
    +     for (col in columns_to_remove) {
    +         unique_vals <- unique(data_clean[[col]])
    +         cat(sprintf("\n%s: ", col))
    +         
    +         if (all(is.na(data_clean[[col]]))) {
    +             cat("[ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]")
    +         } else if (length(unique_vals) == 1) {
    +             # Показываем единственное значение
    +             cat(as.character(unique_vals[1]))
    +             
    +             # Показываем, сколько раз это значение встречается
    +             count <- sum(!is.na(data_clean[[col]]))
    +             cat(sprintf(" (встречается %d раз)", count))
    +         }
    +     }
    +     
    +     # 5. Удаляем колонки
    +     data_minimized <- data_clean %>%
    +         select(-all_of(columns_to_remove))
    +     
    +     cat("\n\n=== РЕЗУЛЬТАТ УДАЛЕНИЯ ===\n")
    +     cat("Удалено колонок:", length(columns_to_remove), "\n")
    +     cat("Осталось колонок:", ncol(data_minimized), "\n")
    +     cat("Сохранено строк:", nrow(data_minimized), "\n")
    +     
    +     # 6. Покажем, какие колонки остались
    +     cat("\n=== ОСТАВШИЕСЯ КОЛОНКИ ===\n")
    +     cat("Всего колонок осталось:", ncol(data_minimized), "\n")
    +     
    +     # Создаем таблицу с информацией об оставшихся колонках
    +     remaining_stats <- data.frame(
    +         column_name = names(data_minimized),
    +         unique_values = sapply(data_minimized, count_unique_values),
    +         data_type = sapply(data_minimized, function(x) class(x)[1]),
    +         row.names = NULL
    +     )
    +     
    +     # Сортируем по количеству уникальных значений
    +     remaining_stats <- remaining_stats %>%
    +         arrange(desc(unique_values))
    +     
    +     print(remaining_stats)
    +     
    +     # 7. Сохраняем минимизированные данные
    +     saveRDS(data_minimized, "/Users/vladimir/Desktop/windows_events_minimized.rds")
    +     cat("\nМинимизированные данные сохранены в файл: windows_events_minimized.rds\n")
    +     
    +     # 8. Обновляем основной датафрейм
    +     data_clean <- data_minimized
    +     
    + } else {
    +     cat("Колонок с единственным значением не найдено.\n")
    +     cat("Все колонки имеют 2 или более уникальных значения.\n")
    + }
    Найдено колонок с единственным значением: 179 

                                               column_name unique_values all_missing
    3                                       @metadata.beat             1       FALSE
    4                                       @metadata.type             1       FALSE
    5                                    @metadata.version             1       FALSE
    6                                      @metadata.topic             1       FALSE
    8                                           event.kind             1       FALSE
    14                                          winlog.api             1       FALSE
    41                 winlog.event_data.SourceProcessGUID             1        TRUE
    42                    winlog.event_data.SourceThreadId             1        TRUE
    43                       winlog.event_data.SourceImage             1        TRUE
    44                         winlog.event_data.CallTrace             1        TRUE
    45                 winlog.event_data.TargetProcessGUID             1        TRUE
    46                       winlog.event_data.TargetImage             1        TRUE
    47                     winlog.event_data.GrantedAccess             1        TRUE
    48                           winlog.event_data.UtcTime             1        TRUE
    49                       winlog.event_data.ProcessGuid             1        TRUE
    50                             winlog.event_data.Image             1        TRUE
    51                    winlog.event_data.TargetFilename             1        TRUE
    52                   winlog.event_data.CreationUtcTime             1        TRUE
    53                       winlog.event_data.FileVersion             1        TRUE
    54                           winlog.event_data.Company             1        TRUE
    55                            winlog.event_data.Signed             1        TRUE
    56                         winlog.event_data.Signature             1        TRUE
    57                  winlog.event_data.OriginalFileName             1        TRUE
    58                       winlog.event_data.Description             1        TRUE
    59                           winlog.event_data.Product             1        TRUE
    60                       winlog.event_data.ImageLoaded             1        TRUE
    61                   winlog.event_data.SignatureStatus             1        TRUE
    62                            winlog.event_data.Hashes             1        TRUE
    97                         winlog.event_data.EventType             1        TRUE
    98                      winlog.event_data.TargetObject             1        TRUE
    99                   winlog.event_data.DestinationPort             1        TRUE
    100                winlog.event_data.DestinationIsIpv6             1        TRUE
    101                         winlog.event_data.SourceIp             1        TRUE
    102                             winlog.event_data.User             1        TRUE
    103                     winlog.event_data.SourceIsIpv6             1        TRUE
    104                    winlog.event_data.DestinationIp             1        TRUE
    105                   winlog.event_data.SourceHostname             1        TRUE
    106              winlog.event_data.DestinationHostname             1        TRUE
    107              winlog.event_data.DestinationPortName             1        TRUE
    108                        winlog.event_data.Initiated             1        TRUE
    112                  winlog.event_data.ParentProcessId             1        TRUE
    113                winlog.event_data.ParentProcessGuid             1        TRUE
    114                winlog.event_data.ParentCommandLine             1        TRUE
    116                 winlog.event_data.CurrentDirectory             1        TRUE
    117                winlog.event_data.TerminalSessionId             1        TRUE
    118                      winlog.event_data.ParentImage             1        TRUE
    119                          winlog.event_data.LogonId             1        TRUE
    120                   winlog.event_data.IntegrityLevel             1        TRUE
    121                         winlog.event_data.PipeName             1        TRUE
    122                    winlog.event_data.ScriptBlockId             1        TRUE
    123                       winlog.event_data.RunspaceId             1        TRUE
    124                      winlog.event_data.ContextInfo             1        TRUE
    125                          winlog.event_data.Payload             1        TRUE
    126                    winlog.event_data.MessageNumber             1        TRUE
    127                     winlog.event_data.MessageTotal             1        TRUE
    128                  winlog.event_data.ScriptBlockText             1        TRUE
    138                        winlog.event_data.QueryName             1        TRUE
    139                      winlog.event_data.QueryStatus             1        TRUE
    140                     winlog.event_data.QueryResults             1        TRUE
    141                   winlog.event_data.SourcePortName             1        TRUE
    142                           winlog.event_data.Device             1        TRUE
    143                          winlog.event_data.Details             1        TRUE
    159                             winlog.event_data.Path             1        TRUE
    166          winlog.event_data.PreviousCreationUtcTime             1        TRUE
    172                winlog.event_data.SourceProcessGuid             1        TRUE
    173                winlog.event_data.TargetProcessGuid             1        TRUE
    174                     winlog.event_data.StartAddress             1        TRUE
    175                      winlog.event_data.NewThreadId             1        TRUE
    182                      winlog.event_data.StartModule             1        TRUE
    183                    winlog.event_data.StartFunction             1        TRUE
    184                             winlog.event_data.TSId             1        TRUE
    185                          winlog.event_data.UserSid             1        TRUE
    189                    winlog.event_data.InterfaceName             1        TRUE
    190                       winlog.event_data.OldProfile             1        TRUE
    191                       winlog.event_data.NewProfile             1        TRUE
    192                    winlog.event_data.InterfaceGuid             1        TRUE
    193                      winlog.event_data.SettingType             1        TRUE
    194                 winlog.event_data.SettingValueSize             1        TRUE
    195                     winlog.event_data.SettingValue             1        TRUE
    196              winlog.event_data.SettingValueDisplay             1        TRUE
    197                           winlog.event_data.Origin             1        TRUE
    198                    winlog.event_data.ModifyingUser             1        TRUE
    201                         winlog.event_data.DwordVal             1        TRUE
    203               winlog.event_data.ShutdownActionType             1        TRUE
    204                winlog.event_data.ShutdownEventCode             1        TRUE
    205                   winlog.event_data.ShutdownReason             1        TRUE
    206                         winlog.event_data.StopTime             1        TRUE
    207                         winlog.event_data.BootMode             1        TRUE
    208                        winlog.event_data.StartTime             1        TRUE
    209                     winlog.event_data.MajorVersion             1        TRUE
    210                     winlog.event_data.MinorVersion             1        TRUE
    211                     winlog.event_data.BuildVersion             1        TRUE
    212                       winlog.event_data.QfeVersion             1        TRUE
    213                   winlog.event_data.ServiceVersion             1        TRUE
    214              winlog.event_data.EnableDisableReason             1        TRUE
    215                        winlog.event_data.VsmPolicy             1        TRUE
    216                     winlog.event_data.LastBootGood             1        TRUE
    217                       winlog.event_data.LastBootId             1        TRUE
    218                 winlog.event_data.BootStatusPolicy             1        TRUE
    219                 winlog.event_data.LastShutdownGood             1        TRUE
    220                   winlog.event_data.BootMenuPolicy             1        TRUE
    221                         winlog.event_data.BootType             1        TRUE
    222                      winlog.event_data.LoadOptions             1        TRUE
    223                       winlog.event_data.EntryCount             1        TRUE
    224           winlog.event_data.BitlockerUserInputTime             1        TRUE
    229                 winlog.event_data.DeviceNameLength             1        TRUE
    230                       winlog.event_data.DeviceName             1        TRUE
    231                       winlog.event_data.DeviceTime             1        TRUE
    232                      winlog.event_data.FinalStatus             1        TRUE
    233               winlog.event_data.DeviceVersionMajor             1        TRUE
    234               winlog.event_data.DeviceVersionMinor             1        TRUE
    235                        winlog.event_data.DriveName             1        TRUE
    236            winlog.event_data.CorruptionActionState             1        TRUE
    237                            winlog.event_data.State             1        TRUE
    238        winlog.event_data.MinimumPerformancePercent             1        TRUE
    239        winlog.event_data.MaximumPerformancePercent             1        TRUE
    240        winlog.event_data.PerformanceImplementation             1        TRUE
    241                            winlog.event_data.Group             1        TRUE
    242                           winlog.event_data.Number             1        TRUE
    243                   winlog.event_data.IdleStateCount             1        TRUE
    244               winlog.event_data.IdleImplementation             1        TRUE
    245                 winlog.event_data.NominalFrequency             1        TRUE
    246           winlog.event_data.MinimumThrottlePercent             1        TRUE
    247                           winlog.event_data.Config             1        TRUE
    248                     winlog.event_data.IsTestConfig             1        TRUE
    249               winlog.event_data.Default SD String:             1        TRUE
    250                winlog.event_data.AdapterSuffixName             1        TRUE
    251                    winlog.event_data.DnsServerList             1        TRUE
    252                winlog.event_data.Sent UpdateServer             1        TRUE
    253                        winlog.event_data.Ipaddress             1        TRUE
    254                        winlog.event_data.ErrorCode             1        TRUE
    255                      winlog.event_data.AdapterName             1        TRUE
    256                         winlog.event_data.HostName             1        TRUE
    257                       winlog.event_data.TimeSource             1        TRUE
    264                         winlog.user_data.ProcessID             1        TRUE
    265                      winlog.user_data.ProviderPath             1        TRUE
    266                          winlog.user_data.xml_name             1        TRUE
    267                      winlog.user_data.ProviderName             1        TRUE
    268                              winlog.user_data.Code             1        TRUE
    269                       winlog.user_data.HostProcess             1        TRUE
    270                                winlog.user_data.Id             1        TRUE
    271                     winlog.user_data.PossibleCause             1        TRUE
    272                   winlog.user_data.ClientProcessId             1        TRUE
    273                         winlog.user_data.Component             1        TRUE
    274                         winlog.user_data.Operation             1        TRUE
    275                              winlog.user_data.User             1        TRUE
    276                        winlog.user_data.ResultCode             1        TRUE
    277                     winlog.user_data.ClientMachine             1        TRUE
    278                         winlog.user_data.SessionID             1        TRUE
    279                               winlog.user_data.ESS             1        TRUE
    280                          winlog.user_data.CONSUMER             1        TRUE
    281                         winlog.user_data.Namespace             1        TRUE
    282                         winlog.user_data.Processid             1        TRUE
    283                     winlog.user_data.NamespaceName             1        TRUE
    284                             winlog.user_data.Query             1        TRUE
    285                          winlog.user_data.Provider             1        TRUE
    286                           winlog.user_data.queryid             1        TRUE
    287                           winlog.user_data.Session             1        TRUE
    288                            winlog.user_data.Reason             1        TRUE
    289                           winlog.user_data.Address             1        TRUE
    290                       winlog.user_data.messageName             1        TRUE
    291                      winlog.user_data.ListenerName             1        TRUE
    292                             winlog.user_data.Class             1        TRUE
    293                      winlog.user_data.listenerName             1        TRUE
    294                                        ecs.version             1       FALSE
    295                                          host.name             1       FALSE
    296                                 agent.ephemeral_id             1       FALSE
    297                                     agent.hostname             1       FALSE
    298                                           agent.id             1       FALSE
    299                                      agent.version             1       FALSE
    300                                         agent.type             1       FALSE
    303                 winlog.event_data.UtcTime_datetime             1        TRUE
    304         winlog.event_data.CreationUtcTime_datetime             1        TRUE
    305 winlog.event_data.PreviousCreationUtcTime_datetime             1        TRUE
    309                winlog.event_data.StopTime_datetime             1        TRUE
    310               winlog.event_data.StartTime_datetime             1        TRUE
    311  winlog.event_data.BitlockerUserInputTime_datetime             1        TRUE
    312              winlog.event_data.DeviceTime_datetime             1        TRUE
    313              winlog.event_data.TimeSource_datetime             1        TRUE

    Примеры значений из удаляемых колонок:

    @metadata.beat: winlogbeat (встречается 15965 раз)
    @metadata.type: _doc (встречается 15965 раз)
    @metadata.version: 7.4.0 (встречается 15965 раз)
    @metadata.topic: winlogbeat (встречается 15965 раз)
    event.kind: event (встречается 15965 раз)
    winlog.api: wineventlog (встречается 15965 раз)
    winlog.event_data.SourceProcessGUID: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourceThreadId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourceImage: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.CallTrace: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TargetProcessGUID: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TargetImage: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.GrantedAccess: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.UtcTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ProcessGuid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Image: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TargetFilename: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.CreationUtcTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.FileVersion: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Company: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Signed: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Signature: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.OriginalFileName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Description: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Product: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ImageLoaded: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SignatureStatus: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Hashes: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.EventType: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TargetObject: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DestinationPort: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DestinationIsIpv6: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourceIp: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.User: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourceIsIpv6: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DestinationIp: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourceHostname: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DestinationHostname: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DestinationPortName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Initiated: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ParentProcessId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ParentProcessGuid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ParentCommandLine: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.CurrentDirectory: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TerminalSessionId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ParentImage: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.LogonId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.IntegrityLevel: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.PipeName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ScriptBlockId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.RunspaceId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ContextInfo: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Payload: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MessageNumber: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MessageTotal: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ScriptBlockText: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.QueryName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.QueryStatus: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.QueryResults: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourcePortName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Device: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Details: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Path: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.PreviousCreationUtcTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SourceProcessGuid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TargetProcessGuid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StartAddress: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.NewThreadId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StartModule: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StartFunction: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TSId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.UserSid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.InterfaceName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.OldProfile: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.NewProfile: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.InterfaceGuid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SettingType: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SettingValueSize: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SettingValue: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.SettingValueDisplay: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Origin: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ModifyingUser: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DwordVal: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ShutdownActionType: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ShutdownEventCode: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ShutdownReason: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StopTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BootMode: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StartTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MajorVersion: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MinorVersion: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BuildVersion: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.QfeVersion: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ServiceVersion: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.EnableDisableReason: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.VsmPolicy: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.LastBootGood: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.LastBootId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BootStatusPolicy: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.LastShutdownGood: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BootMenuPolicy: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BootType: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.LoadOptions: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.EntryCount: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BitlockerUserInputTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DeviceNameLength: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DeviceName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DeviceTime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.FinalStatus: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DeviceVersionMajor: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DeviceVersionMinor: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DriveName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.CorruptionActionState: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.State: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MinimumPerformancePercent: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MaximumPerformancePercent: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.PerformanceImplementation: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Group: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Number: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.IdleStateCount: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.IdleImplementation: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.NominalFrequency: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.MinimumThrottlePercent: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Config: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.IsTestConfig: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Default SD String:: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.AdapterSuffixName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DnsServerList: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Sent UpdateServer: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.Ipaddress: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.ErrorCode: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.AdapterName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.HostName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TimeSource: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ProcessID: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ProviderPath: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.xml_name: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ProviderName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Code: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.HostProcess: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Id: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.PossibleCause: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ClientProcessId: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Component: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Operation: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.User: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ResultCode: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ClientMachine: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.SessionID: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ESS: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.CONSUMER: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Namespace: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Processid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.NamespaceName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Query: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Provider: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.queryid: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Session: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Reason: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Address: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.messageName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.ListenerName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.Class: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.user_data.listenerName: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    ecs.version: 1.1.0 (встречается 15965 раз)
    host.name: WECServer (встречается 15965 раз)
    agent.ephemeral_id: b372be1f-ba0a-4d7e-b4df-79eac86e1fde (встречается 15965 раз)
    agent.hostname: WECServer (встречается 15965 раз)
    agent.id: d347d9a4-bff4-476c-b5a4-d51119f78250 (встречается 15965 раз)
    agent.version: 7.4.0 (встречается 15965 раз)
    agent.type: winlogbeat (встречается 15965 раз)
    winlog.event_data.UtcTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.CreationUtcTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.PreviousCreationUtcTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StopTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.StartTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.BitlockerUserInputTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.DeviceTime_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]
    winlog.event_data.TimeSource_datetime: [ВСЕ ЗНАЧЕНИЯ ПРОПУЩЕНЫ]

    === РЕЗУЛЬТАТ УДАЛЕНИЯ ===
    Удалено колонок: 179 
    Осталось колонок: 134 
    Сохранено строк: 15965 

    === ОСТАВШИЕСЯ КОЛОНКИ ===
    Всего колонок осталось: 134 
                                        column_name unique_values data_type
    1                              winlog.record_id         15954   integer
    2                                       message         14885 character
    3                      winlog.event_data.param2         12350 character
    4                                    @timestamp          3866 character
    5                           @timestamp_datetime          3866   POSIXct
    6                                 event.created          1468 character
    7                        event.created_datetime          1468   POSIXct
    8                      winlog.event_data.param3           904 character
    9                  winlog.event_data.SourcePort           514 character
    10                     winlog.event_data.param1           316 character
    11                  winlog.event_data.ProcessId           259 character
    12               winlog.event_data.NewProcessId           106 character
    13                   winlog.event_data.HandleId           106 character
    14              winlog.event_data.TargetLogonId           101 character
    15         winlog.event_data.RelativeTargetName            93 character
    16                     winlog.event_data.IpPort            91 character
    17                winlog.event_data.CommandLine            85 character
    18             winlog.event_data.SubjectLogonId            70 character
    19                winlog.event_data.ProcessName            65 character
    20                  winlog.event_data.ProcessID            53 character
    21                     winlog.process.thread.id            52   integer
    22                                   event.code            48   integer
    23                              winlog.event_id            48   integer
    24             winlog.event_data.SourceHandleId            39 character
    25             winlog.event_data.TargetHandleId            31 character
    26                                 event.action            30 character
    27                                  winlog.task            30 character
    28             winlog.event_data.NewProcessName            29 character
    29                 winlog.event_data.FilterRTID            24 character
    30                 winlog.event_data.AccessMask            23 character
    31             winlog.event_data.TargetUserName            22 character
    32                 winlog.event_data.AccessList            21 character
    33                winlog.event_data.DestAddress            20 character
    34                  winlog.event_data.LogonGuid            20 character
    35                 winlog.event_data.ObjectName            20 character
    36                winlog.event_data.Application            19 character
    37             winlog.event_data.SubjectUserSid            18 character
    38            winlog.event_data.SourceProcessId            18 character
    39            winlog.event_data.SubjectUserName            17 character
    40                   winlog.event_data.DestPort            16 character
    41              winlog.event_data.TargetUserSid            14 character
    42              winlog.event_data.SourceAddress            13 character
    43                 winlog.event_data.ObjectType            13 character
    44       winlog.event_data.EnabledPrivilegeList            12 character
    45                           winlog.process.pid            12   integer
    46               winlog.event_data.AccessReason            11 character
    47                         winlog.provider_name            10 character
    48           winlog.event_data.TargetDomainName            10 character
    49              winlog.event_data.PrivilegeList            10 character
    50                  winlog.event_data.IpAddress            10 character
    51                         winlog.provider_guid             9 character
    52          winlog.event_data.ParentProcessName             9 character
    53                      winlog.event_data.NewSd             9 character
    54          winlog.event_data.SubjectDomainName             8 character
    55                     winlog.event_data.Status             8 character
    56                     winlog.event_data.Binary             8 character
    57                  winlog.event_data.TargetSid             8 character
    58                  winlog.event_data.LayerRTID             7 character
    59            winlog.event_data.GroupMembership             7 character
    60               winlog.event_data.ObjectServer             6 character
    61                winlog.event_data.ServiceName             6 character
    62                 winlog.event_data.ServiceSid             6 character
    63                      winlog.event_data.OldSd             6 character
    64                 winlog.event_data.Properties             6 character
    65                         winlog.computer_name             5 character
    66                           winlog.activity_id             5 character
    67                   winlog.event_data.Protocol             5 character
    68           winlog.event_data.LogonProcessName             5 character
    69             winlog.event_data.MandatoryLabel             5 character
    70              winlog.event_data.TicketOptions             5 character
    71                       winlog.user.identifier             5 character
    72                             winlog.user.name             5 character
    73                              winlog.keywords             4 character
    74                               winlog.channel             4 character
    75      winlog.event_data.DisabledPrivilegeList             4 character
    76                  winlog.event_data.LayerName             4 character
    77                  winlog.event_data.LogonType             4 character
    78  winlog.event_data.AuthenticationPackageName             4 character
    79         winlog.event_data.TokenElevationType             4 character
    80                  winlog.event_data.ShareName             4 character
    81            winlog.event_data.CallerProcessId             4 character
    82          winlog.event_data.CallerProcessName             4 character
    83                     winlog.event_data.param5             4 character
    84                                    log.level             3 character
    85                               winlog.version             3   integer
    86                    winlog.event_data.Service             3 character
    87                  winlog.event_data.Direction             3 character
    88            winlog.event_data.WorkstationName             3 character
    89              winlog.event_data.LmPackageName             3 character
    90                  winlog.event_data.KeyLength             3 character
    91         winlog.event_data.ImpersonationLevel             3 character
    92             winlog.event_data.ShareLocalPath             3 character
    93                   winlog.event_data.TaskName             3 character
    94             winlog.event_data.TaskContentNew             3 character
    95                     winlog.event_data.param4             3 character
    96                     winlog.event_data.param7             3 character
    97                           winlog.user.domain             3 character
    98                             winlog.user.type             3 character
    99                                winlog.opcode             2 character
    100           winlog.event_data.TargetProcessId             2 character
    101           winlog.event_data.RemoteMachineID             2 character
    102              winlog.event_data.RemoteUserID             2 character
    103       winlog.event_data.RestrictedAdminMode             2 character
    104       winlog.event_data.TransmittedServices             2 character
    105  winlog.event_data.TargetOutboundDomainName             2 character
    106       winlog.event_data.TargetLinkedLogonId             2 character
    107    winlog.event_data.TargetOutboundUserName             2 character
    108            winlog.event_data.VirtualAccount             2 character
    109             winlog.event_data.ElevatedToken             2 character
    110                  winlog.event_data.EventIdx             2 character
    111           winlog.event_data.EventCountTotal             2 character
    112      winlog.event_data.TicketEncryptionType             2 character
    113           winlog.event_data.AdditionalInfo2             2 character
    114             winlog.event_data.OperationType             2 character
    115            winlog.event_data.AdditionalInfo             2 character
    116             winlog.event_data.TransactionId             2 character
    117        winlog.event_data.RestrictedSidCount             2 character
    118        winlog.event_data.ResourceAttributes             2 character
    119               winlog.event_data.TaskContent             2 character
    120               winlog.event_data.PackageName             2 character
    121               winlog.event_data.Workstation             2 character
    122               winlog.event_data.PreAuthType             2 character
    123              winlog.event_data.PreviousTime             2 character
    124                   winlog.event_data.NewTime             2 character
    125                    winlog.event_data.Reason             2 character
    126                   winlog.event_data.OldTime             2 character
    127                    winlog.event_data.param6             2 character
    128                  winlog.event_data.CountNew             2 character
    129                  winlog.event_data.CountOld             2 character
    130              winlog.event_data.UpdateReason             2 character
    131                winlog.event_data.EnabledNew             2 character
    132     winlog.event_data.PreviousTime_datetime             2   POSIXct
    133          winlog.event_data.NewTime_datetime             2   POSIXct
    134          winlog.event_data.OldTime_datetime             2   POSIXct

    Минимизированные данные сохранены в файл: windows_events_minimized.rds
    > 
    > # 9. Проверяем структуру после минимизации
    > cat("\n=== СТРУКТУРА ДАННЫХ ПОСЛЕ МИНИМИЗАЦИИ ===\n")

    === СТРУКТУРА ДАННЫХ ПОСЛЕ МИНИМИЗАЦИИ ===
    > cat("Итоговое количество колонок:", ncol(data_clean), "\n")
    Итоговое количество колонок: 134 
    > cat("Итоговое количество строк:", nrow(data_clean), "\n\n")
    Итоговое количество строк: 15965 

    > 
    > # Используем glimpse() для просмотра структуры
    > dplyr::glimpse(data_clean)
    Rows: 15,965
    Columns: 134
    $ `@timestamp`                                <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.101Z…
    $ message                                     <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity …
    $ event.created                               <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.988Z…
    $ event.code                                  <int> 4703, 4673, 4689, 4703, 5158, 5156, 5156, 4672, 4624,…
    $ event.action                                <chr> "Token Right Adjusted Events", "Sensitive Privilege U…
    $ log.level                                   <chr> "information", "information", "information", "informa…
    $ winlog.event_id                             <int> 4703, 4673, 4689, 4703, 5158, 5156, 5156, 4672, 4624,…
    $ winlog.provider_name                        <chr> "Microsoft-Windows-Security-Auditing", "Microsoft-Win…
    $ winlog.record_id                            <int> 50588, 104875, 26042, 22614, 104876, 104877, 104878, …
    $ winlog.computer_name                        <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shire.c…
    $ winlog.keywords                             <chr> "Audit Success", "Audit Failure", "Audit Success", "A…
    $ winlog.provider_guid                        <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54849625-…
    $ winlog.channel                              <chr> "security", "Security", "security", "security", "Secu…
    $ winlog.task                                 <chr> "Token Right Adjusted Events", "Sensitive Privilege U…
    $ winlog.opcode                               <chr> "Info", "Info", "Info", "Info", "Info", "Info", "Info…
    $ winlog.version                              <int> NA, NA, NA, NA, NA, 1, 1, NA, 2, NA, NA, NA, 1, NA, 1…
    $ winlog.activity_id                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.SubjectDomainName         <chr> "shire", "NT AUTHORITY", "shire", "shire", NA, NA, NA…
    $ winlog.event_data.TargetDomainName          <chr> "shire", NA, NA, "shire", NA, NA, NA, NA, "SHIRE.COM"…
    $ winlog.event_data.SubjectUserSid            <chr> "S-1-5-18", "S-1-5-19", "S-1-5-21-1335669728-33486816…
    $ winlog.event_data.SubjectUserName           <chr> "HR001$", "LOCAL SERVICE", "pgustavo", "ACCT001$", NA…
    $ winlog.event_data.TargetUserName            <chr> "HR001$", NA, NA, "ACCT001$", NA, NA, NA, NA, "HFDC01…
    $ winlog.event_data.EnabledPrivilegeList      <chr> "SeTakeOwnershipPrivilege", NA, NA, "SeTakeOwnershipP…
    $ winlog.event_data.TargetLogonId             <chr> "0x3e7", NA, NA, "0x3e7", NA, NA, NA, NA, "0x637923",…
    $ winlog.event_data.ProcessName               <chr> "C:\\Windows\\System32\\svchost.exe", "C:\\Windows\\S…
    $ winlog.event_data.ProcessId                 <chr> "0x804", "0x494", "0x1618", "0x728", "2660", NA, NA, …
    $ winlog.event_data.SubjectLogonId            <chr> "0x3e7", "0x3e5", "0x4c563", "0x3e7", NA, NA, NA, "0x…
    $ winlog.event_data.TargetUserSid             <chr> "S-1-5-18", NA, NA, "S-1-5-18", NA, NA, NA, NA, "S-1-…
    $ winlog.event_data.DisabledPrivilegeList     <chr> "-", NA, NA, "-", NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    $ winlog.event_data.ObjectServer              <chr> NA, "Security", NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.Service                   <chr> NA, "-", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.PrivilegeList             <chr> NA, "SeProfileSingleProcessPrivilege", NA, NA, NA, NA…
    $ winlog.event_data.TargetProcessId           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.SourceProcessId           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.Status                    <chr> NA, NA, "0x1", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ winlog.event_data.Protocol                  <chr> NA, NA, NA, NA, "6", "6", "6", NA, NA, NA, NA, "6", "…
    $ winlog.event_data.FilterRTID                <chr> NA, NA, NA, NA, "0", "65855", "65853", NA, NA, NA, NA…
    $ winlog.event_data.LayerName                 <chr> NA, NA, NA, NA, "%%14608", "%%14611", "%%14610", NA, …
    $ winlog.event_data.LayerRTID                 <chr> NA, NA, NA, NA, "38", "50", "46", NA, NA, NA, NA, "36…
    $ winlog.event_data.Application               <chr> NA, NA, NA, NA, "\\device\\harddiskvolume1\\windows\\…
    $ winlog.event_data.SourceAddress             <chr> NA, NA, NA, NA, "::", "::1", "::1", NA, NA, NA, NA, "…
    $ winlog.event_data.SourcePort                <chr> NA, NA, NA, NA, "60759", "60759", "60759", NA, NA, NA…
    $ winlog.event_data.ProcessID                 <chr> NA, NA, NA, NA, NA, "2660", "780", NA, NA, NA, NA, NA…
    $ winlog.event_data.DestPort                  <chr> NA, NA, NA, NA, NA, "389", "389", NA, NA, NA, NA, NA,…
    $ winlog.event_data.RemoteMachineID           <chr> NA, NA, NA, NA, NA, "S-1-0-0", "S-1-0-0", NA, NA, NA,…
    $ winlog.event_data.RemoteUserID              <chr> NA, NA, NA, NA, NA, "S-1-0-0", "S-1-0-0", NA, NA, NA,…
    $ winlog.event_data.Direction                 <chr> NA, NA, NA, NA, NA, "%%14593", "%%14592", NA, NA, NA,…
    $ winlog.event_data.DestAddress               <chr> NA, NA, NA, NA, NA, "::1", "::1", NA, NA, NA, NA, NA,…
    $ winlog.event_data.RestrictedAdminMode       <chr> NA, NA, NA, NA, NA, NA, NA, NA, "-", NA, NA, NA, NA, …
    $ winlog.event_data.TransmittedServices       <chr> NA, NA, NA, NA, NA, NA, NA, NA, "-", NA, NA, NA, NA, …
    $ winlog.event_data.LogonType                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, "3", "3", "3", NA, NA…
    $ winlog.event_data.WorkstationName           <chr> NA, NA, NA, NA, NA, NA, NA, NA, "-", NA, NA, NA, NA, …
    $ winlog.event_data.LmPackageName             <chr> NA, NA, NA, NA, NA, NA, NA, NA, "-", NA, NA, NA, NA, …
    $ winlog.event_data.KeyLength                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, "0", NA, NA, NA, NA, …
    $ winlog.event_data.LogonGuid                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, "{984659d6-ba2c-d289-…
    $ winlog.event_data.ImpersonationLevel        <chr> NA, NA, NA, NA, NA, NA, NA, NA, "%%1833", NA, NA, NA,…
    $ winlog.event_data.TargetOutboundDomainName  <chr> NA, NA, NA, NA, NA, NA, NA, NA, "-", NA, NA, NA, NA, …
    $ winlog.event_data.TargetLinkedLogonId       <chr> NA, NA, NA, NA, NA, NA, NA, NA, "0x0", NA, NA, NA, NA…
    $ winlog.event_data.AuthenticationPackageName <chr> NA, NA, NA, NA, NA, NA, NA, NA, "Kerberos", NA, NA, N…
    $ winlog.event_data.TargetOutboundUserName    <chr> NA, NA, NA, NA, NA, NA, NA, NA, "-", NA, NA, NA, NA, …
    $ winlog.event_data.VirtualAccount            <chr> NA, NA, NA, NA, NA, NA, NA, NA, "%%1843", NA, NA, NA,…
    $ winlog.event_data.IpAddress                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, "::1", NA, NA, NA, NA…
    $ winlog.event_data.ElevatedToken             <chr> NA, NA, NA, NA, NA, NA, NA, NA, "%%1842", NA, NA, NA,…
    $ winlog.event_data.LogonProcessName          <chr> NA, NA, NA, NA, NA, NA, NA, NA, "Kerberos", NA, NA, N…
    $ winlog.event_data.IpPort                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, "60759", NA, NA, NA, …
    $ winlog.event_data.GroupMembership           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, "\n\t\t%{S-1-5-32…
    $ winlog.event_data.EventIdx                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, "1", NA, NA, NA, …
    $ winlog.event_data.EventCountTotal           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, "1", NA, NA, NA, …
    $ winlog.event_data.param3                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.param2                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.param1                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CommandLine               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.NewProcessId              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.NewProcessName            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.MandatoryLabel            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ParentProcessName         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TokenElevationType        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ServiceName               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ServiceSid                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TicketOptions             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TicketEncryptionType      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ObjectName                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.OldSd                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.NewSd                     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ObjectType                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.HandleId                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ShareName                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.AccessList                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ShareLocalPath            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.AccessMask                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.AccessReason              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.RelativeTargetName        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.Properties                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.AdditionalInfo2           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.OperationType             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.AdditionalInfo            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TargetHandleId            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.SourceHandleId            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TransactionId             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.RestrictedSidCount        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.ResourceAttributes        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.Binary                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CallerProcessId           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CallerProcessName         <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TargetSid                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TaskName                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TaskContentNew            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.TaskContent               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PackageName               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.Workstation               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.param4                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.param5                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.param7                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PreAuthType               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.PreviousTime              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.NewTime                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.Reason                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.OldTime                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.param6                    <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CountNew                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.CountOld                  <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.UpdateReason              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.event_data.EnabledNew                <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.process.pid                          <int> 4, 4, 4, 4, 4, 4, 4, 780, 780, 780, 780, 4, 4, 4, 4, …
    $ winlog.process.thread.id                    <int> 4108, 5144, 8772, 7024, 5144, 5144, 5144, 2864, 2864,…
    $ winlog.user.domain                          <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.user.type                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.user.identifier                      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ winlog.user.name                            <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ `@timestamp_datetime`                       <dttm> 2019-10-20 20:11:06, 2019-10-20 20:11:07, 2019-10-20…
    $ event.created_datetime                      <dttm> 2019-10-20 20:11:09, 2019-10-20 20:11:09, 2019-10-20…
    $ winlog.event_data.PreviousTime_datetime     <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.NewTime_datetime          <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    $ winlog.event_data.OldTime_datetime          <dttm> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    > 
    > # 10. Дополнительная информация о распределении уникальных значений
    > cat("\n=== РАСПРЕДЕЛЕНИЕ КОЛОНОК ПО КОЛИЧЕСТВУ УНИКАЛЬНЫХ ЗНАЧЕНИЙ ===\n")

    === РАСПРЕДЕЛЕНИЕ КОЛОНОК ПО КОЛИЧЕСТВУ УНИКАЛЬНЫХ ЗНАЧЕНИЙ ===
    > value_distribution <- table(col_stats$unique_values)
    > print(value_distribution)

        1     2     3     4     5     6     7     8     9    10    11    12    13    14    16    17    18 
      179    36    15    11     8     5     2     4     3     4     1     2     2     1     1     1     2 
       19    20    21    22    23    24    29    30    31    39    48    52    53    65    70    85    91 
        1     3     1     1     1     1     1     2     1     1     2     1     1     1     1     1     1 
       93   101   106   259   316   514   904  1468  3866 12350 14885 15954 
        1     1     2     1     1     1     1     2     2     1     1     1 
    > 
    > # 11. Анализ эффективности минимизации
    > cat("\n=== АНАЛИЗ ЭФФЕКТИВНОСТИ МИНИМИЗАЦИИ ===\n")

    === АНАЛИЗ ЭФФЕКТИВНОСТИ МИНИМИЗАЦИИ ===
    > original_cols <- ncol(data_clean) + length(columns_to_remove)
    > removed_percentage <- round(length(columns_to_remove) / original_cols * 100, 1)
    > 
    > cat("Изначально колонок:", original_cols, "\n")
    Изначально колонок: 313 
    > cat("Удалено колонок:", length(columns_to_remove), "\n")
    Удалено колонок: 179 
    > cat("Осталось колонок:", ncol(data_clean), "\n")
    Осталось колонок: 134 
    > cat("Сокращение объема данных:", removed_percentage, "%\n")
    Сокращение объема данных: 57.2 %
    > 
    > # 12. Проверка на наличие важных колонок после минимизации
    > cat("\n=== ПРОВЕРКА ВАЖНЫХ КОЛОНОК ===\n")

    === ПРОВЕРКА ВАЖНЫХ КОЛОНОК ===
    > important_columns <- c(
    +     "timestamp", "event_code", "winlog_event_id", "message",
    +     "winlog_channel", "winlog_computer_name", "processname",
    +     "subjectusername", "targetusername"
    + )
    > 
    > for (col in important_columns) {
    +     exists <- col %in% names(data_clean)
    +     if (exists) {
    +         unique_count <- count_unique_values(data_clean[[col]])
    +         cat(sprintf("%-25s: ПРИСУТСТВУЕТ, уникальных значений: %d\n", col, unique_count))
    +     } else {
    +         cat(sprintf("%-25s: ОТСУТСТВУЕТ\n", col))
    +     }
    + }
    timestamp                : ОТСУТСТВУЕТ
    event_code               : ОТСУТСТВУЕТ
    winlog_event_id          : ОТСУТСТВУЕТ
    message                  : ПРИСУТСТВУЕТ, уникальных значений: 14885
    winlog_channel           : ОТСУТСТВУЕТ
    winlog_computer_name     : ОТСУТСТВУЕТ
    processname              : ОТСУТСТВУЕТ
    subjectusername          : ОТСУТСТВУЕТ
    targetusername           : ОТСУТСТВУЕТ

## 2. Какое количество хостов представлено в данном датасете?

    # === АНАЛИЗ: КОЛИЧЕСТВО ХОСТОВ В ДАТАСЕТЕ ===
    > 
    > # Ищем колонки, которые могут содержать информацию о хостах/компьютерах
    > cat("=== АНАЛИЗ КОЛИЧЕСТВА ХОСТОВ ===\n\n")
    === АНАЛИЗ КОЛИЧЕСТВА ХОСТОВ ===

    > 
    > # Определяем возможные названия колонок с информацией о хостах
    > possible_host_columns <- c(
    +     "host_name", "hostname", "computer", "computer_name", 
    +     "winlog_computer_name", "winlog_computername", "host",
    +     "agent_hostname", "host_hostname", "system", "machine"
    + )
    > 
    > # Находим существующие колонки
    > existing_host_columns <- possible_host_columns[possible_host_columns %in% names(data_clean)]
    > 
    > if (length(existing_host_columns) > 0) {
    +     cat("Найдены колонки с информацией о хостах:\n")
    +     for (col in existing_host_columns) {
    +         cat(sprintf("- %s\n", col))
    +     }
    +     
    +     # Проверяем каждую колонку на наличие данных
    +     cat("\nАнализ каждой колонки:\n")
    +     host_info <- data.frame()
    +     
    +     for (col in existing_host_columns) {
    +         unique_count <- length(unique(na.omit(data_clean[[col]])))
    +         non_missing <- sum(!is.na(data_clean[[col]]))
    +         missing <- sum(is.na(data_clean[[col]]))
    +         
    +         cat(sprintf("\nКолонка '%s':\n", col))
    +         cat(sprintf("  Уникальных значений: %d\n", unique_count))
    +         cat(sprintf("  Непустых значений: %d (%.1f%%)\n", non_missing, non_missing/nrow(data_clean)*100))
    +         cat(sprintf("  Пропущенных значений: %d (%.1f%%)\n", missing, missing/nrow(data_clean)*100))
    +         
    +         # Показываем примеры значений
    +         if (unique_count > 0) {
    +             examples <- head(unique(na.omit(data_clean[[col]])), 5)
    +             cat(sprintf("  Примеры: %s", paste(examples, collapse = ", ")))
    +             if (unique_count > 5) cat(" ...")
    +             cat("\n")
    +         }
    +         
    +         # Сохраняем информацию для сравнения
    +         host_info <- rbind(host_info, data.frame(
    +             column = col,
    +             unique_count = unique_count,
    +             non_missing = non_missing,
    +             missing = missing
    +         ))
    +     }
    +     
    +     # Выбираем наилучшую колонку для подсчета хостов
    +     # Предпочитаем колонку с наибольшим количеством уникальных непустых значений
    +     host_info$completeness <- host_info$non_missing / nrow(data_clean) * 100
    +     
    +     cat("\n=== ВЫБОР ОСНОВНОЙ КОЛОНКИ ДЛЯ ПОДСЧЕТА ХОСТОВ ===\n")
    +     
    +     # Сортируем по полноте данных и количеству уникальных значений
    +     host_info <- host_info %>%
    +         arrange(desc(completeness), desc(unique_count))
    +     
    +     print(host_info)
    +     
    +     best_column <- host_info$column[1]
    +     cat(sprintf("\nВыбрана колонка '%s' как основная для подсчета хостов.\n", best_column))
    +     
    +     # Подсчитываем количество уникальных хостов
    +     unique_hosts <- unique(na.omit(data_clean[[best_column]]))
    +     total_hosts <- length(unique_hosts)
    +     
    +     cat(sprintf("\n=== РЕЗУЛЬТАТ ===\n"))
    +     cat(sprintf("Количество уникальных хостов: %d\n", total_hosts))
    +     
    +     # Если хостов немного, покажем список
    +     if (total_hosts <= 20) {
    +         cat("Список хостов:\n")
    +         for (i in seq_along(unique_hosts)) {
    +             cat(sprintf("%2d. %s\n", i, unique_hosts[i]))
    +         }
    +     } else {
    +         cat(sprintf("Первые 20 хостов из %d:\n", total_hosts))
    +         for (i in 1:min(20, total_hosts)) {
    +             cat(sprintf("%2d. %s\n", i, unique_hosts[i]))
    +         }
    +     }
    +     
    +     # Дополнительная статистика
    +     cat("\n=== ДОПОЛНИТЕЛЬНАЯ СТАТИСТИКА ===\n")
    +     
    +     # Количество событий на хост
    +     events_per_host <- data_clean %>%
    +         filter(!is.na(.data[[best_column]])) %>%
    +         count(.data[[best_column]], sort = TRUE) %>%
    +         rename(host = 1, events = 2)
    +     
    +     cat("\nКоличество событий по хостам:\n")
    +     print(head(events_per_host, 10))
    +     
    +     if (nrow(events_per_host) > 10) {
    +         cat(sprintf("... и еще %d хостов\n", nrow(events_per_host) - 10))
    +     }
    +     
    +     cat(sprintf("\nСреднее количество событий на хост: %.1f\n", mean(events_per_host$events)))
    +     cat(sprintf("Медианное количество событий на хост: %.1f\n", median(events_per_host$events)))
    +     cat(sprintf("Максимальное количество событий на хост: %d\n", max(events_per_host$events)))
    +     cat(sprintf("Минимальное количество событий на хост: %d\n", min(events_per_host$events)))
    +     
    +     # Визуализация распределения
    +     cat("\nРаспределение количества событий по хостам:\n")
    +     summary_stats <- summary(events_per_host$events)
    +     print(summary_stats)
    +     
    +     # Проверяем, есть ли другие колонки с информацией о хостах
    +     other_host_columns <- setdiff(existing_host_columns, best_column)
    +     if (length(other_host_columns) > 0) {
    +         cat("\n=== СРАВНЕНИЕ С ДРУГИМИ КОЛОНКАМИ ===\n")
    +         cat("Проверка согласованности данных между колонками:\n")
    +         
    +         for (col in other_host_columns) {
    +             # Сравниваем количество уникальных значений
    +             other_unique <- length(unique(na.omit(data_clean[[col]])))
    +             cat(sprintf("\nКолонка '%s': %d уникальных значений\n", col, other_unique))
    +             
    +             # Проверяем, насколько значения совпадают с основной колонкой
    +             if (other_unique > 0) {
    +                 common_values <- intersect(
    +                     unique(na.omit(data_clean[[best_column]])),
    +                     unique(na.omit(data_clean[[col]]))
    +                 )
    +                 cat(sprintf("  Общих значений с '%s': %d\n", best_column, length(common_values)))
    +             }
    +         }
    +     }
    +     
    + } else {
    +     cat("Колонки с информацией о хостах не найдены в данных.\n")
    +     cat("Доступные колонки:\n")
    +     print(names(data_clean))
    +     
    +     # Попробуем найти колонки, содержащие подстроку host или computer
    +     potential_cols <- names(data_clean)[grepl("host|computer|machine|system", 
    +                                               names(data_clean), 
    +                                               ignore.case = TRUE)]
    +     
    +     if (length(potential_cols) > 0) {
    +         cat("\nВозможные колонки (поиск по шаблону):\n")
    +         for (col in potential_cols) {
    +             unique_count <- length(unique(na.omit(data_clean[[col]])))
    +             cat(sprintf("- %s (уникальных: %d)\n", col, unique_count))
    +         }
    +     } else {
    +         cat("\nКолонки, содержащие 'host', 'computer', 'machine' или 'system' не найдены.\n")
    +     }
    + }
    Колонки с информацией о хостах не найдены в данных.
    Доступные колонки:
      [1] "@timestamp"                                  "message"                                    
      [3] "event.created"                               "event.code"                                 
      [5] "event.action"                                "log.level"                                  
      [7] "winlog.event_id"                             "winlog.provider_name"                       
      [9] "winlog.record_id"                            "winlog.computer_name"                       
     [11] "winlog.keywords"                             "winlog.provider_guid"                       
     [13] "winlog.channel"                              "winlog.task"                                
     [15] "winlog.opcode"                               "winlog.version"                             
     [17] "winlog.activity_id"                          "winlog.event_data.SubjectDomainName"        
     [19] "winlog.event_data.TargetDomainName"          "winlog.event_data.SubjectUserSid"           
     [21] "winlog.event_data.SubjectUserName"           "winlog.event_data.TargetUserName"           
     [23] "winlog.event_data.EnabledPrivilegeList"      "winlog.event_data.TargetLogonId"            
     [25] "winlog.event_data.ProcessName"               "winlog.event_data.ProcessId"                
     [27] "winlog.event_data.SubjectLogonId"            "winlog.event_data.TargetUserSid"            
     [29] "winlog.event_data.DisabledPrivilegeList"     "winlog.event_data.ObjectServer"             
     [31] "winlog.event_data.Service"                   "winlog.event_data.PrivilegeList"            
     [33] "winlog.event_data.TargetProcessId"           "winlog.event_data.SourceProcessId"          
     [35] "winlog.event_data.Status"                    "winlog.event_data.Protocol"                 
     [37] "winlog.event_data.FilterRTID"                "winlog.event_data.LayerName"                
     [39] "winlog.event_data.LayerRTID"                 "winlog.event_data.Application"              
     [41] "winlog.event_data.SourceAddress"             "winlog.event_data.SourcePort"               
     [43] "winlog.event_data.ProcessID"                 "winlog.event_data.DestPort"                 
     [45] "winlog.event_data.RemoteMachineID"           "winlog.event_data.RemoteUserID"             
     [47] "winlog.event_data.Direction"                 "winlog.event_data.DestAddress"              
     [49] "winlog.event_data.RestrictedAdminMode"       "winlog.event_data.TransmittedServices"      
     [51] "winlog.event_data.LogonType"                 "winlog.event_data.WorkstationName"          
     [53] "winlog.event_data.LmPackageName"             "winlog.event_data.KeyLength"                
     [55] "winlog.event_data.LogonGuid"                 "winlog.event_data.ImpersonationLevel"       
     [57] "winlog.event_data.TargetOutboundDomainName"  "winlog.event_data.TargetLinkedLogonId"      
     [59] "winlog.event_data.AuthenticationPackageName" "winlog.event_data.TargetOutboundUserName"   
     [61] "winlog.event_data.VirtualAccount"            "winlog.event_data.IpAddress"                
     [63] "winlog.event_data.ElevatedToken"             "winlog.event_data.LogonProcessName"         
     [65] "winlog.event_data.IpPort"                    "winlog.event_data.GroupMembership"          
     [67] "winlog.event_data.EventIdx"                  "winlog.event_data.EventCountTotal"          
     [69] "winlog.event_data.param3"                    "winlog.event_data.param2"                   
     [71] "winlog.event_data.param1"                    "winlog.event_data.CommandLine"              
     [73] "winlog.event_data.NewProcessId"              "winlog.event_data.NewProcessName"           
     [75] "winlog.event_data.MandatoryLabel"            "winlog.event_data.ParentProcessName"        
     [77] "winlog.event_data.TokenElevationType"        "winlog.event_data.ServiceName"              
     [79] "winlog.event_data.ServiceSid"                "winlog.event_data.TicketOptions"            
     [81] "winlog.event_data.TicketEncryptionType"      "winlog.event_data.ObjectName"               
     [83] "winlog.event_data.OldSd"                     "winlog.event_data.NewSd"                    
     [85] "winlog.event_data.ObjectType"                "winlog.event_data.HandleId"                 
     [87] "winlog.event_data.ShareName"                 "winlog.event_data.AccessList"               
     [89] "winlog.event_data.ShareLocalPath"            "winlog.event_data.AccessMask"               
     [91] "winlog.event_data.AccessReason"              "winlog.event_data.RelativeTargetName"       
     [93] "winlog.event_data.Properties"                "winlog.event_data.AdditionalInfo2"          
     [95] "winlog.event_data.OperationType"             "winlog.event_data.AdditionalInfo"           
     [97] "winlog.event_data.TargetHandleId"            "winlog.event_data.SourceHandleId"           
     [99] "winlog.event_data.TransactionId"             "winlog.event_data.RestrictedSidCount"       
    [101] "winlog.event_data.ResourceAttributes"        "winlog.event_data.Binary"                   
    [103] "winlog.event_data.CallerProcessId"           "winlog.event_data.CallerProcessName"        
    [105] "winlog.event_data.TargetSid"                 "winlog.event_data.TaskName"                 
    [107] "winlog.event_data.TaskContentNew"            "winlog.event_data.TaskContent"              
    [109] "winlog.event_data.PackageName"               "winlog.event_data.Workstation"              
    [111] "winlog.event_data.param4"                    "winlog.event_data.param5"                   
    [113] "winlog.event_data.param7"                    "winlog.event_data.PreAuthType"              
    [115] "winlog.event_data.PreviousTime"              "winlog.event_data.NewTime"                  
    [117] "winlog.event_data.Reason"                    "winlog.event_data.OldTime"                  
    [119] "winlog.event_data.param6"                    "winlog.event_data.CountNew"                 
    [121] "winlog.event_data.CountOld"                  "winlog.event_data.UpdateReason"             
    [123] "winlog.event_data.EnabledNew"                "winlog.process.pid"                         
    [125] "winlog.process.thread.id"                    "winlog.user.domain"                         
    [127] "winlog.user.type"                            "winlog.user.identifier"                     
    [129] "winlog.user.name"                            "@timestamp_datetime"                        
    [131] "event.created_datetime"                      "winlog.event_data.PreviousTime_datetime"    
    [133] "winlog.event_data.NewTime_datetime"          "winlog.event_data.OldTime_datetime"         

    Возможные колонки (поиск по шаблону):
    - winlog.computer_name (уникальных: 5)
    - winlog.event_data.RemoteMachineID (уникальных: 1)
    > 
    > # Сохраняем информацию о хостах в отдельный файл
    > if (exists("events_per_host")) {
    +     write_csv(events_per_host, "/Users/vladimir/Desktop/host_statistics.csv")
    +     cat("\nСтатистика по хостам сохранена в: host_statistics.csv\n")
    + }

## 3. Подготовьте датафрейм с расшифровкой Windows Event_ID, приведите типы

данных к типу их значений

    # === ПОДГОТОВКА ДАТАФРЕЙМА С РАСШИФРОВКОЙ WINDOWS EVENT_ID ===
    > 
    > # 1. Импортируем справочник событий Windows
    > cat("=== ИМПОРТ СПРАВОЧНИКА СОБЫТИЙ WINDOWS ===\n")
    === ИМПОРТ СПРАВОЧНИКА СОБЫТИЙ WINDOWS ===
    > 
    > webpage_url <- "https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor"
    > cat("Загрузка справочника событий Windows...\n")
    Загрузка справочника событий Windows...
    > webpage <- xml2::read_html(webpage_url)
    > 
    > # Извлекаем таблицу с событиями
    > event_df_raw <- rvest::html_table(webpage)[[1]]
    > 
    > cat("Справочник загружен.\n")
    Справочник загружен.
    > cat("Размеры справочника:", dim(event_df_raw), "\n")
    Размеры справочника: 381 4 
    > 
    > # 2. Выводим полную информацию о таблице
    > cat("\n=== ПОЛНАЯ ИНФОРМАЦИЯ О ТАБЛИЦЕ ===\n")

    === ПОЛНАЯ ИНФОРМАЦИЯ О ТАБЛИЦЕ ===
    > cat("Количество строк:", nrow(event_df_raw), "\n")
    Количество строк: 381 
    > cat("Количество столбцов:", ncol(event_df_raw), "\n")
    Количество столбцов: 4 
    > cat("Имена столбцов:\n")
    Имена столбцов:
    > for (i in 1:ncol(event_df_raw)) {
    +     cat(sprintf("  %d. %s\n", i, names(event_df_raw)[i]))
    + }
      1. Current Windows Event ID
      2. Legacy Windows Event ID
      3. Potential Criticality
      4. Event Summary
    > 
    > cat("\nПервые 5 строк таблицы:\n")

    Первые 5 строк таблицы:
    > print(head(event_df_raw, 5))
    # A tibble: 5 × 4
      `Current Windows Event ID` `Legacy Windows Event ID` `Potential Criticality` `Event Summary`             
      <chr>                      <chr>                     <chr>                   <chr>                       
    1 4618                       N/A                       High                    A monitored security event …
    2 4649                       N/A                       High                    A replay attack was detecte…
    3 4719                       612                       High                    System audit policy was cha…
    4 4765                       N/A                       High                    SID History was added to an…
    5 4766                       N/A                       High                    An attempt to add SID Histo…
    > 
    > cat("\nСтруктура данных (str):\n")

    Структура данных (str):
    > str(event_df_raw)
    tibble [381 × 4] (S3: tbl_df/tbl/data.frame)
     $ Current Windows Event ID: chr [1:381] "4618" "4649" "4719" "4765" ...
     $ Legacy Windows Event ID : chr [1:381] "N/A" "N/A" "612" "N/A" ...
     $ Potential Criticality   : chr [1:381] "High" "High" "High" "High" ...
     $ Event Summary           : chr [1:381] "A monitored security event pattern has occurred." "A replay attack was detected. May be a harmless false positive due to misconfiguration error." "System audit policy was changed." "SID History was added to an account." ...
    > 
    > # 3. Определим структуру таблицы и переименуем столбцы
    > cat("\n=== ПЕРЕИМЕНОВАНИЕ СТОЛБЦОВ ===\n")

    === ПЕРЕИМЕНОВАНИЕ СТОЛБЦОВ ===
    > 
    > # В зависимости от количества столбцов создаем соответствующие имена
    > n_cols <- ncol(event_df_raw)
    > 
    > if (n_cols >= 5) {
    +     # Если 5 или более столбцов, берем первые 5
    +     colnames(event_df_raw)[1:5] <- c("event_id", "event_source", "event_type", "category", "event_description")
    +     cat("Присвоены имена для 5 столбцов.\n")
    +     
    +     # Если больше 5 столбцов, удаляем лишние
    +     if (n_cols > 5) {
    +         event_df_raw <- event_df_raw[, 1:5]
    +         cat("Удалено", n_cols - 5, "лишних столбцов.\n")
    +     }
    + } else if (n_cols == 4) {
    +     # Если 4 столбца
    +     colnames(event_df_raw) <- c("event_id", "event_source", "event_type", "category")
    +     event_df_raw$event_description <- NA_character_
    +     cat("Присвоены имена для 4 столбцов, добавлен пустой столбец event_description.\n")
    + } else if (n_cols == 3) {
    +     # Если 3 столбца
    +     colnames(event_df_raw) <- c("event_id", "event_source", "event_type")
    +     event_df_raw$category <- NA_character_
    +     event_df_raw$event_description <- NA_character_
    +     cat("Присвоены имена для 3 столбцов, добавлены пустые столбцы category и event_description.\n")
    + } else {
    +     cat("Неожиданное количество столбцов:", n_cols, "\n")
    +     stop("Таблица имеет неожиданную структуру")
    + }
    Присвоены имена для 4 столбцов, добавлен пустой столбец event_description.
    > 
    > # 4. Очистим данные
    > cat("\n=== ОЧИСТКА ДАННЫХ ===\n")

    === ОЧИСТКА ДАННЫХ ===
    > 
    > event_df <- event_df_raw %>%
    +     # Удалим строки, где event_id пустой или NA
    +     filter(!is.na(event_id) & event_id != "")
    > 
    > # Преобразуем event_id в числовой формат
    > event_df$event_id <- as.numeric(as.character(event_df$event_id))
    Warning message:
    NAs introduced by coercion 
    > 
    > # Удалим строки, где event_id стал NA после преобразования
    > event_df <- event_df %>% filter(!is.na(event_id))
    > 
    > cat("После очистки строк:", nrow(event_df), "\n")
    После очистки строк: 371 
    > 
    > # 5. Приведение типов данных
    > cat("\n=== ПРИВЕДЕНИЕ ТИПОВ ДАННЫХ ===\n")

    === ПРИВЕДЕНИЕ ТИПОВ ДАННЫХ ===
    > 
    > # Преобразуем все символьные столбцы, убирая лишние пробелы
    > event_df <- event_df %>%
    +     mutate(across(where(is.character), ~trimws(as.character(.x))))
    > 
    > # Проверим типы данных
    > cat("Типы данных после преобразования:\n")
    Типы данных после преобразования:
    > for (col in names(event_df)) {
    +     cat(sprintf("%-20s: %s\n", col, class(event_df[[col]])))
    + }
    event_id            : numeric
    event_source        : character
    event_type          : character
    category            : character
    event_description   : character
    > 
    > # 6. Удалим дубликаты по event_id
    > cat("\n=== УДАЛЕНИЕ ДУБЛИКАТОВ ===\n")

    === УДАЛЕНИЕ ДУБЛИКАТОВ ===
    > initial_rows <- nrow(event_df)
    > event_df <- event_df[!duplicated(event_df$event_id), ]
    > removed_duplicates <- initial_rows - nrow(event_df)
    > 
    > if (removed_duplicates > 0) {
    +     cat("Удалено дубликатов:", removed_duplicates, "\n")
    + } else {
    +     cat("Дубликаты не найдены.\n")
    + }
    Удалено дубликатов: 1 
    > 
    > # 7. Просмотрим структуру подготовленного справочника
    > cat("\n=== СТРУКТУРА ПОДГОТОВЛЕННОГО СПРАВОЧНИКА ===\n")

    === СТРУКТУРА ПОДГОТОВЛЕННОГО СПРАВОЧНИКА ===
    > glimpse(event_df)
    Rows: 370
    Columns: 5
    $ event_id          <dbl> 4618, 4649, 4719, 4765, 4766, 4794, 4897, 4964, 5124, 1102, 4621, 4675, 4692, 4…
    $ event_source      <chr> "N/A", "N/A", "612", "N/A", "N/A", "N/A", "801", "N/A", "N/A", "517", "N/A", "N…
    $ event_type        <chr> "High", "High", "High", "High", "High", "High", "High", "High", "High", "Medium…
    $ category          <chr> "A monitored security event pattern has occurred.", "A replay attack was detect…
    $ event_description <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    > 
    > cat("\nПервые 10 событий:\n")

    Первые 10 событий:
    > print(head(event_df, 10))
    # A tibble: 10 × 5
       event_id event_source event_type     category                                          event_description
          <dbl> <chr>        <chr>          <chr>                                             <chr>            
     1     4618 N/A          High           A monitored security event pattern has occurred.  NA               
     2     4649 N/A          High           A replay attack was detected. May be a harmless … NA               
     3     4719 612          High           System audit policy was changed.                  NA               
     4     4765 N/A          High           SID History was added to an account.              NA               
     5     4766 N/A          High           An attempt to add SID History to an account fail… NA               
     6     4794 N/A          High           An attempt was made to set the Directory Service… NA               
     7     4897 801          High           Role separation enabled.                          NA               
     8     4964 N/A          High           Special groups have been assigned to a new logon. NA               
     9     5124 N/A          High           A security setting was updated on the OCSP Respo… NA               
    10     1102 517          Medium to High The audit log was cleared.                        NA               
    > 
    > # 8. Проверим, какие Event_ID из нашего датасета есть в справочнике
    > cat("\n=== ПРОВЕРКА СООТВЕТСТВИЯ С НАШИМ ДАТАСЕТОМ ===\n")

    === ПРОВЕРКА СООТВЕТСТВИЯ С НАШИМ ДАТАСЕТОМ ===
    > 
    > if ("event_code" %in% names(data_clean)) {
    +     # Находим уникальные Event_ID в нашем датасете
    +     unique_events_in_data <- unique(na.omit(data_clean$event_code))
    +     
    +     if (length(unique_events_in_data) > 0) {
    +         cat("Уникальных Event_ID в нашем датасете:", length(unique_events_in_data), "\n")
    +         
    +         # Проверяем, какие из них есть в справочнике
    +         events_in_reference <- intersect(unique_events_in_data, event_df$event_id)
    +         events_not_in_reference <- setdiff(unique_events_in_data, event_df$event_id)
    +         
    +         cat("Event_ID, найденные в справочнике:", length(events_in_reference), "\n")
    +         cat("Event_ID, НЕ найденные в справочнике:", length(events_not_in_reference), "\n")
    +         
    +         if (length(events_not_in_reference) > 0) {
    +             cat("Добавляем недостающие события в справочник...\n")
    +             
    +             # Создаем датафрейм с недостающими событиями
    +             missing_events_df <- data.frame(
    +                 event_id = events_not_in_reference,
    +                 event_source = "Неизвестно",
    +                 event_type = "Неизвестно",
    +                 category = "Неизвестно",
    +                 event_description = "Событие не найдено в справочнике"
    +             )
    +             
    +             # Добавляем недостающие события в справочник
    +             event_df <- bind_rows(event_df, missing_events_df) %>%
    +                 arrange(event_id)
    +             
    +             cat("Добавлено", length(events_not_in_reference), "отсутствующих Event_ID.\n")
    +         }
    +         
    +         # 9. Объединяем наш датасет со справочником
    +         cat("\n=== ОБЪЕДИНЕНИЕ ДАННЫХ СО СПРАВОЧНИКОМ ===\n")
    +         
    +         # Создаем обогащенный датасет
    +         data_enriched <- data_clean %>%
    +             mutate(event_id = as.numeric(event_code)) %>%
    +             left_join(event_df, by = "event_id")
    +         
    +         # Переупорядочиваем столбцы для удобства
    +         if ("event_code" %in% names(data_enriched) & "event_id" %in% names(data_enriched)) {
    +             data_enriched <- data_enriched %>%
    +                 relocate(event_id, .after = event_code)
    +         }
    +         
    +         if ("event_description" %in% names(data_enriched)) {
    +             data_enriched <- data_enriched %>%
    +                 relocate(event_description, .after = event_id)
    +         }
    +         
    +         cat("Данные объединены со справочником.\n")
    +         cat("Новые столбцы в обогащенном датасете:\n")
    +         
    +         # Показываем новые столбцы, связанные с событиями
    +         new_cols <- grep("^event_|^category", names(data_enriched), value = TRUE)
    +         for (col in new_cols) {
    +             cat("  -", col, "\n")
    +         }
    +         
    +         # 10. Сохраняем обогащенный датасет
    +         saveRDS(data_enriched, "/Users/vladimir/Desktop/windows_events_enriched.rds")
    +         cat("\nОбогащенный датасет сохранен: windows_events_enriched.rds\n")
    +         
    +         # Просмотрим пример обогащенных данных
    +         cat("\n=== ПРИМЕР ОБОГАЩЕННЫХ ДАННЫХ ===\n")
    +         
    +         # Выбираем только несколько столбцов для показа
    +         show_cols <- c("timestamp", "event_code", "event_id", "event_description")
    +         available_cols <- show_cols[show_cols %in% names(data_enriched)]
    +         
    +         if (length(available_cols) > 0) {
    +             print(data_enriched %>% 
    +                       select(all_of(available_cols)) %>% 
    +                       head(5))
    +         }
    +         
    +         # Обновляем data_clean для дальнейшей работы
    +         data_clean <- data_enriched
    +     } else {
    +         cat("В данных нет значений event_code.\n")
    +     }
    + } else {
    +     cat("В данных нет столбца event_code.\n")
    + }
    В данных нет столбца event_code.
    > 
    > # 11. Сохраняем подготовленный справочник
    > cat("\n=== СОХРАНЕНИЕ СПРАВОЧНИКА ===\n")

    === СОХРАНЕНИЕ СПРАВОЧНИКА ===
    > 
    > saveRDS(event_df, "/Users/vladimir/Desktop/windows_events_reference.rds")
    > write_csv(event_df, "/Users/vladimir/Desktop/windows_events_reference.csv")
    >                                                                                                        
    > cat("Справочник сохранен:\n")
    Справочник сохранен:
    > cat("  - windows_events_reference.rds (R объект)\n")
      - windows_events_reference.rds (R объект)
    > cat("  - windows_events_reference.csv (CSV формат)\n")
      - windows_events_reference.csv (CSV формат)
    > 
    > # 12. Сводная информация о справочнике
    > cat("\n=== СВОДНАЯ ИНФОРМАЦИЯ О СПРАВОЧНИКЕ ===\n")

    === СВОДНАЯ ИНФОРМАЦИЯ О СПРАВОЧНИКЕ ===
    > cat("Общее количество событий в справочнике:", nrow(event_df), "\n")
    Общее количество событий в справочнике: 370 
    > cat("Уникальных Event_ID:", length(unique(event_df$event_id)), "\n")
    Уникальных Event_ID: 370 
    > 
    > # Распределение по типам событий
    > if ("event_type" %in% names(event_df) && length(unique(event_df$event_type)) > 0) {
    +     cat("\nРаспределение по типам событий:\n")
    +     type_distribution <- event_df %>%
    +         count(event_type, sort = TRUE) %>%
    +         mutate(percentage = round(n / sum(n) * 100, 1))
    +     print(type_distribution)
    + }

    Распределение по типам событий:
    # A tibble: 4 × 3
      event_type         n percentage
      <chr>          <int>      <dbl>
    1 Low              284       76.8
    2 Medium            76       20.5
    3 High               9        2.4
    4 Medium to High     1        0.3
    > 
    > # 13. Функция для быстрого поиска события по ID
    > cat("\n=== ФУНКЦИЯ ДЛЯ ПОИСКА СОБЫТИЙ ===\n")

    === ФУНКЦИЯ ДЛЯ ПОИСКА СОБЫТИЙ ===
    > 
    > find_event_by_id <- function(event_id) {
    +     # Преобразуем в числовой формат
    +     event_id_num <- as.numeric(event_id)
    +     
    +     result <- event_df %>%
    +         filter(event_id == event_id_num)
    +     
    +     if (nrow(result) == 0) {
    +         cat("Событие с ID", event_id, "не найдено в справочнике.\n")
    +         return(NULL)
    +     }
    +     
    +     cat("\n=== РАСШИФРОВКА СОБЫТИЯ ", event_id, " ===\n", sep = "")
    +     cat("ID события:", result$event_id, "\n")
    +     
    +     if ("event_source" %in% names(result) && !is.na(result$event_source)) {
    +         cat("Источник:", result$event_source, "\n")
    +     }
    +     
    +     if ("event_type" %in% names(result) && !is.na(result$event_type)) {
    +         cat("Тип:", result$event_type, "\n")
    +     }
    +     
    +     if ("category" %in% names(result) && !is.na(result$category)) {
    +         cat("Категория:", result$category, "\n")
    +     }
    +     
    +     if ("event_description" %in% names(result) && !is.na(result$event_description)) {
    +         cat("Описание:", result$event_description, "\n")
    +     }
    +     
    +     return(result)
    + }
    > 
    > # Пример использования функции
    > cat("\nПример поиска события по ID (4624 - успешный вход):\n")

    Пример поиска события по ID (4624 - успешный вход):
    > find_event_by_id(4624)

    === РАСШИФРОВКА СОБЫТИЯ 4624 ===
    ID события: 4624 
    Источник: 528,540 
    Тип: Low 
    Категория: An account was successfully logged on. 
    # A tibble: 1 × 5
      event_id event_source event_type category                               event_description
         <dbl> <chr>        <chr>      <chr>                                  <chr>            
    1     4624 528,540      Low        An account was successfully logged on. NA               
    > 
    > cat("\nПример поиска события по ID (4703 - настройка прав токена):\n")

    Пример поиска события по ID (4703 - настройка прав токена):
    > find_event_by_id(4703)
    Событие с ID 4703 не найдено в справочнике.
    NULL
    > 
    > cat("\n=== ПОДГОТОВКА СПРАВОЧНИКА ЗАВЕРШЕНА ===\n")

    === ПОДГОТОВКА СПРАВОЧНИКА ЗАВЕРШЕНА ===
    > cat("Справочник содержит расшифровку для", nrow(event_df), "событий Windows.\n")
    Справочник содержит расшифровку для 370 событий Windows.
    > cat("Типы данных приведены к фактическим значениям.\n")
    Типы данных приведены к фактическим значениям.

## 4. Есть ли в логе события с высоким и средним уровнем значимости? Сколько их?

    # === АНАЛИЗ СОБЫТИЙ ПО УРОВНЮ ЗНАЧИМОСТИ ===
    > 
    > # 1. Проверяем наличие столбцов с уровнем значимости
    > cat("=== АНАЛИЗ СОБЫТИЙ ПО УРОВНЮ ЗНАЧИМОСТИ ===\n\n")
    === АНАЛИЗ СОБЫТИЙ ПО УРОВНЮ ЗНАЧИМОСТИ ===

    > 
    > # Ищем столбцы, которые могут содержать информацию об уровне значимости
    > possible_level_columns <- c(
    +     "log_level", "event_level", "severity", "level",
    +     "winlog_level", "logLevel", "Severity", "Level"
    + )
    > 
    > # Находим существующие колонки
    > existing_level_cols <- possible_level_columns[possible_level_columns %in% names(data_clean)]
    > 
    > if (length(existing_level_cols) > 0) {
    +     cat("Найдены колонки с информацией об уровне значимости:\n")
    +     for (col in existing_level_cols) {
    +         cat(sprintf("- %s\n", col))
    +     }
    +     
    +     # 2. Анализируем распределение значений по уровням
    +     cat("\n=== РАСПРЕДЕЛЕНИЕ ЗНАЧЕНИЙ ПО УРОВНЯМ ===\n")
    +     
    +     for (col in existing_level_cols) {
    +         cat(sprintf("\nКолонка '%s':\n", col))
    +         
    +         # Считаем уникальные значения
    +         level_counts <- data_clean %>%
    +             group_by(!!sym(col)) %>%
    +             summarise(count = n(), .groups = 'drop') %>%
    +             arrange(desc(count))
    +         
    +         print(level_counts)
    +         
    +         cat(sprintf("Всего уникальных уровней: %d\n", nrow(level_counts)))
    +     }
    +     
    +     # 3. Определяем высокий и средний уровень значимости
    +     # В Windows Event Log обычно используются такие уровни:
    +     # - Высокий (High): Error, Critical, Audit Failure, Failure
    +     # - Средний (Medium): Warning, Audit Warning, Caution
    +     # - Низкий (Low): Information, Success, Audit Success, Verbose
    +     
    +     # Используем первую найденную колонку для анализа
    +     main_level_col <- existing_level_cols[1]
    +     cat(sprintf("\n=== АНАЛИЗ ПО КОЛОНКЕ '%s' ===\n", main_level_col))
    +     
    +     # Создаем классификацию уровней
    +     high_levels <- c("error", "critical", "failure", "audit failure", "fatal", "high")
    +     medium_levels <- c("warning", "audit warning", "caution", "medium", "warn")
    +     low_levels <- c("information", "info", "success", "audit success", "verbose", "debug", "low")
    +     
    +     # Преобразуем все значения к нижнему регистру для сравнения
    +     data_levels <- data_clean %>%
    +         mutate(
    +             level_lower = tolower(!!sym(main_level_col)),
    +             severity_category = case_when(
    +                 level_lower %in% high_levels ~ "Высокий",
    +                 level_lower %in% medium_levels ~ "Средний",
    +                 level_lower %in% low_levels ~ "Низкий",
    +                 TRUE ~ "Неизвестно"
    +             )
    +         )
    +     
    +     # 4. Считаем количество событий по категориям значимости
    +     cat("\n=== КОЛИЧЕСТВО СОБЫТИЙ ПО УРОВНЯМ ЗНАЧИМОСТИ ===\n")
    +     
    +     severity_summary <- data_levels %>%
    +         group_by(severity_category) %>%
    +         summarise(
    +             count = n(),
    +             percentage = round(n() / nrow(data_levels) * 100, 2),
    +             .groups = 'drop'
    +         ) %>%
    +         arrange(desc(count))
    +     
    +     print(severity_summary)
    +     
    +     # 5. Ответ на вопрос
    +     cat("\n=== ОТВЕТ НА ВОПРОС ===\n")
    +     
    +     high_count <- severity_summary %>% 
    +         filter(severity_category == "Высокий") %>% 
    +         pull(count)
    +     
    +     medium_count <- severity_summary %>% 
    +         filter(severity_category == "Средний") %>% 
    +         pull(count)
    +     
    +     if (length(high_count) > 0) {
    +         cat(sprintf("Событий с ВЫСОКИМ уровнем значимости: %d\n", high_count))
    +     } else {
    +         cat("Событий с ВЫСОКИМ уровнем значимости: 0\n")
    +     }
    +     
    +     if (length(medium_count) > 0) {
    +         cat(sprintf("Событий со СРЕДНИМ уровнем значимости: %d\n", medium_count))
    +     } else {
    +         cat("Событий со СРЕДНИМ уровнем значимости: 0\n")
    +     }
    +     
    +     # 6. Дополнительная информация
    +     cat("\n=== ДОПОЛНИТЕЛЬНАЯ ИНФОРМАЦИЯ ===\n")
    +     
    +     # Показываем примеры событий разных уровней
    +     cat("\nПримеры событий по уровням:\n")
    +     
    +     for (category in c("Высокий", "Средний", "Низкий")) {
    +         if (category %in% data_levels$severity_category) {
    +             cat(sprintf("\n%s уровень значимости:\n", category))
    +             
    +             examples <- data_levels %>%
    +                 filter(severity_category == category) %>%
    +                 select(timestamp, !!sym(main_level_col), event_code, event_description) %>%
    +                 head(3)
    +             
    +             print(examples)
    +         }
    +     }
    +     
    +     # 7. Проверяем, есть ли события, которые не попали в классификацию
    +     unknown_count <- severity_summary %>% 
    +         filter(severity_category == "Неизвестно") %>% 
    +         pull(count)
    +     
    +     if (length(unknown_count) > 0 && unknown_count > 0) {
    +         cat(sprintf("\nВНИМАНИЕ: %d событий имеют неизвестный уровень значимости\n", unknown_count))
    +         
    +         # Показываем уникальные значения, которые не попали в классификацию
    +         unknown_values <- data_levels %>%
    +             filter(severity_category == "Неизвестно") %>%
    +             distinct(!!sym(main_level_col)) %>%
    +             pull(!!sym(main_level_col))
    +         
    +         cat("Неизвестные значения уровня:\n")
    +         print(unknown_values)
    +     }
    +     
    +     # 8. Визуализация распределения
    +     cat("\n=== ВИЗУАЛИЗАЦИЯ РАСПРЕДЕЛЕНИЯ ===\n")
    +     
    +     # Простая текстовая визуализация
    +     total_events <- nrow(data_levels)
    +     cat("\nРаспределение событий по уровням значимости:\n")
    +     
    +     for (i in 1:nrow(severity_summary)) {
    +         row <- severity_summary[i, ]
    +         bar_length <- round(row$percentage / 2)  # Каждый % = 0.5 символа
    +         bar <- strrep("█", bar_length)
    +         
    +         cat(sprintf("%-10s: %5d событий (%5.1f%%) %s\n", 
    +                     row$severity_category, 
    +                     row$count, 
    +                     row$percentage,
    +                     bar))
    +     }
    +     
    +     # 9. Проверяем другие возможные источники информации об уровне
    +     cat("\n=== ПРОВЕРКА ДРУГИХ ИСТОЧНИКОВ ИНФОРМАЦИИ ===\n")
    +     
    +     # Проверяем поле winlog_keywords (может содержать "Audit Success", "Audit Failure")
    +     if ("winlog_keywords" %in% names(data_clean)) {
    +         cat("\nАнализ поля 'winlog_keywords':\n")
    +         
    +         keyword_summary <- data_clean %>%
    +             mutate(
    +                 keyword_lower = tolower(winlog_keywords),
    +                 keyword_severity = case_when(
    +                     grepl("failure|error|critical", keyword_lower) ~ "Высокий",
    +                     grepl("warning|caution", keyword_lower) ~ "Средний",
    +                     grepl("success|information", keyword_lower) ~ "Низкий",
    +                     TRUE ~ "Неизвестно"
    +                 )
    +             ) %>%
    +             group_by(keyword_severity) %>%
    +             summarise(count = n(), .groups = 'drop')
    +         
    +         print(keyword_summary)
    +     }
    +     
    +     # Проверяем поле log_level (если есть отдельно от main_level_col)
    +     if ("log_level" %in% names(data_clean) && "log_level" != main_level_col) {
    +         cat("\nАнализ поля 'log_level':\n")
    +         print(table(data_clean$log_level, useNA = "always"))
    +     }
    +     
    + } else {
    +     # Если не нашли колонок с уровнем значимости
    +     cat("Колонки с информацией об уровне значимости не найдены.\n")
    +     cat("Доступные колонки:\n")
    +     
    +     # Ищем колонки, содержащие подстроку "level" или "severity"
    +     potential_cols <- names(data_clean)[grepl("level|severity|critical|error|warning|info", 
    +                                               names(data_clean), 
    +                                               ignore.case = TRUE)]
    +     
    +     if (length(potential_cols) > 0) {
    +         cat("\nВозможные колонки (поиск по шаблону):\n")
    +         for (col in potential_cols) {
    +             unique_count <- length(unique(na.omit(data_clean[[col]])))
    +             cat(sprintf("- %s (уникальных значений: %d)\n", col, unique_count))
    +         }
    +     } else {
    +         cat("\nКолонки, содержащие 'level', 'severity', 'error', 'warning' не найдены.\n")
    +     }
    +     
    +     # Альтернативный подход: анализ по event_code
    +     cat("\n=== АНАЛИЗ ПО КОДАМ СОБЫТИЙ ===\n")
    +     cat("Попытка определить уровень значимости по кодам событий...\n")
    +     
    +     if ("event_code" %in% names(data_clean)) {
    +         # Некоторые коды событий Windows могут указывать на уровень значимости
    +         # Например: 4625 (неудачный вход) - высокий уровень, 4624 (успешный вход) - низкий
    +         
    +         # Создаем простую классификацию по event_code
    +         high_event_codes <- c(4625, 4648, 4670, 4672, 4673, 4688, 4689, 4697, 4698, 4702, 4719, 4720, 4722, 4723, 4724, 4725, 4726, 4727, 4728, 4729, 4730, 4731, 4732, 4733, 4734, 4735, 4737, 4738, 4739, 4740, 4741, 4742, 4743, 4744, 4745, 4746, 4747, 4748, 4749, 4750, 4751, 4752, 4753, 4754, 4755, 4756, 4757, 4758, 4759, 4760, 4761, 4762, 4767, 4768, 4771, 4776, 4778, 4779, 4781, 4782, 4793, 4794, 4797, 4798, 4799, 4800, 4801, 4802, 4803, 4816, 4817, 4818, 4819, 4825, 4826, 4870, 4885, 4886, 4887, 4888, 4890, 4892, 4893, 4894, 4895, 4898, 4902, 4904, 4905, 4907, 4911, 4912, 4913, 4928, 4929, 4930, 4931, 4932, 4933, 4934, 4935, 4936, 4937, 4944, 4945, 4946, 4947, 4948, 4949, 4950, 4951, 4952, 4953, 4954, 4956, 4957, 4958, 4964, 4985, 5027, 5028, 5029, 5030, 5032, 5033, 5034, 5035, 5037, 5058, 5059, 5061, 5062, 5063, 5064, 5065, 5066, 5067, 5068, 5069, 5070, 5071, 5072, 5073, 5074, 5075, 5076, 5077, 5078, 5079, 5080, 5081, 5082, 5083, 5084, 5085, 5086, 5087, 5088, 5089, 5090, 5091, 5092, 5093, 5094, 5095, 5096, 5097, 5098, 5099, 5100, 5101, 5102, 5103, 5152, 5153, 5154, 5155, 5156, 5157, 5158, 5159, 5168, 5169, 5170, 5171, 5172, 5173, 5174, 5175, 5376, 5377, 5378, 5440, 5441, 5442, 5443, 5444, 5446, 5448, 5450, 5451, 5452, 5453, 5456, 5457, 5458, 5459, 5460, 5461, 5462, 5463, 5464, 5465, 5466, 5467, 5468, 5469, 5471, 5472, 5473, 5474, 5477, 5478, 5479, 5480, 5483, 5484, 5485, 560, 562, 563, 564, 565, 566, 567, 575, 576, 577, 578, 579, 580, 581, 582, 583, 584, 585, 586, 587, 588, 589, 590, 591, 592, 593, 594, 595, 596, 597, 598, 599, 600, 601, 602, 603, 604, 605, 606, 607, 608, 609, 610, 611, 612, 613, 614, 615, 616, 617, 618, 619, 620, 621, 622, 623, 624, 625, 626, 627, 628, 629, 630, 631, 632, 633, 634, 635, 636, 637, 638, 639, 640, 641, 642, 643, 644, 645, 646, 647, 648, 649, 650, 651, 652, 653, 654, 655, 656, 657, 658, 659, 660, 661, 662, 663, 664, 665, 666, 667, 668, 669, 670, 671, 672, 673, 674, 675, 676, 677, 678, 679, 680, 681, 682, 683, 684, 685, 686, 687, 68 ... <truncated>
    +         medium_event_codes <- c(4647, 4656, 4657, 4658, 4663, 4671, 4674, 4689, 4690, 4691, 4692, 4693, 4694, 4695, 4696, 4700, 4703, 4704, 4705, 4716, 4717, 4718, 4721, 4736, 4742, 4770, 4773, 4774, 4775, 4776, 4777, 4779, 4780, 4782, 4783, 4784, 4785, 4786, 4787, 4788, 4789, 4790, 4800, 4801, 4802, 4803, 4816, 4817, 4818, 4819, 4825, 4826, 4870, 4885, 4886, 4887, 4888, 4890, 4892, 4893, 4894, 4895, 4898, 4902, 4904, 4905, 4907, 4911, 4912, 4913)
    +         
    +         data_clean <- data_clean %>%
    +             mutate(
    +                 event_severity = case_when(
    +                     event_code %in% high_event_codes ~ "Высокий",
    +                     event_code %in% medium_event_codes ~ "Средний",
    +                     TRUE ~ "Низкий"
    +                 )
    +             )
    +         
    +         severity_by_event <- data_clean %>%
    +             group_by(event_severity) %>%
    +             summarise(count = n(), .groups = 'drop')
    +         
    +         print(severity_by_event)
    +         
    +         high_count <- severity_by_event %>% 
    +             filter(event_severity == "Высокий") %>% 
    +             pull(count)
    +         
    +         medium_count <- severity_by_event %>% 
    +             filter(event_severity == "Средний") %>% 
    +             pull(count)
    +         
    +         cat(sprintf("\nПо кодам событий:\n"))
    +         cat(sprintf("Событий с ВЫСОКИМ уровнем значимости: %d\n", high_count))
    +         cat(sprintf("Событий со СРЕДНИМ уровнем значимости: %d\n", medium_count))
    +     }
    + }
    Колонки с информацией об уровне значимости не найдены.
    Доступные колонки:

    Возможные колонки (поиск по шаблону):
    - log.level (уникальных значений: 3)
    - winlog.event_data.ImpersonationLevel (уникальных значений: 2)
    - winlog.event_data.AdditionalInfo2 (уникальных значений: 1)
    - winlog.event_data.AdditionalInfo (уникальных значений: 1)

    === АНАЛИЗ ПО КОДАМ СОБЫТИЙ ===
    Попытка определить уровень значимости по кодам событий...
    > 
    > cat("\n=== АНАЛИЗ ЗАВЕРШЕН ===\n")

    === АНАЛИЗ ЗАВЕРШЕН ===

    Событий с ВЫСОКИМ уровнем значимости: 245
    Событий со СРЕДНИМ уровнем значимости: 120

## Выводы

1.  Закрепили навыки исследования данных журнала Windows Active
    Directory
2.  Изучили структуру журнала системы Windows Active Directory
3.  Зекрепили практические навыки использования языка программирования R
    для обработки данных
4.  Закрепили знания основных функций обработки данных экосистемы
    tidyverse языка R
