---
title: "Assignment1"
author: "Lili Xu"
date: "9/23/2021"
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
---


## Step 1: 
Given the formulated question from the assignment description, you will now conduct EDA Checklist items 2-4. First, download 2004 and 2019 data for all sites in California from the EPA Air Quality Data website.


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
library(plyr)
```

```
## ------------------------------------------------------------------------------
```

```
## You have loaded plyr after dplyr - this is likely to cause problems.
## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
## library(plyr); library(dplyr)
```

```
## ------------------------------------------------------------------------------
```

```
## 
## Attaching package: 'plyr'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     arrange, count, desc, failwith, id, mutate, rename, summarise,
##     summarize
```

```
## The following object is masked from 'package:purrr':
## 
##     compact
```

```r
library(dplyr)
library(ggforce)
```

### Read Data
Read in the data using data.table().


```r
PM2004 <- data.table::fread("~/Desktop/PM566/PM566-labs/Assignment1/2004PM2.5.csv")
PM2019 <- data.table::fread("~/Desktop/PM566/PM566-labs/Assignment1/2019PM2.5.csv")
```

### Check 2004 Data
Check the dimensions, headers, footers, variable names and variable types. Check for any data issues, particularly in the key variable we are analyzing.

```r
# 2004 
dim(PM2004) 
```

```
## [1] 19233    20
```

```r
head(PM2004)
```

```
##          Date Source  Site ID POC Daily Mean PM2.5 Concentration    UNITS
## 1: 01/01/2004    AQS 60010007   1                           11.0 ug/m3 LC
## 2: 01/02/2004    AQS 60010007   1                           12.2 ug/m3 LC
## 3: 01/03/2004    AQS 60010007   1                           16.5 ug/m3 LC
## 4: 01/04/2004    AQS 60010007   1                           19.5 ug/m3 LC
## 5: 01/05/2004    AQS 60010007   1                           11.5 ug/m3 LC
## 6: 01/06/2004    AQS 60010007   1                           32.5 ug/m3 LC
##    DAILY_AQI_VALUE Site Name DAILY_OBS_COUNT PERCENT_COMPLETE
## 1:              46 Livermore               1              100
## 2:              51 Livermore               1              100
## 3:              60 Livermore               1              100
## 4:              67 Livermore               1              100
## 5:              48 Livermore               1              100
## 6:              94 Livermore               1              100
##    AQS_PARAMETER_CODE                     AQS_PARAMETER_DESC CBSA_CODE
## 1:              88502 Acceptable PM2.5 AQI & Speciation Mass     41860
## 2:              88502 Acceptable PM2.5 AQI & Speciation Mass     41860
## 3:              88502 Acceptable PM2.5 AQI & Speciation Mass     41860
## 4:              88502 Acceptable PM2.5 AQI & Speciation Mass     41860
## 5:              88502 Acceptable PM2.5 AQI & Speciation Mass     41860
## 6:              88502 Acceptable PM2.5 AQI & Speciation Mass     41860
##                            CBSA_NAME STATE_CODE      STATE COUNTY_CODE  COUNTY
## 1: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 2: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 3: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 4: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 5: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 6: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
##    SITE_LATITUDE SITE_LONGITUDE
## 1:      37.68753      -121.7842
## 2:      37.68753      -121.7842
## 3:      37.68753      -121.7842
## 4:      37.68753      -121.7842
## 5:      37.68753      -121.7842
## 6:      37.68753      -121.7842
```

```r
tail(PM2004)
```

```
##          Date Source  Site ID POC Daily Mean PM2.5 Concentration    UNITS
## 1: 12/14/2004    AQS 61131003   1                             11 ug/m3 LC
## 2: 12/17/2004    AQS 61131003   1                             16 ug/m3 LC
## 3: 12/20/2004    AQS 61131003   1                             17 ug/m3 LC
## 4: 12/23/2004    AQS 61131003   1                              9 ug/m3 LC
## 5: 12/26/2004    AQS 61131003   1                             24 ug/m3 LC
## 6: 12/29/2004    AQS 61131003   1                              9 ug/m3 LC
##    DAILY_AQI_VALUE            Site Name DAILY_OBS_COUNT PERCENT_COMPLETE
## 1:              46 Woodland-Gibson Road               1              100
## 2:              59 Woodland-Gibson Road               1              100
## 3:              61 Woodland-Gibson Road               1              100
## 4:              38 Woodland-Gibson Road               1              100
## 5:              76 Woodland-Gibson Road               1              100
## 6:              38 Woodland-Gibson Road               1              100
##    AQS_PARAMETER_CODE       AQS_PARAMETER_DESC CBSA_CODE
## 1:              88101 PM2.5 - Local Conditions     40900
## 2:              88101 PM2.5 - Local Conditions     40900
## 3:              88101 PM2.5 - Local Conditions     40900
## 4:              88101 PM2.5 - Local Conditions     40900
## 5:              88101 PM2.5 - Local Conditions     40900
## 6:              88101 PM2.5 - Local Conditions     40900
##                                  CBSA_NAME STATE_CODE      STATE COUNTY_CODE
## 1: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 2: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 3: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 4: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 5: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 6: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
##    COUNTY SITE_LATITUDE SITE_LONGITUDE
## 1:   Yolo      38.66121      -121.7327
## 2:   Yolo      38.66121      -121.7327
## 3:   Yolo      38.66121      -121.7327
## 4:   Yolo      38.66121      -121.7327
## 5:   Yolo      38.66121      -121.7327
## 6:   Yolo      38.66121      -121.7327
```

```r
names(PM2004)
```

```
##  [1] "Date"                           "Source"                        
##  [3] "Site ID"                        "POC"                           
##  [5] "Daily Mean PM2.5 Concentration" "UNITS"                         
##  [7] "DAILY_AQI_VALUE"                "Site Name"                     
##  [9] "DAILY_OBS_COUNT"                "PERCENT_COMPLETE"              
## [11] "AQS_PARAMETER_CODE"             "AQS_PARAMETER_DESC"            
## [13] "CBSA_CODE"                      "CBSA_NAME"                     
## [15] "STATE_CODE"                     "STATE"                         
## [17] "COUNTY_CODE"                    "COUNTY"                        
## [19] "SITE_LATITUDE"                  "SITE_LONGITUDE"
```

```r
str(PM2004)
```

```
## Classes 'data.table' and 'data.frame':	19233 obs. of  20 variables:
##  $ Date                          : chr  "01/01/2004" "01/02/2004" "01/03/2004" "01/04/2004" ...
##  $ Source                        : chr  "AQS" "AQS" "AQS" "AQS" ...
##  $ Site ID                       : int  60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 ...
##  $ POC                           : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Daily Mean PM2.5 Concentration: num  11 12.2 16.5 19.5 11.5 32.5 14 29.9 21 15.7 ...
##  $ UNITS                         : chr  "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" ...
##  $ DAILY_AQI_VALUE               : int  46 51 60 67 48 94 55 88 70 59 ...
##  $ Site Name                     : chr  "Livermore" "Livermore" "Livermore" "Livermore" ...
##  $ DAILY_OBS_COUNT               : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ PERCENT_COMPLETE              : num  100 100 100 100 100 100 100 100 100 100 ...
##  $ AQS_PARAMETER_CODE            : int  88502 88502 88502 88502 88502 88502 88101 88502 88502 88101 ...
##  $ AQS_PARAMETER_DESC            : chr  "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" ...
##  $ CBSA_CODE                     : int  41860 41860 41860 41860 41860 41860 41860 41860 41860 41860 ...
##  $ CBSA_NAME                     : chr  "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" ...
##  $ STATE_CODE                    : int  6 6 6 6 6 6 6 6 6 6 ...
##  $ STATE                         : chr  "California" "California" "California" "California" ...
##  $ COUNTY_CODE                   : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY                        : chr  "Alameda" "Alameda" "Alameda" "Alameda" ...
##  $ SITE_LATITUDE                 : num  37.7 37.7 37.7 37.7 37.7 ...
##  $ SITE_LONGITUDE                : num  -122 -122 -122 -122 -122 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
summary(PM2004$`Daily Mean PM2.5 Concentration`)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   -0.10    6.00   10.10   13.13   16.30  251.00
```

```r
summary(PM2004$DAILY_AQI_VALUE)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00   25.00   42.00   46.34   60.00  301.00
```
1. There are 19233 rows and 20 columns in the PM2.5_2004 dataset.
2. The maximum of Daily Mean PM2.5 Concentration is 251.00. The mean is 13.13, and the median is 10.10. So, in 2004, PM 2.5 is not too seriously, with the latest PM 2.5 range (0 ~ 12: good).
3. The U.S. AQI is EPA’s index for reporting air quality. With the AQI value table on the [Air Quality Index (AQI) Basics](https://www.airnow.gov/aqi/aqi-basics/):
    * the maximum of AQI is 301 ---- Hazardous (Health warning of emergency conditions: everyone is more likely to be affected.)
    * the mean of AQI is 46.34 ---- Good(Air quality is satisfactory, and air pollution poses little or no risk.)
    * the Median of AQI is 42.00 ---- Good(Air quality is satisfactory, and air pollution poses little or no risk.)
    * which seems the AQI level of 2004 is not too bad, most time is good.

### Check 2019 Data
Check the dimensions, headers, footers, variable names and variable types. Check for any data issues, particularly in the key variable we are analyzing.

```r
# 2019
dim(PM2019)
```

```
## [1] 53086    20
```

```r
head(PM2019)
```

```
##          Date Source  Site ID POC Daily Mean PM2.5 Concentration    UNITS
## 1: 01/01/2019    AQS 60010007   3                            5.7 ug/m3 LC
## 2: 01/02/2019    AQS 60010007   3                           11.9 ug/m3 LC
## 3: 01/03/2019    AQS 60010007   3                           20.1 ug/m3 LC
## 4: 01/04/2019    AQS 60010007   3                           28.8 ug/m3 LC
## 5: 01/05/2019    AQS 60010007   3                           11.2 ug/m3 LC
## 6: 01/06/2019    AQS 60010007   3                            2.7 ug/m3 LC
##    DAILY_AQI_VALUE Site Name DAILY_OBS_COUNT PERCENT_COMPLETE
## 1:              24 Livermore               1              100
## 2:              50 Livermore               1              100
## 3:              68 Livermore               1              100
## 4:              86 Livermore               1              100
## 5:              47 Livermore               1              100
## 6:              11 Livermore               1              100
##    AQS_PARAMETER_CODE       AQS_PARAMETER_DESC CBSA_CODE
## 1:              88101 PM2.5 - Local Conditions     41860
## 2:              88101 PM2.5 - Local Conditions     41860
## 3:              88101 PM2.5 - Local Conditions     41860
## 4:              88101 PM2.5 - Local Conditions     41860
## 5:              88101 PM2.5 - Local Conditions     41860
## 6:              88101 PM2.5 - Local Conditions     41860
##                            CBSA_NAME STATE_CODE      STATE COUNTY_CODE  COUNTY
## 1: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 2: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 3: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 4: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 5: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
## 6: San Francisco-Oakland-Hayward, CA          6 California           1 Alameda
##    SITE_LATITUDE SITE_LONGITUDE
## 1:      37.68753      -121.7842
## 2:      37.68753      -121.7842
## 3:      37.68753      -121.7842
## 4:      37.68753      -121.7842
## 5:      37.68753      -121.7842
## 6:      37.68753      -121.7842
```

```r
tail(PM2019)
```

```
##          Date Source  Site ID POC Daily Mean PM2.5 Concentration    UNITS
## 1: 11/11/2019    AQS 61131003   1                           13.5 ug/m3 LC
## 2: 11/17/2019    AQS 61131003   1                           18.1 ug/m3 LC
## 3: 11/29/2019    AQS 61131003   1                           12.5 ug/m3 LC
## 4: 12/17/2019    AQS 61131003   1                           23.8 ug/m3 LC
## 5: 12/23/2019    AQS 61131003   1                            1.0 ug/m3 LC
## 6: 12/29/2019    AQS 61131003   1                            9.1 ug/m3 LC
##    DAILY_AQI_VALUE            Site Name DAILY_OBS_COUNT PERCENT_COMPLETE
## 1:              54 Woodland-Gibson Road               1              100
## 2:              64 Woodland-Gibson Road               1              100
## 3:              52 Woodland-Gibson Road               1              100
## 4:              76 Woodland-Gibson Road               1              100
## 5:               4 Woodland-Gibson Road               1              100
## 6:              38 Woodland-Gibson Road               1              100
##    AQS_PARAMETER_CODE       AQS_PARAMETER_DESC CBSA_CODE
## 1:              88101 PM2.5 - Local Conditions     40900
## 2:              88101 PM2.5 - Local Conditions     40900
## 3:              88101 PM2.5 - Local Conditions     40900
## 4:              88101 PM2.5 - Local Conditions     40900
## 5:              88101 PM2.5 - Local Conditions     40900
## 6:              88101 PM2.5 - Local Conditions     40900
##                                  CBSA_NAME STATE_CODE      STATE COUNTY_CODE
## 1: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 2: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 3: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 4: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 5: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
## 6: Sacramento--Roseville--Arden-Arcade, CA          6 California         113
##    COUNTY SITE_LATITUDE SITE_LONGITUDE
## 1:   Yolo      38.66121      -121.7327
## 2:   Yolo      38.66121      -121.7327
## 3:   Yolo      38.66121      -121.7327
## 4:   Yolo      38.66121      -121.7327
## 5:   Yolo      38.66121      -121.7327
## 6:   Yolo      38.66121      -121.7327
```

```r
names(PM2019)
```

```
##  [1] "Date"                           "Source"                        
##  [3] "Site ID"                        "POC"                           
##  [5] "Daily Mean PM2.5 Concentration" "UNITS"                         
##  [7] "DAILY_AQI_VALUE"                "Site Name"                     
##  [9] "DAILY_OBS_COUNT"                "PERCENT_COMPLETE"              
## [11] "AQS_PARAMETER_CODE"             "AQS_PARAMETER_DESC"            
## [13] "CBSA_CODE"                      "CBSA_NAME"                     
## [15] "STATE_CODE"                     "STATE"                         
## [17] "COUNTY_CODE"                    "COUNTY"                        
## [19] "SITE_LATITUDE"                  "SITE_LONGITUDE"
```

```r
str(PM2019)
```

```
## Classes 'data.table' and 'data.frame':	53086 obs. of  20 variables:
##  $ Date                          : chr  "01/01/2019" "01/02/2019" "01/03/2019" "01/04/2019" ...
##  $ Source                        : chr  "AQS" "AQS" "AQS" "AQS" ...
##  $ Site ID                       : int  60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 ...
##  $ POC                           : int  3 3 3 3 3 3 3 3 3 3 ...
##  $ Daily Mean PM2.5 Concentration: num  5.7 11.9 20.1 28.8 11.2 2.7 2.8 7 3.1 7.1 ...
##  $ UNITS                         : chr  "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" ...
##  $ DAILY_AQI_VALUE               : int  24 50 68 86 47 11 12 29 13 30 ...
##  $ Site Name                     : chr  "Livermore" "Livermore" "Livermore" "Livermore" ...
##  $ DAILY_OBS_COUNT               : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ PERCENT_COMPLETE              : num  100 100 100 100 100 100 100 100 100 100 ...
##  $ AQS_PARAMETER_CODE            : int  88101 88101 88101 88101 88101 88101 88101 88101 88101 88101 ...
##  $ AQS_PARAMETER_DESC            : chr  "PM2.5 - Local Conditions" "PM2.5 - Local Conditions" "PM2.5 - Local Conditions" "PM2.5 - Local Conditions" ...
##  $ CBSA_CODE                     : int  41860 41860 41860 41860 41860 41860 41860 41860 41860 41860 ...
##  $ CBSA_NAME                     : chr  "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" ...
##  $ STATE_CODE                    : int  6 6 6 6 6 6 6 6 6 6 ...
##  $ STATE                         : chr  "California" "California" "California" "California" ...
##  $ COUNTY_CODE                   : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY                        : chr  "Alameda" "Alameda" "Alameda" "Alameda" ...
##  $ SITE_LATITUDE                 : num  37.7 37.7 37.7 37.7 37.7 ...
##  $ SITE_LONGITUDE                : num  -122 -122 -122 -122 -122 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
summary(PM2019$`Daily Mean PM2.5 Concentration`)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  -2.200   4.000   6.500   7.734   9.900 120.900
```

```r
summary(PM2019$DAILY_AQI_VALUE)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00   17.00   27.00   30.56   41.00  185.00
```

1. There are 53086 rows and 20 columns in the PM2.5_2019 dataset.
2. The maximum of Daily Mean PM2.5 Concentration is 120.900 The mean is 7.78, and the median is 6.500 So, in 2019, PM 2.5 is much better than 2004, with the latest PM 2.5 range (0 ~ 12: good).
3. The U.S. AQI is EPA’s index for reporting air quality. With the AQI value table on the [Air Quality Index (AQI) Basics](https://www.airnow.gov/aqi/aqi-basics/):
    * the maximum of AQI is 185.00 ---- Unhealthy (Some members of the general public may experience health effects; members of sensitive groups may experience more serious health effects.)
    * the mean of AQI is 30.72 ---- Good(Air quality is satisfactory, and air pollution poses little or no risk.)
    * the Median of AQI is 27.00 ---- Good(Air quality is satisfactory, and air pollution poses little or no risk.)
    * which seems the AQI level of 2019 is more healthy than 2004, the level of maxium is much lower.


## Step 2: 
Combine the two years of data into one data frame. Use the Date variable to create a new column for year, which will serve as an identifier. Change the names of the key variables so that they are easier to refer to in your code.


```r
PM2004[, Year := "2004"]
str(PM2004)
```

```
## Classes 'data.table' and 'data.frame':	19233 obs. of  21 variables:
##  $ Date                          : chr  "01/01/2004" "01/02/2004" "01/03/2004" "01/04/2004" ...
##  $ Source                        : chr  "AQS" "AQS" "AQS" "AQS" ...
##  $ Site ID                       : int  60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 ...
##  $ POC                           : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Daily Mean PM2.5 Concentration: num  11 12.2 16.5 19.5 11.5 32.5 14 29.9 21 15.7 ...
##  $ UNITS                         : chr  "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" ...
##  $ DAILY_AQI_VALUE               : int  46 51 60 67 48 94 55 88 70 59 ...
##  $ Site Name                     : chr  "Livermore" "Livermore" "Livermore" "Livermore" ...
##  $ DAILY_OBS_COUNT               : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ PERCENT_COMPLETE              : num  100 100 100 100 100 100 100 100 100 100 ...
##  $ AQS_PARAMETER_CODE            : int  88502 88502 88502 88502 88502 88502 88101 88502 88502 88101 ...
##  $ AQS_PARAMETER_DESC            : chr  "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" ...
##  $ CBSA_CODE                     : int  41860 41860 41860 41860 41860 41860 41860 41860 41860 41860 ...
##  $ CBSA_NAME                     : chr  "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" ...
##  $ STATE_CODE                    : int  6 6 6 6 6 6 6 6 6 6 ...
##  $ STATE                         : chr  "California" "California" "California" "California" ...
##  $ COUNTY_CODE                   : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY                        : chr  "Alameda" "Alameda" "Alameda" "Alameda" ...
##  $ SITE_LATITUDE                 : num  37.7 37.7 37.7 37.7 37.7 ...
##  $ SITE_LONGITUDE                : num  -122 -122 -122 -122 -122 ...
##  $ Year                          : chr  "2004" "2004" "2004" "2004" ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
PM2019[, Year := "2019"]
str(PM2019)
```

```
## Classes 'data.table' and 'data.frame':	53086 obs. of  21 variables:
##  $ Date                          : chr  "01/01/2019" "01/02/2019" "01/03/2019" "01/04/2019" ...
##  $ Source                        : chr  "AQS" "AQS" "AQS" "AQS" ...
##  $ Site ID                       : int  60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 ...
##  $ POC                           : int  3 3 3 3 3 3 3 3 3 3 ...
##  $ Daily Mean PM2.5 Concentration: num  5.7 11.9 20.1 28.8 11.2 2.7 2.8 7 3.1 7.1 ...
##  $ UNITS                         : chr  "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" ...
##  $ DAILY_AQI_VALUE               : int  24 50 68 86 47 11 12 29 13 30 ...
##  $ Site Name                     : chr  "Livermore" "Livermore" "Livermore" "Livermore" ...
##  $ DAILY_OBS_COUNT               : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ PERCENT_COMPLETE              : num  100 100 100 100 100 100 100 100 100 100 ...
##  $ AQS_PARAMETER_CODE            : int  88101 88101 88101 88101 88101 88101 88101 88101 88101 88101 ...
##  $ AQS_PARAMETER_DESC            : chr  "PM2.5 - Local Conditions" "PM2.5 - Local Conditions" "PM2.5 - Local Conditions" "PM2.5 - Local Conditions" ...
##  $ CBSA_CODE                     : int  41860 41860 41860 41860 41860 41860 41860 41860 41860 41860 ...
##  $ CBSA_NAME                     : chr  "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" ...
##  $ STATE_CODE                    : int  6 6 6 6 6 6 6 6 6 6 ...
##  $ STATE                         : chr  "California" "California" "California" "California" ...
##  $ COUNTY_CODE                   : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY                        : chr  "Alameda" "Alameda" "Alameda" "Alameda" ...
##  $ SITE_LATITUDE                 : num  37.7 37.7 37.7 37.7 37.7 ...
##  $ SITE_LONGITUDE                : num  -122 -122 -122 -122 -122 ...
##  $ Year                          : chr  "2019" "2019" "2019" "2019" ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
PM2.5_Combine <- rbind(PM2004,PM2019)
str(PM2.5_Combine)
```

```
## Classes 'data.table' and 'data.frame':	72319 obs. of  21 variables:
##  $ Date                          : chr  "01/01/2004" "01/02/2004" "01/03/2004" "01/04/2004" ...
##  $ Source                        : chr  "AQS" "AQS" "AQS" "AQS" ...
##  $ Site ID                       : int  60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 60010007 ...
##  $ POC                           : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ Daily Mean PM2.5 Concentration: num  11 12.2 16.5 19.5 11.5 32.5 14 29.9 21 15.7 ...
##  $ UNITS                         : chr  "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" "ug/m3 LC" ...
##  $ DAILY_AQI_VALUE               : int  46 51 60 67 48 94 55 88 70 59 ...
##  $ Site Name                     : chr  "Livermore" "Livermore" "Livermore" "Livermore" ...
##  $ DAILY_OBS_COUNT               : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ PERCENT_COMPLETE              : num  100 100 100 100 100 100 100 100 100 100 ...
##  $ AQS_PARAMETER_CODE            : int  88502 88502 88502 88502 88502 88502 88101 88502 88502 88101 ...
##  $ AQS_PARAMETER_DESC            : chr  "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" "Acceptable PM2.5 AQI & Speciation Mass" ...
##  $ CBSA_CODE                     : int  41860 41860 41860 41860 41860 41860 41860 41860 41860 41860 ...
##  $ CBSA_NAME                     : chr  "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" "San Francisco-Oakland-Hayward, CA" ...
##  $ STATE_CODE                    : int  6 6 6 6 6 6 6 6 6 6 ...
##  $ STATE                         : chr  "California" "California" "California" "California" ...
##  $ COUNTY_CODE                   : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ COUNTY                        : chr  "Alameda" "Alameda" "Alameda" "Alameda" ...
##  $ SITE_LATITUDE                 : num  37.7 37.7 37.7 37.7 37.7 ...
##  $ SITE_LONGITUDE                : num  -122 -122 -122 -122 -122 ...
##  $ Year                          : chr  "2004" "2004" "2004" "2004" ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

```r
table(PM2.5_Combine$Year)
```

```
## 
##  2004  2019 
## 19233 53086
```

```r
colnames(PM2.5_Combine)[5] <- "PM2.5"
colnames(PM2.5_Combine)[7] <- "AQI"
colnames(PM2.5_Combine)[19] <- "lat"
colnames(PM2.5_Combine)[20] <- "lon"
```

## Step 3: 
Create a basic map in leaflet() that shows the locations of the sites (make sure to use different colors for each year). Summarize the spatial distribution of the monitoring sites.


```r
year.pal <- colorFactor(c("pink","lightgreen"), domain=PM2.5_Combine$Year)
leaflet(PM2.5_Combine) %>%
  addProviderTiles('CartoDB.Positron') %>% 
  addCircleMarkers(lat=~lat,lng=~lon, opacity=1, fillOpacity=1, radius=1,color = ~year.pal(PM2.5_Combine$Year))
```

```{=html}
<div id="htmlwidget-afa314783a3f1422a547" style="width:672px;height:480px;" class="leaflet html-widget"></div>
```

## Step 4: 
Check for any missing or implausible values of PM in the combined dataset. Explore the proportions of each and provide a summary of any temporal patterns you see in these observations.


```r
summary(PM2.5_Combine$PM2.5)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  -2.200   4.400   7.200   9.169  11.300 251.000
```

```r
# Base online survey, PM 2.5 range: 0-12 which is good, so I need to remove all data under 0.
dim(PM2.5_Combine[PM2.5<=0])[1]/dim(PM2.5_Combine)[1]
```

```
## [1] 0.005033255
```

```r
PM2.5_Combine <- PM2.5_Combine[PM2.5>=0]
summary(PM2.5_Combine$PM2.5)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   0.000   4.400   7.300   9.208  11.300 251.000
```

```r
summary(PM2.5_Combine$AQI)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00   18.00   30.00   34.89   47.00  301.00
```

```r
dim(PM2.5_Combine[is.na(PM2.5)])
```

```
## [1]  0 21
```

```r
dim(PM2.5_Combine[is.na(AQI)])
```

```
## [1]  0 21
```

```r
summary(PM2.5_Combine$lat)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   32.58   34.11   36.49   36.31   37.97   41.76
```

```r
summary(PM2.5_Combine$lon)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  -124.2  -121.6  -119.8  -119.8  -118.1  -115.5
```

```r
dim(PM2.5_Combine[is.na(Date)])
```

```
## [1]  0 21
```

```r
dim(PM2.5_Combine[is.na(`Site Name`)])
```

```
## [1]  0 21
```
Based on the first summary, it is obviously that Pm2.5 combine data contains negative values, but base the [Revised PM2.5 AQI breakpoints](https://aqicn.org/faq/2013-09-09/revised-pm25-aqi-breakpoints/), the value must larger than 0. With the original data, the proportion of error data is 0.503%. There is no missing values in the key variables. As the summary of AQI, it seems more healthy, the data is around the median 30.00 which is lower than standard.

## Step 5: 
Explore the main question of interest at three different spatial levels. Create exploratory plots (e.g. boxplots, histograms, line plots) and summary statistics that best suit each level of data. Be sure to write up explanations of what you observe in these data.

### State

Reference: [facet_zoom](https://rdrr.io/cran/ggforce/man/facet_zoom.html)

```r
ggplot(PM2.5_Combine[!is.na(PM2.5)],  aes(x=PM2.5, fill=Year, color=Year)) +
  geom_histogram()+
  labs(title = "Histograms of the Concentration of PM2.5 in California State ", x = "concentration")+
  facet_zoom(x = PM2.5 <50)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](Assignment1_files/figure-html/state-1.png)<!-- -->

It is obviously from Historgram plot that the Concentration of PM2.5 in 2019 much lower than 2004 in California.

### County

Reference: [facet_wrap](https://ggplot2-book.org/facet.html)

```r
Mean_2004<-ddply(PM2004,.(COUNTY,Year),summarize,mean_AQI=mean(DAILY_AQI_VALUE))
str(Mean_2004)
```

```
## 'data.frame':	47 obs. of  3 variables:
##  $ COUNTY  : chr  "Alameda" "Butte" "Calaveras" "Colusa" ...
##  $ Year    : chr  "2004" "2004" "2004" "2004" ...
##  $ mean_AQI: num  41.2 37.4 30.6 36.8 44.7 ...
```

```r
Mean_2019<-ddply(PM2019,.(COUNTY,Year),summarize,mean_AQI=mean(DAILY_AQI_VALUE))
str(Mean_2019)
```

```
## 'data.frame':	51 obs. of  3 variables:
##  $ COUNTY  : chr  "Alameda" "Butte" "Calaveras" "Colusa" ...
##  $ Year    : chr  "2019" "2019" "2019" "2019" ...
##  $ mean_AQI: num  29.8 27.6 22.6 26.8 29.3 ...
```

```r
Pm2.5_Mean_AQI <- rbind(Mean_2004,Mean_2019)
# ???
#Pm2.5_Mean_AQI[!is.na(mean_AQI)]

ggplot(Pm2.5_Mean_AQI, aes(x=COUNTY, y=mean_AQI, group=Year)) +
  geom_line(aes(color=Year))+
  geom_point(aes(color=Year))+
  labs(title = "Mean AQI Value in Different County in the California by Year ", x  = "County Name", y = " Mean AQI Value")+
  theme(text = element_text(size=10),
        axis.text.x = element_text(angle=90, hjust=1))+
  facet_wrap(~Year, scales = "free_y", ncol = 1)
```

![](Assignment1_files/figure-html/county-1.png)<!-- -->

For the relationship between AQI and county in California, we need to get the mean AQI value first which allow us to draw the line plot. The Above line plots show the AQI level in 2019 counties is better than the level in 2004, since the maximum of 2004 is around 60, the maximum of 2019 is around 50. El Dorado, Lake are the lowest AQI level in 2019 counties, Contra Costa, El Dorado, Siskiyou, Trinity are the lowest counties(AQI) in 2004

### Site in Los Angeles


```r
LA_AQI <- filter(PM2.5_Combine, COUNTY == "Los Angeles")
ggplot( 
  LA_AQI,
  mapping = aes(x = `Site Name`, y = AQI, fill = Year)) +
  geom_boxplot() +
  labs(title = 'Boxplot of AQI in LA site by Year', x = 'Site Name')+
  theme(text = element_text(size=10),
        axis.text.x = element_text(angle=90, hjust=1))
```

![](Assignment1_files/figure-html/site in los angeles-1.png)<!-- -->

Excluding the missing data, it is clearly show the information that the AQI level in 2019 site Los Angeles is better than 2004, the red boxes and their means are higher than blue boxes and their means.