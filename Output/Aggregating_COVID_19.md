---
title: "Aggregating_Covid_19"
author: "Bbosa Robert"
date: "29/04/2021"
output:
  word_document: 
    toc: yes
    fig_caption: yes
    keep_md: yes
  html_document: default
---

# Pulling the coronvirus data from John Hopkins repo https://github.com/CSSEGISandData/COVID-19

## Pulling confirmed cases


```r
conf_url <- "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/
master/csse_covid_19_data/csse_covid_19_time_series/
time_series_covid19_confirmed_global.csv"

raw_conf <- read.csv(file = conf_url, stringsAsFactors = FALSE, header = TRUE)

lapply(1:ncol(raw_conf), function(i){
  if(all(is.na(raw_conf[, i]))){
    raw_conf <<- raw_conf[, -i]
    return(print(paste("Column", names(raw_conf)[i], "is missing", sep = " ")))
  } else {
    return(NULL)
  }
})
```

## Pulling death cases


```r
death_url <- "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master
/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv"

raw_death <- read.csv(file =death_url, stringsAsFactors = FALSE, fill =FALSE)

lapply(1:ncol(raw_death), function(i){
  if(all(is.na(raw_death[, i]))){
    raw_death <<- raw_death[, -i]
    return(print(paste("Column", names(raw_death)[i], "is missing", sep = " ")))
  } else {
    return(NULL)
  }
})
```

## Pulling recovered cases


```r
raw_rec <- read.csv(file = "https://raw.githubusercontent.com/CSSEGISandData/COVID-19
/master/csse_covid_19_data/csse_covid_19_time_series
/time_series_covid19_recovered_global.csv", stringsAsFactors = FALSE, fill =FALSE)

lapply(1:ncol(raw_rec), function(i){
  if(all(is.na(raw_rec[, i]))){
    raw_rec <<- raw_rec[, -i]
    return(print(paste("Column", names(raw_rec)[i], "is missing", sep = " ")))
  } else {
    return(NULL)
  }
})
```



# Transforming the data from wide to long.
## Creating new data frames for confirmed cases

```r
library(tidyr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(readr)

df_conf <- raw_conf[, 1:4]

for(i in 5:ncol(raw_conf)){

  raw_conf[,i] <- as.integer(raw_conf[,i])
  # raw_conf[,i] <- ifelse(is.na(raw_conf[, i]), 0 , raw_conf[, i])
    print(names(raw_conf)[i])

  if(i == 5){
    df_conf[[names(raw_conf)[i]]] <- raw_conf[, i]
  } 
    else {
    df_conf[[names(raw_conf)[i]]] <- raw_conf[, i] - raw_conf[, i - 1]
  }

}


df_conf1 <-  df_conf %>% tidyr::pivot_longer(cols = dplyr::starts_with("X"),
                                             names_to = "date_temp",
                                             values_to = "cases_temp")

df_conf1$date_temp <- sub("X","", df_conf1$date_temp)
df_conf1$date_temp <- as.Date(df_conf1$date_temp, format='%m.%d.%y')
```


## Creating new data frame for death cases


```r
df_death <- raw_death[, 1:4]

for(i in 5:ncol(raw_death)){
  print(i)
  raw_death[,i] <- as.integer(raw_death[,i])
  raw_death[,i] <- ifelse(is.na(raw_death[, i]), 0 , raw_death[, i])

  if(i == 5){
    df_death[[names(raw_death)[i]]] <- raw_death[, i]
  } else {
    df_death[[names(raw_death)[i]]] <- raw_death[, i] - raw_death[, i - 1]
  }
}


df_death1 <-  df_death %>% tidyr::pivot_longer(cols = dplyr::starts_with("X"),
                                               names_to = "date_temp",
                                               values_to = "cases_temp")

df_death1$date_temp <- sub("X","", df_death1$date_temp)
df_death1$date_temp <- as.Date(df_death1$date_temp, format='%m.%d.%y')
```

## Creating new data frame for recovered cases


```r
df_rec <- raw_rec[, 1:4]

for(i in 5:ncol(raw_rec)){
  print(i)
  raw_rec[,i] <- as.integer(raw_rec[,i])
  raw_rec[,i] <- ifelse(is.na(raw_rec[, i]), 0 , raw_rec[, i])

  if(i == 5){
    df_rec[[names(raw_rec)[i]]] <- raw_rec[, i]
  } else {
    df_rec[[names(raw_rec)[i]]] <- raw_rec[, i] - raw_rec[, i - 1]
  }
}


df_rec1 <-  df_rec %>% tidyr::pivot_longer(cols = dplyr::starts_with("X"),
                                           names_to = "date_temp",
                                           values_to = "cases_temp")

df_rec1$date_temp <- sub("X","", df_rec1$date_temp)
df_rec1$date_temp <- as.Date(df_rec1$date_temp, format='%m.%d.%y')
```



# Aggregate the data

## Aggregate daily data for confirmed cases


```r
#Rename the variable for the date
names(df_conf1)[names(df_conf1)=="date_temp"]<-"date"

#Then aggregate the data
df_conf2 <- df_conf1 %>%
  dplyr::group_by(Province.State, Country.Region, Lat, Long, date) %>%
  dplyr::summarise(cases = sum(cases_temp)) %>%
  dplyr::ungroup() %>%
  dplyr::mutate(type = "confirmed",
                Country.Region = trimws(Country.Region),
                Province.State = trimws(Province.State))
```

```
## `summarise()` regrouping output by 'Province.State', 'Country.Region', 'Lat', 'Long' (override with `.groups` argument)
```

```r
head(df_conf2)
```

```
## # A tibble: 6 x 7
##   Province.State Country.Region   Lat  Long date       cases type     
##   <chr>          <chr>          <dbl> <dbl> <date>     <int> <chr>    
## 1 ""             Afghanistan     33.9  67.7 2020-01-22     0 confirmed
## 2 ""             Afghanistan     33.9  67.7 2020-01-23     0 confirmed
## 3 ""             Afghanistan     33.9  67.7 2020-01-24     0 confirmed
## 4 ""             Afghanistan     33.9  67.7 2020-01-25     0 confirmed
## 5 ""             Afghanistan     33.9  67.7 2020-01-26     0 confirmed
## 6 ""             Afghanistan     33.9  67.7 2020-01-27     0 confirmed
```

```r
tail(df_conf2)
```

```
## # A tibble: 6 x 7
##   Province.State Country.Region   Lat  Long date       cases type     
##   <chr>          <chr>          <dbl> <dbl> <date>     <int> <chr>    
## 1 Zhejiang       China           29.2  120. 2021-04-23     0 confirmed
## 2 Zhejiang       China           29.2  120. 2021-04-24     0 confirmed
## 3 Zhejiang       China           29.2  120. 2021-04-25     0 confirmed
## 4 Zhejiang       China           29.2  120. 2021-04-26     1 confirmed
## 5 Zhejiang       China           29.2  120. 2021-04-27     0 confirmed
## 6 Zhejiang       China           29.2  120. 2021-04-28    11 confirmed
```

## Aggregate daily data for death cases


```r
#Rename the variable for the date
names(df_death1)[names(df_death1)=="date_temp"]<-"date"

df_death2 <- df_death1 %>%
  dplyr::group_by(Province.State, Country.Region, Lat, Long, date) %>%
  dplyr::summarise(cases = sum(cases_temp)) %>%
  dplyr::ungroup() %>%
  dplyr::mutate(type = "death",
                Country.Region = trimws(Country.Region),
                Province.State = trimws(Province.State))
```

```
## `summarise()` regrouping output by 'Province.State', 'Country.Region', 'Lat', 'Long' (override with `.groups` argument)
```

```r
head(df_death2)
```

```
## # A tibble: 6 x 7
##   Province.State Country.Region   Lat  Long date       cases type 
##   <chr>          <chr>          <dbl> <dbl> <date>     <int> <chr>
## 1 ""             Afghanistan     33.9  67.7 2020-01-22     0 death
## 2 ""             Afghanistan     33.9  67.7 2020-01-23     0 death
## 3 ""             Afghanistan     33.9  67.7 2020-01-24     0 death
## 4 ""             Afghanistan     33.9  67.7 2020-01-25     0 death
## 5 ""             Afghanistan     33.9  67.7 2020-01-26     0 death
## 6 ""             Afghanistan     33.9  67.7 2020-01-27     0 death
```

```r
tail(df_death2)
```

```
## # A tibble: 6 x 7
##   Province.State Country.Region   Lat  Long date       cases type 
##   <chr>          <chr>          <dbl> <dbl> <date>     <int> <chr>
## 1 Zhejiang       China           29.2  120. 2021-04-23     0 death
## 2 Zhejiang       China           29.2  120. 2021-04-24     0 death
## 3 Zhejiang       China           29.2  120. 2021-04-25     0 death
## 4 Zhejiang       China           29.2  120. 2021-04-26     0 death
## 5 Zhejiang       China           29.2  120. 2021-04-27     0 death
## 6 Zhejiang       China           29.2  120. 2021-04-28     0 death
```

## Aggregate daily data for recovered cases


```r
#Rename the variable for the date
names(df_rec1)[names(df_rec1)=="date_temp"]<-"date"

df_rec2 <- df_rec1 %>%
  dplyr::group_by(Province.State, Country.Region, Lat, Long, date) %>%
  dplyr::summarise(cases = sum(cases_temp)) %>%
  dplyr::ungroup() %>%
  dplyr::mutate(type = "recovered",
                Country.Region = trimws(Country.Region),
                Province.State = trimws(Province.State))
```

```
## `summarise()` regrouping output by 'Province.State', 'Country.Region', 'Lat', 'Long' (override with `.groups` argument)
```

```r
head(df_rec2)
```

```
## # A tibble: 6 x 7
##   Province.State Country.Region   Lat  Long date       cases type     
##   <chr>          <chr>          <dbl> <dbl> <date>     <int> <chr>    
## 1 ""             Afghanistan     33.9  67.7 2020-01-22     0 recovered
## 2 ""             Afghanistan     33.9  67.7 2020-01-23     0 recovered
## 3 ""             Afghanistan     33.9  67.7 2020-01-24     0 recovered
## 4 ""             Afghanistan     33.9  67.7 2020-01-25     0 recovered
## 5 ""             Afghanistan     33.9  67.7 2020-01-26     0 recovered
## 6 ""             Afghanistan     33.9  67.7 2020-01-27     0 recovered
```

```r
tail(df_rec2)
```

```
## # A tibble: 6 x 7
##   Province.State Country.Region   Lat  Long date       cases type     
##   <chr>          <chr>          <dbl> <dbl> <date>     <int> <chr>    
## 1 Zhejiang       China           29.2  120. 2021-04-23     0 recovered
## 2 Zhejiang       China           29.2  120. 2021-04-24     0 recovered
## 3 Zhejiang       China           29.2  120. 2021-04-25     0 recovered
## 4 Zhejiang       China           29.2  120. 2021-04-26     0 recovered
## 5 Zhejiang       China           29.2  120. 2021-04-27     0 recovered
## 6 Zhejiang       China           29.2  120. 2021-04-28     2 recovered
```

# Aggregate all the data frames into one.


```r
coronavirus <- dplyr::bind_rows(df_conf2, df_death2, df_rec2) %>% as.data.frame()
```

# Export the coronavirus into a working directory


```r
write.csv(coronavirus,"C:/Users/uganda/OneDrive - BBOSA ROBERT/COVID-19/COVID-19_Data/Data/coronavirus.csv", row.names = FALSE)

writexl::write_xlsx(x = coronavirus, path = "C:/Users/uganda/OneDrive - BBOSA ROBERT/COVID-19/COVID-19_Data/Data/coronavirus.xlsx", col_names = TRUE)
```
