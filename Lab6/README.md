---
title: "Lab6"
author: "Lili Xu"
date: "10/1/2021"
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



## Download the Dataset


```r
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──
```

```
## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
## ✓ tibble  3.1.0     ✓ dplyr   1.0.7
## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
## ✓ readr   1.4.0     ✓ forcats 0.5.1
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(tibble)
library(dplyr)
library(ggplot2)
library(tidytext)
```



```r
if (!file.exists("mtsamples.csv"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv",
    destfile = "mtsamples.csv",
    method = "libcurl",
    timeout = 60
  )
mtsamples <- read.csv("mtsamples.csv")
mtsamples <- as_tibble(mtsamples)
mtsamples
```

```
## # A tibble: 4,999 x 6
##        X description    medical_special… sample_name  transcription   keywords  
##    <int> <chr>          <chr>            <chr>        <chr>           <chr>     
##  1     0 " A 23-year-o… " Allergy / Imm… " Allergic … "SUBJECTIVE:, … "allergy …
##  2     1 " Consult for… " Bariatrics"    " Laparosco… "PAST MEDICAL … "bariatri…
##  3     2 " Consult for… " Bariatrics"    " Laparosco… "HISTORY OF PR… "bariatri…
##  4     3 " 2-D M-Mode.… " Cardiovascula… " 2-D Echoc… "2-D M-MODE: ,… "cardiova…
##  5     4 " 2-D Echocar… " Cardiovascula… " 2-D Echoc… "1.  The left … "cardiova…
##  6     5 " Morbid obes… " Bariatrics"    " Laparosco… "PREOPERATIVE … "bariatri…
##  7     6 " Liposuction… " Bariatrics"    " Liposucti… "PREOPERATIVE … "bariatri…
##  8     7 " 2-D Echocar… " Cardiovascula… " 2-D Echoc… "2-D ECHOCARDI… "cardiova…
##  9     8 " Suction-ass… " Bariatrics"    " Lipectomy… "PREOPERATIVE … "bariatri…
## 10     9 " Echocardiog… " Cardiovascula… " 2-D Echoc… "DESCRIPTION:,… "cardiova…
## # … with 4,989 more rows
```




## Question 1: What specialties do we have?

We can use count() from dplyr to figure out how many different catagories do we have? Are these catagories related? overlapping? evenly distributed?


```r
specialties <- mtsamples %>%
  count(medical_specialty)

specialties %>%
  arrange(desc(n))%>%
  top_n(15) %>%
  knitr::kable()
```

```
## Selecting by n
```



|medical_specialty             |    n|
|:-----------------------------|----:|
|Surgery                       | 1103|
|Consult - History and Phy.    |  516|
|Cardiovascular / Pulmonary    |  372|
|Orthopedic                    |  355|
|Radiology                     |  273|
|General Medicine              |  259|
|Gastroenterology              |  230|
|Neurology                     |  223|
|SOAP / Chart / Progress Notes |  166|
|Obstetrics / Gynecology       |  160|
|Urology                       |  158|
|Discharge Summary             |  108|
|ENT - Otolaryngology          |   98|
|Neurosurgery                  |   94|
|Hematology - Oncology         |   90|

There are 40 specialties. Let's take a look at the distribution.

```r
ggplot(mtsamples, aes(x = medical_specialty)) +
  geom_histogram(stat = "count") +
  coord_flip()
```

```
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

![](README_files/figure-html/dist-1.png)<!-- -->

```r
# method 2
ggplot(specialties, aes(x = n, y = fct_reorder(medical_specialty, n))) +
  geom_col()
```

![](README_files/figure-html/dist2-1.png)<!-- -->

There are not evenly (uniformly) distribution. 
There is no overlapping happen.

## Question 2

Explain what we see from this result. Does it makes sense? What insights (if any) do we get?


```r
mtsamples %>% 
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-transcript-1.png)<!-- -->

The word "patient" seems to be important, but we observe a lot of stop words.

## Question 3
Redo visualization but remove stopwords before
What do we see know that we have removed stop words? Does it give us a better idea of what the text is about?

```r
mtsamples %>% 
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  anti_join(stop_words, by = c("word")) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-transcript-wo-stop-1.png)<!-- -->

Looking better ~~, but we don't like the numbers~~.

Bonus points if you remove numbers as well

```r
# method 1
tokens_clean <- mtsamples %>% 
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  anti_join(stop_words, by = c("word"))

nums <- tokens_clean %>% filter(str_detect(word, "^[0-9]")) %>% select(word) %>% unique()

tokens_clean %>% 
  anti_join(nums, by = c("word")) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-transcript-num-1.png)<!-- -->

```r
# method 2
mtsamples %>% 
  unnest_tokens(output = word, input = transcription) %>%
  count(word, sort = TRUE) %>%
  anti_join(stop_words, by = c("word")) %>%
  # using regular expressions to remove numbers
  filter(!grepl(pattern = "^[0-9]+$", x = word)) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(word,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/token-transcript-num-2.png)<!-- -->
Without the spotwords and number, we can see more relative words in the dataset.


## Question 4

repeat question 2, but this time tokenize into bi-grams. how does the result change if you look at tri-grams?


```r
mtsamples %>% 
  unnest_ngrams(output = bigram, input = transcription, n = 2) %>%
  count(bigram, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(bigram,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/bi-grams-transcript-1.png)<!-- -->

Using bi-grams is not very informative, Let's try trigrams:


```r
mtsamples %>% 
  unnest_ngrams(output = trigram, input = transcription, n = 3) %>%
  count(trigram, sort = TRUE) %>%
  top_n(20) %>%
  ggplot(aes(x=n, y = fct_reorder(trigram,n))) +
    geom_col()
```

```
## Selecting by n
```

![](README_files/figure-html/tri-grams-transcript-1.png)<!-- -->

Now some phrase start to show up, e.g., "tolerated the procedure", "prepped and draped".

## Question 5

Using the results you got from questions 4. Pick a word and count the words that appears after and before it.

Pick the word: Patient

```r
bigrams <- mtsamples %>%
  unnest_ngrams(output = bigram, input = transcription, n = 2) %>%
  separate(bigram, into = c("w1", "w2"), sep = " ") %>%
  filter((w1 == "patient") | (w2 == "patient"))
bigrams %>%
  filter(w1 == "patient") %>%
  select(w1, w2) %>%
  count(w2, sort = TRUE)
```

```
## # A tibble: 588 x 2
##    w2            n
##    <chr>     <int>
##  1 was        6293
##  2 is         3332
##  3 has        1417
##  4 tolerated   994
##  5 had         888
##  6 will        616
##  7 denies      552
##  8 and         377
##  9 states      363
## 10 does        334
## # … with 578 more rows
```

```r
bigrams %>%
  filter(w2 == "patient") %>%
  select(w1, w2) %>%
  count(w1, sort = TRUE)
```

```
## # A tibble: 269 x 2
##    w1            n
##    <chr>     <int>
##  1 the       20307
##  2 this        470
##  3 history     101
##  4 a            67
##  5 and          47
##  6 procedure    32
##  7 female       26
##  8 with         25
##  9 use          24
## 10 old          23
## # … with 259 more rows
```

Since we are looking at single words again, it is a good idea to treat these as
single tokens. So let's remove the stopwords and the numbers


```r
bigrams %>%
  filter(w1 == "patient") %>%
  filter(!(w2 %in% stop_words$word) & !grepl("^[0-9]+$", w2)) %>%
  count(w2, sort = TRUE) %>%
  top_n(10) %>%
  knitr::kable(caption = "Words AFTER 'patient'")
```

```
## Selecting by n
```



Table: Words AFTER 'patient'

|w2         |   n|
|:----------|---:|
|tolerated  | 994|
|denies     | 552|
|underwent  | 180|
|received   | 160|
|reports    | 155|
|understood | 113|
|lives      |  81|
|admits     |  69|
|appears    |  68|
|including  |  67|

```r
bigrams %>%
  filter(w2 == "patient") %>%
  filter(!(w1 %in% stop_words$word) & !grepl("^[0-9]+$", w1)) %>%
  count(w1, sort = TRUE) %>%
  top_n(10) %>%
  knitr::kable(caption = "Words BEFORE 'patient'")
```

```
## Selecting by n
```



Table: Words BEFORE 'patient'

|w1          |   n|
|:-----------|---:|
|history     | 101|
|procedure   |  32|
|female      |  26|
|sample      |  23|
|male        |  22|
|illness     |  16|
|plan        |  16|
|indications |  15|
|allergies   |  14|
|correct     |  11|
|detail      |  11|



## Question 6

Which words are most used in each of the specialties. you can use group_by() and top_n() from dplyr to have the calculations be done within each specialty. Remember to remove stopwords. How about the most 5 used words?


```r
mtsamples %>% 
  unnest_tokens(output = word, input = transcription) %>%
  group_by(medical_specialty) %>%
  count(word, sort = TRUE) %>%
  filter(!(word %in% stop_words$word) & !grepl("^[0-9]+$", word)) %>%
  top_n(5) %>%
  arrange(medical_specialty, desc(n)) %>%
  knitr::kable()
```

```
## Selecting by n
```



|medical_specialty             |word         |    n|
|:-----------------------------|:------------|----:|
|Allergy / Immunology          |history      |   38|
|Allergy / Immunology          |noted        |   23|
|Allergy / Immunology          |patient      |   22|
|Allergy / Immunology          |allergies    |   21|
|Allergy / Immunology          |nasal        |   13|
|Allergy / Immunology          |past         |   13|
|Autopsy                       |left         |   83|
|Autopsy                       |inch         |   59|
|Autopsy                       |neck         |   55|
|Autopsy                       |anterior     |   47|
|Autopsy                       |body         |   40|
|Bariatrics                    |patient      |   62|
|Bariatrics                    |history      |   50|
|Bariatrics                    |weight       |   36|
|Bariatrics                    |surgery      |   34|
|Bariatrics                    |gastric      |   30|
|Cardiovascular / Pulmonary    |left         | 1550|
|Cardiovascular / Pulmonary    |patient      | 1516|
|Cardiovascular / Pulmonary    |artery       | 1085|
|Cardiovascular / Pulmonary    |coronary     |  681|
|Cardiovascular / Pulmonary    |history      |  654|
|Chiropractic                  |pain         |  187|
|Chiropractic                  |patient      |   85|
|Chiropractic                  |dr           |   66|
|Chiropractic                  |history      |   56|
|Chiropractic                  |left         |   54|
|Consult - History and Phy.    |patient      | 3046|
|Consult - History and Phy.    |history      | 2820|
|Consult - History and Phy.    |normal       | 1368|
|Consult - History and Phy.    |pain         | 1153|
|Consult - History and Phy.    |mg           |  908|
|Cosmetic / Plastic Surgery    |patient      |  116|
|Cosmetic / Plastic Surgery    |procedure    |   98|
|Cosmetic / Plastic Surgery    |breast       |   95|
|Cosmetic / Plastic Surgery    |skin         |   88|
|Cosmetic / Plastic Surgery    |incision     |   67|
|Dentistry                     |patient      |  195|
|Dentistry                     |tooth        |  108|
|Dentistry                     |teeth        |  104|
|Dentistry                     |left         |   94|
|Dentistry                     |procedure    |   82|
|Dermatology                   |patient      |  101|
|Dermatology                   |skin         |  101|
|Dermatology                   |cm           |   77|
|Dermatology                   |left         |   58|
|Dermatology                   |procedure    |   44|
|Diets and Nutritions          |patient      |   43|
|Diets and Nutritions          |weight       |   40|
|Diets and Nutritions          |carbohydrate |   37|
|Diets and Nutritions          |day          |   28|
|Diets and Nutritions          |food         |   27|
|Diets and Nutritions          |plan         |   27|
|Discharge Summary             |patient      |  672|
|Discharge Summary             |discharge    |  358|
|Discharge Summary             |mg           |  301|
|Discharge Summary             |history      |  208|
|Discharge Summary             |hospital     |  183|
|Emergency Room Reports        |patient      |  685|
|Emergency Room Reports        |history      |  356|
|Emergency Room Reports        |pain         |  273|
|Emergency Room Reports        |normal       |  255|
|Emergency Room Reports        |denies       |  149|
|Endocrinology                 |thyroid      |  129|
|Endocrinology                 |patient      |  121|
|Endocrinology                 |left         |   63|
|Endocrinology                 |history      |   57|
|Endocrinology                 |dissection   |   45|
|Endocrinology                 |gland        |   45|
|Endocrinology                 |nerve        |   45|
|ENT - Otolaryngology          |patient      |  415|
|ENT - Otolaryngology          |nasal        |  281|
|ENT - Otolaryngology          |left         |  219|
|ENT - Otolaryngology          |ear          |  182|
|ENT - Otolaryngology          |procedure    |  181|
|Gastroenterology              |patient      |  872|
|Gastroenterology              |procedure    |  470|
|Gastroenterology              |history      |  341|
|Gastroenterology              |normal       |  328|
|Gastroenterology              |colon        |  240|
|General Medicine              |patient      | 1356|
|General Medicine              |history      | 1027|
|General Medicine              |normal       |  717|
|General Medicine              |pain         |  567|
|General Medicine              |mg           |  503|
|Hematology - Oncology         |patient      |  316|
|Hematology - Oncology         |history      |  290|
|Hematology - Oncology         |left         |  187|
|Hematology - Oncology         |mg           |  107|
|Hematology - Oncology         |mass         |   97|
|Hospice - Palliative Care     |patient      |   43|
|Hospice - Palliative Care     |mg           |   28|
|Hospice - Palliative Care     |history      |   27|
|Hospice - Palliative Care     |daughter     |   22|
|Hospice - Palliative Care     |family       |   19|
|Hospice - Palliative Care     |pain         |   19|
|IME-QME-Work Comp etc.        |pain         |  152|
|IME-QME-Work Comp etc.        |patient      |  106|
|IME-QME-Work Comp etc.        |dr           |   82|
|IME-QME-Work Comp etc.        |injury       |   81|
|IME-QME-Work Comp etc.        |left         |   70|
|Lab Medicine - Pathology      |cm           |   35|
|Lab Medicine - Pathology      |tumor        |   35|
|Lab Medicine - Pathology      |lymph        |   30|
|Lab Medicine - Pathology      |lobe         |   29|
|Lab Medicine - Pathology      |upper        |   20|
|Letters                       |pain         |   80|
|Letters                       |abc          |   71|
|Letters                       |patient      |   65|
|Letters                       |normal       |   53|
|Letters                       |dr           |   46|
|Nephrology                    |patient      |  348|
|Nephrology                    |renal        |  257|
|Nephrology                    |history      |  160|
|Nephrology                    |kidney       |  144|
|Nephrology                    |left         |  132|
|Neurology                     |left         |  672|
|Neurology                     |patient      |  648|
|Neurology                     |normal       |  485|
|Neurology                     |history      |  429|
|Neurology                     |time         |  278|
|Neurosurgery                  |patient      |  374|
|Neurosurgery                  |c5           |  289|
|Neurosurgery                  |c6           |  266|
|Neurosurgery                  |procedure    |  247|
|Neurosurgery                  |left         |  222|
|Obstetrics / Gynecology       |patient      |  628|
|Obstetrics / Gynecology       |uterus       |  317|
|Obstetrics / Gynecology       |procedure    |  301|
|Obstetrics / Gynecology       |incision     |  293|
|Obstetrics / Gynecology       |normal       |  276|
|Office Notes                  |normal       |  230|
|Office Notes                  |negative     |  193|
|Office Notes                  |patient      |   94|
|Office Notes                  |history      |   76|
|Office Notes                  |noted        |   60|
|Ophthalmology                 |eye          |  456|
|Ophthalmology                 |patient      |  258|
|Ophthalmology                 |procedure    |  176|
|Ophthalmology                 |anterior     |  150|
|Ophthalmology                 |chamber      |  149|
|Orthopedic                    |patient      | 1711|
|Orthopedic                    |left         |  998|
|Orthopedic                    |pain         |  763|
|Orthopedic                    |procedure    |  669|
|Orthopedic                    |lateral      |  472|
|Pain Management               |patient      |  236|
|Pain Management               |procedure    |  197|
|Pain Management               |needle       |  156|
|Pain Management               |injected     |   76|
|Pain Management               |pain         |   76|
|Pediatrics - Neonatal         |patient      |  247|
|Pediatrics - Neonatal         |history      |  235|
|Pediatrics - Neonatal         |normal       |  155|
|Pediatrics - Neonatal         |child        |   82|
|Pediatrics - Neonatal         |mom          |   82|
|Physical Medicine - Rehab     |patient      |  220|
|Physical Medicine - Rehab     |left         |  104|
|Physical Medicine - Rehab     |pain         |   95|
|Physical Medicine - Rehab     |motor        |   62|
|Physical Medicine - Rehab     |history      |   54|
|Podiatry                      |foot         |  232|
|Podiatry                      |patient      |  231|
|Podiatry                      |left         |  137|
|Podiatry                      |tendon       |   98|
|Podiatry                      |incision     |   96|
|Psychiatry / Psychology       |patient      |  532|
|Psychiatry / Psychology       |history      |  344|
|Psychiatry / Psychology       |mg           |  183|
|Psychiatry / Psychology       |mother       |  164|
|Psychiatry / Psychology       |reported     |  141|
|Radiology                     |left         |  701|
|Radiology                     |normal       |  644|
|Radiology                     |patient      |  304|
|Radiology                     |exam         |  302|
|Radiology                     |mild         |  242|
|Rheumatology                  |history      |   50|
|Rheumatology                  |patient      |   34|
|Rheumatology                  |mg           |   26|
|Rheumatology                  |pain         |   23|
|Rheumatology                  |day          |   22|
|Rheumatology                  |examination  |   22|
|Rheumatology                  |joints       |   22|
|Sleep Medicine                |sleep        |  143|
|Sleep Medicine                |patient      |   69|
|Sleep Medicine                |apnea        |   35|
|Sleep Medicine                |activity     |   31|
|Sleep Medicine                |stage        |   29|
|SOAP / Chart / Progress Notes |patient      |  537|
|SOAP / Chart / Progress Notes |mg           |  302|
|SOAP / Chart / Progress Notes |history      |  254|
|SOAP / Chart / Progress Notes |pain         |  239|
|SOAP / Chart / Progress Notes |blood        |  194|
|Speech - Language             |patient      |  105|
|Speech - Language             |therapy      |   41|
|Speech - Language             |speech       |   35|
|Speech - Language             |patient's    |   28|
|Speech - Language             |evaluation   |   17|
|Speech - Language             |goals        |   17|
|Speech - Language             |term         |   17|
|Speech - Language             |time         |   17|
|Surgery                       |patient      | 4855|
|Surgery                       |left         | 3263|
|Surgery                       |procedure    | 3243|
|Surgery                       |anesthesia   | 1687|
|Surgery                       |incision     | 1641|
|Urology                       |patient      |  776|
|Urology                       |bladder      |  357|
|Urology                       |procedure    |  306|
|Urology                       |left         |  288|
|Urology                       |history      |  196|

The table shown above is top 5 word per medical specialty. But if there exist ties, like Allergy / Immunology, which contains different words but same times, it also will show up.

## Question 7

Find your own insight in the data:

According to the Question 6,It is not surprisingly to see that all categories contains *patient*. Some top words also contains the *left*, *history*. And at the same time we can clearly observe that the top 5 words in each category can directly explain what this category mean, or what the specific function of this category is, for example: *Bariatrics --- gastric*, *Cardiovascular / Pulmonary --- coronary and artery*. As some not professional persons, they cannot directly understand the medical words, these top words which can easily to show normal people which category is.
