---
title: "Lab 5"
author: "Lili Xu"
date: "9/24/2021"
output: 
    html_document:
      toc: yes 
      toc_float: yes 
      highlight: haddock
      theme: cosmo
      keep_md: yes
    github_document:
      keep_html: false
      html_preview: false
always_allow_html: true
---



## Download the data



```r
library(data.table)
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──
```

```
## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
## ✓ tibble  3.1.0     ✓ dplyr   1.0.5
## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
## ✓ readr   1.4.0     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::between()   masks data.table::between()
## x dplyr::filter()    masks stats::filter()
## x dplyr::first()     masks data.table::first()
## x dplyr::lag()       masks stats::lag()
## x dplyr::last()      masks data.table::last()
## x purrr::transpose() masks data.table::transpose()
```

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:data.table':
## 
##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
##     yday, year
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```

```r
library(leaflet)
```


```r
if (!file.exists("../met_all.gz"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz",
    destfile = "met_all.gz",
    method = "libcurl",
    timeout = 60
  )
met <- data.table::fread("../met_all.gz")

# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

```
## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion
```

```r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```

Merging the Met dataset and the Station dataset

```r
met <- merge(
  # Data
  x     = met,      
  y     = stations, 
  # List of variables to match
  by.x  = "USAFID",
  by.y  = "USAF", 
  # Which obs to keep?
  all.x = TRUE,      
  all.y = FALSE
  )
```

## Question 1: Representative station for the US

What is the median station in terms of temperature, wind speed, and atmospheric pressure? Look for the three weather stations that best represent continental US using the quantile() function. Do these three coincide?

First, generate a representative version of each station. We will use the averages (median could be a good way to represent it, but it will depend on the case).

```r
station_average <- met[,.(
  temp = mean(temp, na.rm=TRUE),
  wind.sp = mean(wind.sp, na.rm=TRUE),
  atm.press = mean(atm.press, na.rm=TRUE),
  lon = mean(lon, na.rm = TRUE),
  lat = mean(lat, na.rm = TRUE)
), by = .(USAFID)]
```

Now, we need to identify the median per variable.

```r
medians <- station_average[,.(
  temp_50 = quantile(temp, probs = .5, na.rm = TRUE),
  wind.sp_50 = quantile(wind.sp, probs = .5, na.rm = TRUE),
  atm.press_50 = quantile(atm.press, probs = .5, na.rm = TRUE)
)]
```
Now, we can find the stations that are the closest to these. (hint: `which.min()`)


```r
station_average[, temp_dist := abs(temp - medians$temp_50)]
median_temp_station <- station_average[order(temp_dist)][1] 

knitr::kable(median_temp_station)
```



| USAFID|     temp|  wind.sp| atm.press|     lon|    lat| temp_dist|
|------:|--------:|--------:|---------:|-------:|------:|---------:|
| 720458| 23.68173| 1.209682|       NaN| -82.637| 37.751| 0.0023289|

The median temperature station is 720458.


```r
station_average[, wind.sp_dist := abs(wind.sp - medians$wind.sp_50)]
median_wind.sp_station <- station_average[order(wind.sp_dist)][1] 

knitr::kable(median_wind.sp_station)
```



| USAFID|     temp|  wind.sp| atm.press|     lon|    lat| temp_dist| wind.sp_dist|
|------:|--------:|--------:|---------:|-------:|------:|---------:|------------:|
| 720929| 17.43277| 2.461838|       NaN| -91.981| 45.506|  6.251284|            0|

The median wind speed station is 720929.


```r
station_average[, atm.press_dist := abs(atm.press - medians$atm.press_50)]
median_atm.press_station <- station_average[order(atm.press_dist)][1] 

knitr::kable(median_atm.press_station)
```



| USAFID|     temp|  wind.sp| atm.press|       lon|     lat| temp_dist| wind.sp_dist| atm.press_dist|
|------:|--------:|--------:|---------:|---------:|-------:|---------:|------------:|--------------:|
| 722238| 26.13978| 1.472656|  1014.691| -85.66667| 31.3499|  2.455719|    0.9891817|      0.0005376|

The median atmospheric pressure station is 722238.

These stations of temperature, wind.speed, atmospheric pressure are not coincide.


## Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the most representative, the median, station per state. This time, instead of looking at one variable at a time, look at the euclidean distance. If multiple stations show in the median, select the one located at the lowest latitude.

We first need to recover the state variable, be merging :)!

```r
station_average <- merge(
  x = station_average, y = stations, 
  by.x  = "USAFID",
  by.y  = "USAF", 
  all.x = TRUE, all.y = FALSE
  )

station_average <- station_average[,.(
  temp = mean(temp, na.rm=TRUE),
  wind.sp = mean(wind.sp, na.rm=TRUE),
  atm.press = mean(atm.press, na.rm=TRUE),
  lon = mean(lon, na.rm = TRUE),
  lat = mean(lat, na.rm = TRUE),
  STATE = STATE
), by = .(USAFID)]
```

Now we can compute the median per state

```r
station_average[, temp_50 := quantile(temp, probs = .5, na.rm = TRUE), by = STATE]
station_average[, wind.sp_50 := quantile(wind.sp, probs = .5, na.rm = TRUE), by = STATE]
station_average[, atm.press_50 := quantile(atm.press, probs = .5, na.rm = TRUE), by = STATE]
```

Now, the euclidean distance ... $\sqrt{\sum_i(x_i - y_i)^2}$


```r
station_average[, eudist := sqrt(
  (temp - temp_50)^2 + (wind.sp - wind.sp_50)^2 + (atm.press - atm.press_50)^2
  )]

station_state <- station_average[ , .SD[which.min(eudist)], by = STATE]

# station_average
knitr::kable(station_state)
```



|STATE | USAFID|     temp|  wind.sp| atm.press|        lon|      lat|  temp_50| wind.sp_50| atm.press_50|    eudist|
|:-----|------:|--------:|--------:|---------:|----------:|--------:|--------:|----------:|------------:|---------:|
|CA    | 722970| 22.76040| 2.325982|  1012.710| -118.14652| 33.81264| 22.66268|   2.565446|     1012.557| 0.3004951|
|TX    | 722416| 29.75394| 3.539980|  1012.331|  -98.04599| 29.70899| 29.75188|   3.413737|     1012.460| 0.1802934|
|MI    | 725395| 20.44096| 2.357275|  1015.245|  -84.46697| 42.26697| 20.51970|   2.273423|     1014.927| 0.3387562|
|SC    | 723190| 25.73726| 2.253408|  1015.116|  -82.71000| 34.49800| 25.80545|   1.696119|     1015.281| 0.5852964|
|IL    | 725440| 22.84806| 2.566829|  1014.760|  -90.52032| 41.46325| 22.43194|   2.237622|     1014.760| 0.5305934|
|MO    | 723495| 24.31621| 2.550940|  1014.296|  -94.49501| 37.15200| 23.95109|   2.453547|     1014.522| 0.4404840|
|AR    | 723407| 25.86949| 2.208652|  1014.575|  -90.64600| 35.83100| 26.24296|   1.938625|     1014.591| 0.4611299|
|OR    | 725895| 18.79793| 2.307326|  1014.726| -121.72405| 42.14705| 17.98061|   2.011436|     1015.269| 1.0252745|
|GA    | 723160| 26.59746| 1.684538|  1014.985|  -82.50700| 31.53600| 26.70404|   1.495596|     1015.208| 0.3115758|
|MN    | 726550| 19.11831| 2.832794|  1015.319|  -94.05102| 45.54301| 19.63017|   2.617071|     1015.042| 0.6209640|
|AL    | 722286| 26.35793| 1.675828|  1014.909|  -87.61600| 33.21200| 26.33664|   1.662132|     1014.959| 0.0560838|
|IN    | 725327| 22.40044| 2.547951|  1015.145|  -87.00600| 41.45300| 22.25059|   2.344333|     1015.063| 0.2657731|
|NC    | 723174| 24.95288| 1.744838|  1015.350|  -79.47700| 36.04700| 24.72953|   1.627306|     1015.420| 0.2621319|
|VA    | 724016| 24.29327| 1.588105|  1014.946|  -78.45499| 38.13701| 24.37799|   1.653032|     1015.158| 0.2366533|
|IA    | 725480| 21.43686| 2.764312|  1014.814|  -92.40088| 42.55358| 21.33461|   2.680875|     1014.964| 0.1992693|
|PA    | 725130| 21.69177| 1.970192|  1015.125|  -75.72500| 41.33380| 21.69177|   1.784167|     1015.435| 0.3623458|
|NE    | 725560| 21.80411| 3.428358|  1014.386|  -97.43479| 41.98568| 21.87354|   3.192539|     1014.332| 0.2515990|
|ID    | 725867| 20.81272| 2.702517|  1012.802| -113.76605| 42.54201| 20.56798|   2.568944|     1012.855| 0.2837769|
|WI    | 726452| 19.21728| 2.411747|  1015.180|  -89.83701| 44.35900| 18.85524|   2.053283|     1014.893| 0.5844788|
|WV    | 724176| 21.94072| 1.649151|  1015.982|  -79.91600| 39.64300| 21.94446|   1.633487|     1015.762| 0.2208248|
|MD    | 724057| 25.00877| 2.033233|  1014.497|  -76.16996| 39.47174| 24.89883|   1.883499|     1014.824| 0.3763051|
|AZ    | 722745| 30.31538| 3.307632|  1010.144| -110.88300| 32.16695| 30.32372|   3.074359|     1010.144| 0.2334219|
|OK    | 723545| 27.03555| 3.852697|  1012.711|  -97.08896| 36.16199| 27.14427|   3.852697|     1012.567| 0.1805246|
|WY    | 726650| 19.75554| 4.243727|  1013.527| -105.54099| 44.33905| 19.80699|   3.873392|     1013.157| 0.5264904|
|LA    | 722486| 28.16413| 1.592840|  1014.544|  -92.04098| 32.51596| 27.87430|   1.592840|     1014.593| 0.2939969|
|KY    | 724240| 23.79463| 2.450704|  1015.375|  -85.96723| 37.90032| 23.88844|   1.895486|     1015.245| 0.5778636|
|FL    | 722106| 27.52774| 2.711121|  1015.322|  -81.86101| 26.58501| 27.57325|   2.705069|     1015.335| 0.0477234|
|CO    | 724767| 21.97732| 2.780364|  1014.082| -108.62600| 37.30699| 21.49638|   3.098777|     1013.334| 0.9442283|
|OH    | 724298| 21.79537| 2.771958|  1015.248|  -84.02700| 40.70800| 22.02062|   2.554397|     1015.351| 0.3296961|
|NJ    | 724090| 23.47238| 2.148606|  1015.095|  -74.35016| 40.03300| 23.47238|   2.148606|     1014.825| 0.2697149|
|NM    | 722686| 26.00522| 4.503610|  1012.742| -103.31565| 34.38358| 24.94447|   3.776083|     1012.525| 1.3043763|
|KS    | 724580| 24.01181| 3.548029|  1013.449|  -97.65090| 39.55090| 24.21220|   3.680613|     1013.389| 0.2475134|
|VT    | 726115| 18.60548| 1.101301|  1014.985|  -72.51800| 43.34400| 18.61379|   1.408247|     1014.792| 0.3626106|
|MS    | 722358| 26.54093| 1.747426|  1014.722|  -90.47100| 31.18298| 26.69258|   1.636392|     1014.836| 0.2196615|
|CT    | 725087| 22.57539| 2.126514|  1014.534|  -72.65098| 41.73601| 22.36880|   2.101801|     1014.810| 0.3463514|
|NV    | 725805| 25.21743| 3.101560|  1012.461| -118.56898| 40.06799| 24.56293|   3.035050|     1012.204| 0.7062378|
|UT    | 725755| 24.31031| 3.361211|  1012.243| -111.96637| 41.11737| 24.35182|   3.145427|     1011.972| 0.3492381|
|SD    | 726590| 19.95928| 3.550722|  1014.284|  -98.41344| 45.44377| 20.35662|   3.665638|     1014.398| 0.4291087|
|TN    | 723346| 24.59407| 1.493532|  1015.144|  -88.91700| 35.59302| 24.88657|   1.576035|     1015.144| 0.3039125|
|NY    | 725194| 20.37207| 2.444051|  1015.327|  -77.05599| 42.64299| 20.40674|   2.304075|     1014.887| 0.4625700|
|RI    | 725079| 22.27697| 2.583469|  1014.620|  -71.28300| 41.53300| 22.53551|   2.583469|     1014.728| 0.2803959|
|MA    | 725064| 21.40933| 2.786213|  1014.721|  -70.72900| 41.91000| 21.30662|   2.710943|     1014.751| 0.1308438|
|DE    | 724180| 24.56026| 2.752930|  1015.046|  -75.60600| 39.67400| 24.56026|   2.752930|     1015.046| 0.0000000|
|NH    | 726050| 19.86188| 1.732752|  1014.487|  -71.50245| 43.20409| 19.55054|   1.563826|     1014.689| 0.4077850|
|ME    | 726077| 18.49969| 2.337241|  1014.475|  -68.36677| 44.45000| 18.79016|   2.237210|     1014.399| 0.3165330|
|MT    | 726798| 19.47014| 4.445783|  1014.072| -110.44004| 45.69800| 19.15492|   4.151737|     1014.185| 0.4458281|

This above table shows the most representative, the median, station per state at the lowest latitude.

## Question 3: In the middle?
For each state, identify what is the station that is closest to the mid-point of the state. Combining these with the stations you identified in the previous question, use leaflet() to visualize all ~100 points in the same figure, applying different colors for those identified in this question.


```r
# Compute the median per state for lon and lat
station_average[, lat_50 := quantile(lat, probs = .5, na.rm = TRUE), by = STATE]
station_average[, lon_50 := quantile(lon, probs = .5, na.rm = TRUE), by = STATE]

# Now, the euclidean distance ... $\sqrt{\sum_i(x_i - y_i)^2}$
station_average[, latlon_eudist := sqrt(
  (lat - lat_50)^2 + (lon - lon_50)^2
  )]

# get the minmium state center
mid_point_state <- station_average[ , .SD[which.min(latlon_eudist)], by = STATE]
knitr::kable(mid_point_state)
```



|STATE | USAFID|     temp|   wind.sp| atm.press|        lon|      lat|  temp_50| wind.sp_50| atm.press_50|     eudist|   lat_50|     lon_50| latlon_eudist|
|:-----|------:|--------:|---------:|---------:|----------:|--------:|--------:|----------:|------------:|----------:|--------:|----------:|-------------:|
|CA    | 723898| 28.40804| 2.8102319|  1011.520| -119.62800| 36.31899| 22.66268|   2.565446|     1012.557|  5.8433023| 36.02904| -119.84200|     0.3603698|
|TX    | 722575| 30.30429|       NaN|  1012.605|  -97.68315| 31.08315| 29.75188|   3.413737|     1012.460|        NaN| 31.10600|  -97.68315|     0.0228469|
|MI    | 725405| 19.91292| 1.9441737|       NaN|  -84.68800| 43.32200| 20.51970|   2.273423|     1014.927|        NaN| 43.06700|  -84.68800|     0.2550000|
|SC    | 720603| 26.32987| 1.5150567|       NaN|  -80.56700| 34.28300| 25.80545|   1.696119|     1015.281|        NaN| 34.18450|  -80.75853|     0.2153743|
|IL    | 724397| 22.03413| 3.4911565|  1015.253|  -88.94836| 40.48271| 22.43194|   2.237622|     1014.760|  1.4046440| 40.38787|  -88.86202|     0.1282519|
|MO    | 724453| 22.13591| 3.1443238|  1014.957|  -93.18299| 38.70400| 23.95109|   2.453547|     1014.522|  1.9902499| 38.59100|  -92.69100|     0.5048041|
|AR    | 720401| 25.83741| 0.5805530|       NaN|  -92.45000| 35.60000| 26.24296|   1.938625|     1014.591|        NaN| 35.33300|  -92.59000|     0.3014752|
|OR    | 725970| 23.45945| 1.9420802|  1013.924| -122.87087| 42.37773| 17.98061|   2.011436|     1015.269|  5.6420036| 42.27219| -123.11743|     0.2682059|
|WA    | 720388| 19.35326| 0.5583820|       NaN| -122.28683| 47.10383| 19.24684|   1.268571|           NA|        NaN| 47.10383| -122.41621|     0.1293725|
|GA    | 722175| 26.53220| 1.8966816|  1015.247|  -83.59972| 32.63325| 26.70404|   1.495596|     1015.208|  0.4380635| 32.62112|  -83.31141|     0.2885594|
|MN    | 726569| 19.56518| 2.1263578|       NaN|  -94.38213| 44.85913| 19.63017|   2.617071|     1015.042|        NaN| 44.87106|  -94.23901|     0.1436173|
|AL    | 722265| 27.10960| 1.8515312|  1014.800|  -86.35061| 32.38300| 26.33664|   1.662132|     1014.959|  0.8115269| 32.76548|  -86.58400|     0.4480676|
|IN    | 720961| 20.94252| 2.0536596|       NaN|  -86.37500| 40.71100| 22.25059|   2.344333|     1015.063|        NaN| 40.58895|  -86.28743|     0.1502182|
|NC    | 722201| 24.18637| 0.9103853|       NaN|  -79.10100| 35.58208| 24.72953|   1.627306|     1015.420|        NaN| 35.45755|  -78.99656|     0.1625292|
|VA    | 720498| 24.92926| 2.0898389|  1016.447|  -77.51700| 37.40000| 24.37799|   1.653032|     1015.158|  1.4684786| 37.32227|  -77.48300|     0.0848370|
|IA    | 725466| 21.79148| 2.7225459|       NaN|  -93.56600| 41.69100| 21.33461|   2.680875|     1014.964|        NaN| 41.83043|  -93.46252|     0.1736330|
|PA    | 725118| 24.58627| 2.0405376|  1015.104|  -76.85100| 40.21700| 21.69177|   1.784167|     1015.435|  2.9246829| 40.43500|  -76.92148|     0.2291098|
|NE    | 725520| 21.85539| 3.6775510|  1015.050|  -98.31427| 40.96155| 21.87354|   3.192539|     1014.332|  0.8663505| 41.18895|  -98.31427|     0.2273952|
|ID    | 725865| 17.08795| 3.7480456|       NaN| -114.29974| 43.50026| 20.56798|   2.568944|     1012.855|        NaN| 43.58100| -114.48602|     0.2030290|
|WI    | 726452| 19.21728| 2.4117467|  1015.180|  -89.83701| 44.35900| 18.85524|   2.053283|     1014.893|  0.5844788| 44.35900|  -89.71108|     0.1259283|
|WV    | 720328| 21.94820| 1.6178231|       NaN|  -80.27400| 39.00000| 21.94446|   1.633487|     1015.762|        NaN| 38.94249|  -80.52348|     0.2560234|
|MD    | 724067| 24.18765| 1.6004804|       NaN|  -76.41667| 39.33223| 24.89883|   1.883499|     1014.824|        NaN| 39.16703|  -76.42900|     0.1656565|
|AZ    | 722783| 34.84379| 2.6428760|  1008.210| -111.73290| 33.46688| 30.32372|   3.074359|     1010.144|  4.9353757| 33.53830| -111.69972|     0.0787485|
|OK    | 723540| 26.70174| 4.1168113|  1013.747|  -97.38319| 35.41690| 27.14427|   3.852697|     1012.567|  1.2870288| 35.48300|  -97.38319|     0.0660961|
|WY    | 726720| 21.70287| 3.8003344|  1012.771| -108.45695| 43.06440| 19.80699|   3.873392|     1013.157|  1.9360426| 42.80550| -108.23161|     0.3432262|
|LA    | 720468| 26.89874| 1.1130356|       NaN|  -92.09900| 30.55800| 27.87430|   1.592840|     1014.593|        NaN| 30.43202|  -91.93350|     0.2079911|
|KY    | 720448| 23.52994| 1.6049055|       NaN|  -84.77000| 37.57800| 23.88844|   1.895486|     1015.245|        NaN| 37.69132|  -84.81300|     0.1211996|
|FL    | 722011| 27.56952| 2.6740741|  1016.063|  -81.43700| 28.29000| 27.57325|   2.705069|     1015.335|  0.7287390| 28.36194|  -81.77900|     0.3494838|
|CO    | 726396| 12.93812| 2.5374887|       NaN| -105.51004| 39.05000| 21.49638|   3.098777|     1013.334|        NaN| 39.22305| -105.63798|     0.2152104|
|OH    | 720928| 21.87803| 2.0763441|       NaN|  -83.11500| 40.28000| 22.02062|   2.554397|     1015.351|        NaN| 40.35044|  -82.99833|     0.1362871|
|NJ    | 724090| 23.47238| 2.1486061|  1015.095|  -74.35016| 40.03300| 23.47238|   2.148606|     1014.825|  0.2697149| 40.27700|  -74.41689|     0.2529604|
|NM    | 722677| 21.33037| 4.9883663|  1015.128| -105.66201| 35.00300| 24.94447|   3.776083|     1012.525|  4.6161143| 35.00300| -105.66627|     0.0042620|
|KS    | 724509| 23.67510| 4.0668332|  1013.863|  -97.27500| 38.06763| 24.21220|   3.680613|     1013.389|  0.8136450| 38.33939|  -97.35268|     0.2826462|
|ND    | 720867| 18.20367| 4.2037684|       NaN| -100.02400| 48.39000| 18.52849|   3.956459|           NA|        NaN| 48.04850|  -99.82250|     0.3965154|
|VT    | 726114| 17.46999| 1.1657614|  1014.792|  -72.61400| 44.53400| 18.61379|   1.408247|     1014.792|  1.1692211| 44.44390|  -72.58800|     0.0937806|
|MS    | 722350| 27.03108| 2.0681041|  1014.575|  -90.07897| 32.32020| 26.69258|   1.636392|     1014.836|  0.6076094| 32.44350|  -89.66850|     0.4285930|
|CT    | 725027| 21.87299| 1.6481552|  1014.760|  -72.82800| 41.51000| 22.36880|   2.101801|     1014.810|  0.6739065| 41.43343|  -72.75506|     0.1057508|
|NV    | 724770| 21.37610| 3.3832432|  1012.475| -116.00502| 39.60100| 24.56293|   3.035050|     1012.204|  3.2172261| 39.35635| -116.45795|     0.5147797|
|UT    | 725724| 24.39332| 2.7791506|  1012.675| -111.72300| 40.21900| 24.35182|   3.145427|     1011.972|  0.7934400| 39.91401| -111.98868|     0.4044813|
|SD    | 726560| 20.55785| 4.5733734|  1013.479| -100.28500| 44.38101| 20.35662|   3.665638|     1014.398|  1.3069242| 44.14975|  -99.84202|     0.4997119|
|TN    | 723273| 25.01262| 1.7828603|  1004.715|  -86.52000| 36.00900| 24.88657|   1.576035|     1015.144| 10.4316511| 35.88439|  -86.38300|     0.1851926|
|NY    | 725145| 19.13882| 2.0224490|  1016.326|  -74.79500| 41.70103| 20.40674|   2.304075|     1014.887|  1.9381201| 42.36222|  -75.12899|     0.7407521|
|RI    | 725074| 24.45822| 4.7099462|       NaN|  -71.41200| 41.59700| 22.53551|   2.583469|     1014.728|        NaN| 41.59700|  -71.43299|     0.0209882|
|MA    | 725068| 20.83820| 1.3823436|  1014.377|  -71.02100| 41.87600| 21.30662|   2.710943|     1014.751|  1.4576715| 42.08504|  -70.93800|     0.2249155|
|DE    | 724088| 24.72840| 3.0324747|  1014.860|  -75.46697| 39.13291| 24.56026|   2.752930|     1015.046|  0.3755446| 39.13291|  -75.46697|     0.0000000|
|NH    | 726155| 19.96899| 1.9610188|  1014.689|  -71.43251| 43.56721| 19.55054|   1.563826|     1014.689|  0.5769452| 43.42260|  -71.46748|     0.1487728|
|ME    | 726073| 18.82098| 1.4142598|  1015.944|  -69.66723| 44.53300| 18.79016|   2.237210|     1014.399|  1.7506746| 44.38306|  -69.63112|     0.1542284|
|MT    | 726770| 22.99419| 4.1517371|  1013.286| -108.54002| 45.80547| 19.15492|   4.151737|     1014.185|  3.9431802| 45.80547| -109.45702|     0.9169993|

```r
# combine two states datasets
station_state[, type := "Center of the temperature, wind speed, atmospheric pressure"]
mid_point_state[, type := "Center of the State"]
center_states <- rbind(station_state, mid_point_state, fill = TRUE)

# Draw the map
leaflet(center_states) %>%
  addProviderTiles("OpenStreetMap") %>%
  addCircles(
    lat = ~lat, lng = ~lon, 
    color=~ifelse(type=="Center of the State",'pink','lightgreen'), 
    opacity=1,fillOpacity=0.7, radius=100
    )
```

```{=html}
<div id="htmlwidget-c19b6923a507f685529d" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-c19b6923a507f685529d">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["OpenStreetMap",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addCircles","args":[[33.8126445959104,29.7089922705314,42.2669724409449,34.4979967776584,41.4632502351834,37.152,35.831000967118,42.1470528169014,31.536,45.5430087241003,33.212,41.4530010638298,36.047,38.137011684518,42.5535839285714,41.3338032388664,41.9856836734694,42.5420088495575,44.3590049261084,39.643,39.4717428571429,32.1669504405286,36.1619871031746,44.3390462962963,32.5159599542334,37.9003206568712,26.5850051546392,37.3069902080783,40.708,40.033,34.3835781584582,39.5509035769829,43.344,31.1829822852081,41.7360101010101,40.0679923664122,41.1173656050955,45.443765323993,35.5930237288136,42.642987628866,41.5329991281604,41.9099972527472,39.6740047984645,43.2040901033973,44.45,45.6980046136102,36.3189948519949,31.0831530984204,43.322,34.283,40.4827108433735,38.7040028195489,35.6,42.3777333333333,47.1038340767172,32.6332458296752,44.859130600572,32.383,40.711,35.5820807424594,37.4,41.691,40.217,40.9615540935673,43.500262987013,44.3590049261084,39,39.3322308300395,33.4668795483061,35.4169039487727,43.0643970117396,30.558,37.578,28.29,39.05,40.28,40.033,35.0029964747356,38.0676310679612,48.39,44.5340018796992,32.3202025316456,41.5099990825688,39.6009961832061,40.219,44.3810077071291,36.009,41.7010332434861,41.597,41.876,39.1329054054054,43.5672086956522,44.533,45.8054722474977],[-118.146523855891,-98.0459922705314,-84.466968503937,-82.7099989258861,-90.5203170272813,-94.4950114942529,-90.646,-121.724052816901,-82.507,-94.0510196292257,-87.616,-87.0060010638298,-79.477,-78.454988315482,-92.4008803571429,-75.7249967611336,-97.4347891156463,-113.766053097345,-89.8370098522168,-79.916,-76.1699571428571,-110.883,-97.0889613095238,-105.540990740741,-92.04097597254,-85.9672290406223,-81.8610051546392,-108.626004895961,-84.027,-74.3501562130177,-103.315653104925,-97.6509035769829,-72.5179990974729,-90.4710035429584,-72.6509797979798,-118.568984732824,-111.966365605096,-98.413442206655,-88.9169966101695,-77.055993814433,-71.2829991281604,-70.729,-75.6060009596929,-71.5024542097489,-68.3667746192893,-110.440038062284,-119.628,-97.6831530984204,-84.688,-80.567,-88.9483614457831,-93.1829934210526,-92.45,-122.870865740741,-122.286834076717,-83.5997190517998,-94.382130600572,-86.3506071794872,-86.375,-79.101,-77.517,-93.566,-76.851,-98.3142660818713,-114.299737012987,-89.8370098522168,-80.274,-76.4166703557312,-111.732899623588,-97.3831921024546,-108.456947705443,-92.099,-84.77,-81.437,-105.510043030031,-83.115,-74.3501562130177,-105.662009400705,-97.275,-100.024,-72.614,-90.0789738924051,-72.8280009174312,-116.005020356234,-111.723,-100.285004816956,-86.52,-74.795,-71.412,-71.021,-75.4669684684685,-71.4325130434783,-69.6672303618711,-108.540024567789],100,null,null,{"interactive":true,"className":"","stroke":true,"color":["lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink"],"weight":5,"opacity":1,"fill":true,"fillColor":["lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","lightgreen","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink"],"fillOpacity":0.7},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]}],"limits":{"lat":[26.5850051546392,48.39],"lng":[-122.870865740741,-68.3667746192893]}},"evals":[],"jsHooks":[]}</script>
```

## Question 4: Means of means
Using the quantile() function, generate a summary table that shows the number of states included, average temperature, wind-speed, and atmospheric pressure by the variable “average temperature level,” which you’ll need to create.

Start by computing the states’ average temperature. Use that measurement to classify them according to the following criteria:

    * low: temp < 20
    * Mid: temp >= 20 and temp < 25
    * High: temp >= 25
Once you are done with that, you can compute the following:

    * Number of entries (records),
    * Number of NA entries,
    * Number of stations,
    * Number of states included, and
    * Mean temperature, wind-speed, and atmospheric pressure.
    
All by the levels described before.

```r
met[, state_temp := mean(temp, na.rm = TRUE), by = STATE]
met[, state_wind.sp := mean(wind.sp, na.rm = TRUE),by = STATE]
met[, state_atm.press := mean(atm.press, na.rm = TRUE),by = STATE]
met[, temp_cat := fifelse(
  state_temp < 20, "low-temp",
  fifelse(state_temp < 25, "mid-temp", "high-temp"))]
```

Lets make sure that we don't have NAs

```r
table(met$temp_cat, useNA = "always")
```

```
## 
## high-temp  low-temp  mid-temp      <NA> 
##    811126    430794   1135423         0
```

Now, let's summarize


```r
tab <- met[, .(
  N_entries = .N,
  N_stations = length(unique(USAFID)),
  avg_temp = mean(state_temp),
  avg_wind.sp = mean(state_wind.sp),
  avg_atm.press = mean(state_atm.press)
), by = temp_cat]

knitr::kable(tab)
```



|temp_cat  | N_entries| N_stations| avg_temp| avg_wind.sp| avg_atm.press|
|:---------|---------:|----------:|--------:|-----------:|-------------:|
|mid-temp  |   1135423|        781| 22.39870|    2.352673|      1014.587|
|high-temp |    811126|        555| 27.74436|    2.512854|      1013.679|
|low-temp  |    430794|        259| 18.96402|    2.642823|            NA|

