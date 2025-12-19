# Lab2
vov41234567890@yandex.ru
2025-11-18

# Цели работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных

2.  Закрепить знания базовых типов данных языка R

3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

## Исходные данные

1.  Ноутбук с ОС MacOS
2.  RStudio
3.  Интерпретатор языка R 4.5.1

## Общий план выполнения работы

1.  Проанализировать встроенный в пакет dplyr набор данных starwars
2.  Выполнить аталитические задачи с помощью языка R

## Атализ встроенного пакета dplyr инабора данных starwars

``` r
library(dplyr)
```


    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

## Проверка кол-ва строк в датафрейме

``` r
starwars |> nrow()
```

    [1] 87

## Проверка кол-ва столбцов в датафрейме

``` r
starwars |> ncol()
```

    [1] 14

## Просмотр примерного вида датафрейма

``` r
starwars %>% glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the J…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

## Просмотр уникальных рас персонажей

``` r
starwars %>% summarise(unique_species = n_distinct(species))
```

    # A tibble: 1 × 1
      unique_species
               <int>
    1             38

## Поиск самого высокого персонажа

``` r
starwars %>%   filter(height == max(height, na.rm = TRUE))
```

    # A tibble: 1 × 14
      name      height  mass hair_color skin_color eye_color birth_year sex   gender
      <chr>      <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
    1 Yarael P…    264    NA none       white      yellow            NA male  mascu…
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

## Перечень персонажей меньше 170

``` r
starwars %>%   filter(height < 170)
```

    # A tibble: 22 × 14
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 C-3PO       167    75 <NA>       gold       yellow           112 none  mascu…
     2 R2-D2        96    32 <NA>       white, bl… red               33 none  mascu…
     3 Leia Or…    150    49 brown      light      brown             19 fema… femin…
     4 Beru Wh…    165    75 brown      light      blue              47 fema… femin…
     5 R5-D4        97    32 <NA>       white, red red               NA none  mascu…
     6 Yoda         66    17 white      green      brown            896 male  mascu…
     7 Mon Mot…    150    NA auburn     fair       blue              48 fema… femin…
     8 Wicket …     88    20 brown      brown      brown              8 male  mascu…
     9 Nien Nu…    160    68 none       grey       black             NA male  mascu…
    10 Watto       137    NA black      blue, grey yellow            NA male  mascu…
    # ℹ 12 more rows
    # ℹ 5 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>

## Подсчитать ИМТ

``` r
starwars %>% mutate(BMI = mass / ( (height/100)^2 ))
```

    # A tibble: 87 × 15
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Luke Sk…    172    77 blond      fair       blue            19   male  mascu…
     2 C-3PO       167    75 <NA>       gold       yellow         112   none  mascu…
     3 R2-D2        96    32 <NA>       white, bl… red             33   none  mascu…
     4 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     5 Leia Or…    150    49 brown      light      brown           19   fema… femin…
     6 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     7 Beru Wh…    165    75 brown      light      blue            47   fema… femin…
     8 R5-D4        97    32 <NA>       white, red red             NA   none  mascu…
     9 Biggs D…    183    84 black      light      brown           24   male  mascu…
    10 Obi-Wan…    182    77 auburn, w… fair       blue-gray       57   male  mascu…
    # ℹ 77 more rows
    # ℹ 6 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>, BMI <dbl>

## Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

``` r
 starwars %>% mutate(stretch = mass / height) 
```

    # A tibble: 87 × 15
       name     height  mass hair_color skin_color eye_color birth_year sex   gender
       <chr>     <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
     1 Luke Sk…    172    77 blond      fair       blue            19   male  mascu…
     2 C-3PO       167    75 <NA>       gold       yellow         112   none  mascu…
     3 R2-D2        96    32 <NA>       white, bl… red             33   none  mascu…
     4 Darth V…    202   136 none       white      yellow          41.9 male  mascu…
     5 Leia Or…    150    49 brown      light      brown           19   fema… femin…
     6 Owen La…    178   120 brown, gr… light      blue            52   male  mascu…
     7 Beru Wh…    165    75 brown      light      blue            47   fema… femin…
     8 R5-D4        97    32 <NA>       white, red red             NA   none  mascu…
     9 Biggs D…    183    84 black      light      brown           24   male  mascu…
    10 Obi-Wan…    182    77 auburn, w… fair       blue-gray       57   male  mascu…
    # ℹ 77 more rows
    # ℹ 6 more variables: homeworld <chr>, species <chr>, films <list>,
    #   vehicles <list>, starships <list>, stretch <dbl>

## Нахождение средего возраста персонажей каждой расы вселенной Звездных войн

``` r
starwars %>% mutate(age = 2025 - birth_year) %>%
     group_by(species) %>%
     summarise(
         avg_age = mean(age, na.rm = TRUE)
     ) %>%
     arrange(desc(avg_age))
```

    # A tibble: 38 × 2
       species      avg_age
       <chr>          <dbl>
     1 Ewok           2017 
     2 Kel Dor        2003 
     3 Mon Calamari   1984 
     4 Rodian         1981 
     5 Twi'lek        1977 
     6 Mirialan       1976 
     7 Gungan         1973 
     8 Trandoshan     1972 
     9 Droid          1972.
    10 Human          1971.
    # ℹ 28 more rows

## Нахождение самый распространенный цвет глаз персонажей вселенной Звездных войн.

``` r
starwars %>%
     filter(!is.na(eye_color)) %>%
     group_by(eye_color) %>%
     summarise(
         count = n()
     ) %>%
     arrange(desc(count)) %>%
     slice(1)
```

    # A tibble: 1 × 2
      eye_color count
      <chr>     <int>
    1 brown        21

## Подсчет средней длины имени в каждой расе вселенной Звездных войн.

``` r
starwars %>%
     filter(!is.na(species) & !is.na(name)) %>%   
     group_by(species) %>%                        
     summarise(
         avg_name_length = mean(nchar(name))       
     ) %>%
     arrange(desc(avg_name_length))  
```

    # A tibble: 37 × 2
       species   avg_name_length
       <chr>               <dbl>
     1 Ewok                 21  
     2 Hutt                 21  
     3 Geonosian            17  
     4 Besalisk             15  
     5 Mirialan             14  
     6 Toong                14  
     7 Aleena               12  
     8 Cerean               12  
     9 Gungan               11.7
    10 Human                11.3
    # ℹ 27 more rows

## Оценка результатов

В ходе лабораторной работы были успешно развиты практические навыки
обработки данных на языке R с использованием функций пакета dplyr. Все
задачи по анализу встроенного датасета starwars, включая фильтрацию,
трансформацию и агрегацию данных, выполнены корректно и в полном объеме.

## ВЫВОДЫ:

1.  Развиты практические навыки использования языка программирования R
    для обработки данных
2.  Закреплены знания базовых типов данных языка R
3.  Развиты практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()
