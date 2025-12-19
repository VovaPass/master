# Lab1
vov41234567890@yandex.ru
2025-11-18

# Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Развить навыки работы в Rstudio IDE: установка пакетов работа с
    проектами в Rstudio настройка и работа с Git
3.  Закрепить знания базовых типов данных языка R и простейших операций
    с ними

# Исходные данные

1.  Ноутбук с ОС MacOS
2.  RStudio
3.  Интерпретатор языка R 4.5.1

# Общий план выполнения работы

1.  Установить интерпретатор R
2.  Установить Rstudio IDE
3.  Установить программный пакет swirl:
4.  Запустить задание с помощью swirl::swirl()
5.  Выбрать из меню курсов R Programming: The basics of programming in R
    и выполнить:
    -   базовые структурные блоки (Basic Building Blocks)
    -   рабочие пространства и файлы (Workspace and Files)
    -   последовательности чисел (Sequences of Numbers)
    -   векторы (Vectors)
    -   пропущенные значения (Missing Values)

# Шаги

## 1. Установка интерпретатора R Studio

Скачиваем установщик по адресу https://posit.co/download/rstudio-desktop
и устанавливаем R Studio

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

## 2. Установка программного пакета swirl

# Установка пакета swirl с указанием зеркала CRAN для России

``` r
r <- getOption("repos")
r["CRAN"] <- "https://mirror.truenetwork.ru/CRAN/"
options(repos = r)
install.packages("swirl")
```


    The downloaded binary packages are in
        /var/folders/lg/2xmw4mvd0zj8v_t4qmtb_lmm0000gn/T//RtmpMVVUZs/downloaded_packages

## 3. Запуск задания через swirl

    {swirl::swirl()}

## Курс базовые структурные блоки (Basic Building Blocks)

### Присвоение значения х

``` r
x <- 5 + 7
```

### Изучаем синтактсис и записываем пример

``` r
y <- x - 3 
```

### Присвоение значений переменной

``` r
z <- c(1.1, 9, 3.14)
```

### Запросим хелпу команды

``` r
?c
```

### Вывод содержимого

``` r
 z
```

    [1] 1.10 9.00 3.14

### Объединение векторов

``` r
c(z, 555, z) 
```

    [1]   1.10   9.00   3.14 555.00   1.10   9.00   3.14

### Числовые вектора в арифметических выражениях

``` r
z * 2 + 100 
```

    [1] 102.20 118.00 106.28

### Возьмем квадратный корень из z - 1 и присвем его новой переменной

``` r
my_sqrt <- sqrt(z - 1)
```

### Выведем что находится в my_sqrt.

``` r
my_sqrt  
```

    [1] 0.3162278 2.8284271 1.4628739

### Создаем новую переменную которая равноа значению z деленное на my_sqrt.

``` r
 my_div <- z/my_sqrt
```

### Выведем что находится в my_div.

``` r
my_div 
```

    [1] 3.478505 3.181981 2.146460

### Сложим вектора

``` r
c(1, 2, 3, 4) + c(0, 10) 
```

    [1]  1 12  3 14

``` r
c(1, 2, 3, 4) + c(0, 10, 100)
```

    Warning in c(1, 2, 3, 4) + c(0, 10, 100): longer object length is not a
    multiple of shorter object length

    [1]   1  12 103   4

## Курс рабочие пространства и файлы (Workspace and Files)

### Определим рабочий каталог

``` r
 getwd() 
```

    [1] "/Users/vladimir/Documents/GitHub/master/pr_1"

### Выведем объекты в рабочем пространстве

``` r
 ls()  
```

    [1] "my_div"  "my_sqrt" "r"       "x"       "y"       "z"      

### Присвоение значения

``` r
 x <- 9
```

### Выведем список всех файлов в рабочем каталоге

``` r
 list.files()  
```

    [1] "lab_1.qmd"       "lab_1.rmarkdown" "Lab1_files"      "mytest2.R"      
    [5] "mytest3.R"       "README.md"       "testdir"         "testdir2"       

### Вывелем хелпу по команде

``` r
 ?list.files
```

### Используем функцию args() для определения аргументов list.files().

``` r
args(list.files) 
```

    function (path = ".", pattern = NULL, all.files = FALSE, full.names = FALSE, 
        recursive = FALSE, ignore.case = FALSE, include.dirs = FALSE, 
        no.. = FALSE) 
    NULL

### Присвоем значение текущего рабочего каталога переменной

``` r
old.dir <- getwd()
```

### Создание каталога

``` r
dir.create("testdir")
```

    Warning in dir.create("testdir"): 'testdir' already exists

### Поменяем рабочий каталог

``` r
setwd("testdir")
```

### Создание нового файла

``` r
file.create("mytest.R")
```

    [1] TRUE

### Провекра каталога

``` r
 list.files() 
```

    [1] "lab_1.qmd"       "lab_1.rmarkdown" "Lab1_files"      "mytest.R"       
    [5] "mytest2.R"       "mytest3.R"       "README.md"       "testdir"        
    [9] "testdir2"       

### Проверка наличия в каталоге файла

``` r
file.exists("mytest.R") 
```

    [1] TRUE

### Проверим информацию о файле

``` r
file.info("mytest.R") 
```

             size isdir mode               mtime               ctime
    mytest.R    0 FALSE  644 2025-12-19 21:25:42 2025-12-19 21:25:42
                           atime uid gid    uname grname
    mytest.R 2025-12-19 21:25:42 501  20 vladimir  staff

### Поменяем имя файла

``` r
file.rename("mytest.R", "mytest2.R") 
```

    [1] TRUE

### Сделать копию файла

``` r
file.copy("mytest2.R", "mytest3.R")
```

    [1] FALSE

### Проверим путь к файлу

``` r
file.path("mytest3.R") 
```

    [1] "mytest3.R"

``` r
file.path("folder1", "folder2") 
```

    [1] "folder1/folder2"

### Выводим хелпу команды

``` r
?dir.create
```

### Создадим директорию

``` r
 dir.create(file.path('testdir2', 'testdir3'), recursive = TRUE)
```

    Warning in dir.create(file.path("testdir2", "testdir3"), recursive = TRUE):
    'testdir2/testdir3' already exists

### Сменим директорию

``` r
 setwd(old.dir)
```

## Курс последовательности чисел (Sequences of Numbers)

### Создание последовательности чисел различными способами

``` r
1:20 
```

     [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20

``` r
pi:10 
```

    [1] 3.141593 4.141593 5.141593 6.141593 7.141593 8.141593 9.141593

``` r
15:1 
```

     [1] 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1

``` r
?`:`
```

``` r
seq(1, 20) 
```

     [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20

``` r
seq(0, 10, by=0.5) 
```

     [1]  0.0  0.5  1.0  1.5  2.0  2.5  3.0  3.5  4.0  4.5  5.0  5.5  6.0  6.5  7.0
    [16]  7.5  8.0  8.5  9.0  9.5 10.0

``` r
my_seq <- seq(5, 10, length=30)
```

``` r
length(my_seq) 
```

    [1] 30

``` r
1:length(my_seq) 
```

     [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
    [26] 26 27 28 29 30

``` r
seq(along.with = my_seq) 
```

     [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
    [26] 26 27 28 29 30

``` r
seq_along(my_seq) 
```

     [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
    [26] 26 27 28 29 30

``` r
rep(0, times = 40) 
```

     [1] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
    [39] 0 0

``` r
rep(c(0, 1, 2), times = 10) 
```

     [1] 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2

``` r
rep(c(0, 1, 2), each = 10) 
```

     [1] 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2

## Курс векторы (Vectors)

### Cоздание числового вектора различными способами

``` r
num_vect <- c(0.5, 55, -10, 6)
```

``` r
tf <- num_vect < 1
```

``` r
num_vect >= 6 
```

    [1] FALSE  TRUE FALSE  TRUE

### Занесем значения в переменную с дальнейшем выводом результата различными способами(работа с символами)

``` r
my_char <- c("My", "name", "is")
```

``` r
paste(my_char, collapse = " ") 
```

    [1] "My name is"

``` r
my_name <- c(my_char, "Vladimir")
```

``` r
paste(my_name, collapse = " ") 
```

    [1] "My name is Vladimir"

``` r
paste("Hello","world!", sep = " ") 
```

    [1] "Hello world!"

``` r
paste(1:3, c("X", "Y", "Z"), sep = "")
```

    [1] "1X" "2Y" "3Z"

``` r
paste(LETTERS, 1:4, sep = "-")
```

     [1] "A-1" "B-2" "C-3" "D-4" "E-1" "F-2" "G-3" "H-4" "I-1" "J-2" "K-3" "L-4"
    [13] "M-1" "N-2" "O-3" "P-4" "Q-1" "R-2" "S-3" "T-4" "U-1" "V-2" "W-3" "X-4"
    [25] "Y-1" "Z-2"

## Курс пропущенные значения (Missing Values)

``` r
x <- c(44, NA, 5, NA)
```

``` r
x * 3 
```

    [1] 132  NA  15  NA

``` r
y <- rnorm(1000)
```

``` r
z <- rep(NA, 1000)
```

``` r
my_data <- sample(c(y, z), 100)
```

``` r
my_na <- is.na(my_data)
```

``` r
my_na 
```

      [1] FALSE  TRUE  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE
     [13]  TRUE FALSE  TRUE FALSE  TRUE  TRUE FALSE  TRUE FALSE  TRUE  TRUE  TRUE
     [25]  TRUE FALSE  TRUE  TRUE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE  TRUE
     [37] FALSE FALSE  TRUE  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE FALSE FALSE
     [49] FALSE  TRUE  TRUE  TRUE  TRUE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE
     [61] FALSE FALSE  TRUE FALSE FALSE FALSE  TRUE FALSE  TRUE  TRUE  TRUE  TRUE
     [73] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE FALSE FALSE
     [85] FALSE FALSE FALSE  TRUE FALSE  TRUE FALSE FALSE  TRUE FALSE  TRUE FALSE
     [97] FALSE  TRUE  TRUE  TRUE

``` r
my_data 
```

      [1] -0.01596890          NA          NA  0.10612345          NA -2.74613943
      [7] -0.59156891          NA -0.53057013  0.52628118  1.24765760          NA
     [13]          NA -0.26358845          NA  0.64454893          NA          NA
     [19]  0.90697866          NA -0.39906468          NA          NA          NA
     [25]          NA  1.30621244          NA          NA -1.96590012          NA
     [31] -0.05029268  1.51813014 -0.73664147          NA  0.93303866          NA
     [37] -0.19856607  2.33111424          NA          NA -2.33365122          NA
     [43]          NA          NA          NA          NA -0.46119687  0.53982973
     [49]  1.33029311          NA          NA          NA          NA -0.76520300
     [55]  1.45724054  0.38492169          NA          NA -0.30096603 -0.92730050
     [61]  0.03610027 -1.25087763          NA -1.36092753 -0.68228691 -1.30171913
     [67]          NA -0.70412412          NA          NA          NA          NA
     [73] -1.44489137 -1.10284366 -0.49927092  1.98098617  0.03558054  1.73047965
     [79] -0.70576590 -0.09311535          NA          NA  0.82988537 -0.05085466
     [85] -0.33283142 -1.23960926 -0.11416481          NA -0.04580919          NA
     [91]  0.35184471 -1.72049917          NA -0.10471823          NA -0.55762313
     [97] -0.73614021          NA          NA          NA

``` r
Inf - Inf 
```

    [1] NaN

# Оценка результатов

-   Изучены основы языка R: работа с векторами, последовательностями,
    пропущенными значениями и файловой системой.
-   Приобретены навыки использования Rmarkdown для создания отчётов.
-   Работа загружена в репозиторий GitHub.

# Вывод

В результате выполнения практической работы 1 усвоены базовые операции,
работа с векторами, последовательностями, пропущенными значениями и
файловой системой. Получены практические навыки работы в RStudio с
основными типами данных и функциями языка R.
