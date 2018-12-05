Qualtrics Import Workshop
================
Ungson, Nick D.

Created: 2018-11-26 | *Last updated: 2018-12-05 09:17:09*

In this document, I will walk you through my process of using R to (1) **Import and tidy** raw Qualtrics data, (2) **Calculate** variables, and (3) **Analyze** that data.

This document contains code snippets and their respective output, but you can easily copy and paste the code below and adapt it for your own data and analyses!

Preliminaries
=============

Study Design and Variables
--------------------------

The sample data come from a **3** (Tweet: animal vs. funny vs. baseline) x **2** (Animal: cat vs. dog) mixed design study.

Independent Variables:

-   **Tweet** was the between-subjects variable: Participants either viewed (1) funny animal-themed tweets \[*Animal* condition\], (2) funny non-animal funny tweets \[*Funny* condition\], or (3) no tweets \[*Baseline*\].
-   **Animal** was the within-subjects variable. Participants completed the shortened Big Five Inventory (BFI, see below) in the third-“person” with regards to a (1) *dog* and (2) *cat*, in counterbalanced order.

Dependent Variable: The BFI has five subscales: **Openness to Experience, Conscientiousness, Extraversion, Agreeableness, Neuroticism**. Extraversion and Agreeableness were measured using three items; the remaining subscales were measured using two items. Each item was scored on a scale of 1 to 5, with higher scores indicating "more" of that trait.

Other Measured Variables: For each of the six tweets (if participants were assigned to *Animal* or *Funny* conditions), participants answered three questions regarding perceived funniness, likilihood to "fav", and likelihood to "share" that tweet on a scale from 1 (*not at all funny*/*not at all likely*) to 5 (*extremely funny*/*extremely likely*).

Load R Packages
---------------

First, load the `tidyverse` package. If you don't have this package installed, running `install.packages("tidyverse")` (which is commented out in the code below) will install the package. You only need to install each package once, but you need to load the package using `require()` or `library()` (they're the same) every time you start a new R session. The `tidyverse` package includes and automatically loads other cool packages, like \``ggplot` and `dplyr`, so you don't have to individually load those if you are loading the entire "tidyverse," so to speak.

``` r
#install.packages("tidyverse")
require(tidyverse)
```

Import and Tidy
===============

Import raw data
---------------

First, import the "fresh" raw data from Qualtrics and save as a dataframe called `raw`. As long as the current R Markdown document is in the same directory as `"raw qualtrics data.csv"`, the full file name is all you need to put in `read.csv()`.

``` r
raw <- read.csv("raw qualtrics data.csv", 
                header = TRUE, 
                stringsAsFactors = FALSE)
```

Before moving on, convert `raw` to a tibble (pun on "table"?), which works much more nicely than data frames with the `dplyr` package (which is automatically loaded as part of the `tidyverse`). ([For more info on tibbles](https://cran.r-project.org/web/packages/tibble/vignettes/tibble.html)).

``` r
# coerce to tibble
raw <- as_tibble(raw)
```

Tidy
----

**Remove superfluous rows** First, let's take a look at the first 10 rows and first 5 columns of `raw`. You can click the arrows in the output below to scroll through the columns.

``` r
raw[1:10, 1:5]
```

    ## # A tibble: 10 x 5
    ##    StartDate          EndDate          Status     IPAddress    Progress   
    ##    <chr>              <chr>            <chr>      <chr>        <chr>      
    ##  1 Start Date         End Date         Response ~ IP Address   Progress   
    ##  2 "{\"ImportId\":\"~ "{\"ImportId\":~ "{\"Impor~ "{\"ImportI~ "{\"Import~
    ##  3 2018-11-19 13:43:~ 2018-11-19 13:4~ 0          128.180.132~ 100        
    ##  4 2018-11-19 13:52:~ 2018-11-19 13:5~ 0          128.180.132~ 100        
    ##  5 2018-11-19 14:03:~ 2018-11-19 14:0~ 0          64.121.123.~ 100        
    ##  6 2018-11-19 14:05:~ 2018-11-19 14:1~ 0          128.180.107~ 100        
    ##  7 2018-11-19 14:13:~ 2018-11-19 14:1~ 0          128.180.96.~ 100        
    ##  8 2018-11-19 14:17:~ 2018-11-19 14:2~ 0          70.167.95.1~ 100        
    ##  9 2018-11-19 14:24:~ 2018-11-19 14:2~ 0          149.31.125.~ 100        
    ## 10 2018-11-19 14:39:~ 2018-11-19 14:4~ 0          149.31.82.1~ 100

Something that should jump out at you is that rows 1 and 2 contain unnecessary information. Row 1 simply repeats the question labels; although for later columns, this row contains question wording and you could need them for some studies. Row 2 contains metadata for each question. For our purposes, you don't want either of them. **Remove rows 1 and 2** by using indexing (`[]`) and using the negative sign (`-`) to exclude rows 1 and 2 (`-c(1:2)`) and overwrite `raw`.

``` r
# remove 1st and 2nd row (unnecessary qualtrics metadata)
raw <- raw[-c(1:2), ]
```

**Remove superfluous columns** Next, let's take a look at the column headings in `raw`.

``` r
colnames(raw)
```

    ##  [1] "StartDate"             "EndDate"              
    ##  [3] "Status"                "IPAddress"            
    ##  [5] "Progress"              "Duration..in.seconds."
    ##  [7] "Finished"              "RecordedDate"         
    ##  [9] "ResponseId"            "RecipientLastName"    
    ## [11] "RecipientFirstName"    "RecipientEmail"       
    ## [13] "ExternalReference"     "LocationLatitude"     
    ## [15] "LocationLongitude"     "DistributionChannel"  
    ## [17] "UserLanguage"          "anim1_1"              
    ## [19] "anim1_2"               "anim1_3"              
    ## [21] "anim2_1"               "anim2_2"              
    ## [23] "anim2_3"               "anim3_1"              
    ## [25] "anim3_2"               "anim3_3"              
    ## [27] "anim4_1"               "anim4_2"              
    ## [29] "anim4_3"               "anim5_1"              
    ## [31] "anim5_2"               "anim5_3"              
    ## [33] "anim6_1"               "anim6_2"              
    ## [35] "anim6_3"               "fun1_1"               
    ## [37] "fun1_2"                "fun1_3"               
    ## [39] "fun2_1"                "fun2_2"               
    ## [41] "fun2_3"                "fun3_1"               
    ## [43] "fun3_2"                "fun3_3"               
    ## [45] "fun4_1"                "fun4_2"               
    ## [47] "fun4_3"                "fun5_1"               
    ## [49] "fun5_2"                "fun5_3"               
    ## [51] "fun6_1"                "fun6_2"               
    ## [53] "fun6_3"                "dog_extra1r"          
    ## [55] "dog_extra3"            "dog_agree1"           
    ## [57] "dog_agree3"            "dog_cons1r"           
    ## [59] "dog_neur1r"            "dog_open1r"           
    ## [61] "dog_extra2"            "dog_agree2r"          
    ## [63] "dog_cons2"             "dog_neur2"            
    ## [65] "dog_open2"             "cat_extra1r"          
    ## [67] "cat_extra3"            "cat_agree1"           
    ## [69] "cat_agree3"            "cat_cons1r"           
    ## [71] "cat_neur1r"            "cat_open1r"           
    ## [73] "cat_extra2"            "cat_agree2r"          
    ## [75] "cat_cons2"             "cat_neur2"            
    ## [77] "cat_open2"             "age"                  
    ## [79] "gender"                "gender_fr"            
    ## [81] "lehigh_status"         "check_manipulation"   
    ## [83] "condition"

The variable `$anim1_2` is the first variable that is part of experimental study. However, you can see that Qualtrics has added 17 variables/columns to the beginning of our data. Sometimes you may want to retain some of them (e.g., `$StartDate` or `$EndDate`), but for now let's only keep study progress (`$Progress`) measured from 0-100, and time of completion (`$Duration..in.seconds.`) measured in second.

Importantly, since you are now excluding variables, create a new data frame called `data` and use the `select()` function to choose which variables to keep from `raw` and to rename when necessary (e.g., `$Duration..in.seconds.` is renamed as `$time_sec`). This way, the raw data remains relatively untouched in case you need to go back to it later.

*Note*: The code below uses the pipe operator `%>%` from the `dplyr` package. Basically, the operator takes whatever came before it (below, the `raw` data) and subjects it to what comes after: selecting variables using `select()`. One great thing about `dplyr` is you won't have to use the `$` operator to identify variables as much.

Check out the following guides on `dplyr` that I return to *all the time*:

-   [Introduction to dplyr](https://cran.r-project.org/web/packages/dplyr/vignettes/dplyr.html)
-   [dplyr tutorial by MIT Biomedical Data Science](http://genomicsclass.github.io/book/pages/dplyr_tutorial.html)

``` r
data <- raw %>% 
  select(progress = Progress,
         time_sec = Duration..in.seconds., 
         # select all variables between $anim1_1 and $anim6_3
         anim1_1:anim6_3, 
         # select all variables that contain "fun"
         contains("fun"), 
         # select all dog and cat variables in order I specify
         dog_extra1r, dog_extra2, dog_extra3, 
         dog_agree1, dog_agree2r, dog_agree3, 
         dog_cons1r, dog_cons2, 
         dog_neur1r, dog_neur2, 
         dog_open1r, dog_open2, 
         cat_extra1r, cat_extra2, cat_extra3, 
         cat_agree1, cat_agree2r, cat_agree3, 
         cat_cons1r, cat_cons2, 
         cat_neur1r, cat_neur2, 
         cat_open1r, cat_open2, 
         # select all variables between age and condition
         age:condition)
```

Great! Now we have a working `data` dataframe that we can start to mess around with. Remember, you can always do stuff like `colnames(data)`, `glimpse(data)`, or `str(data)` to investigate anything further.

``` r
# look at variables in 'data'
colnames(data)
```

    ##  [1] "progress"           "time_sec"           "anim1_1"           
    ##  [4] "anim1_2"            "anim1_3"            "anim2_1"           
    ##  [7] "anim2_2"            "anim2_3"            "anim3_1"           
    ## [10] "anim3_2"            "anim3_3"            "anim4_1"           
    ## [13] "anim4_2"            "anim4_3"            "anim5_1"           
    ## [16] "anim5_2"            "anim5_3"            "anim6_1"           
    ## [19] "anim6_2"            "anim6_3"            "fun1_1"            
    ## [22] "fun1_2"             "fun1_3"             "fun2_1"            
    ## [25] "fun2_2"             "fun2_3"             "fun3_1"            
    ## [28] "fun3_2"             "fun3_3"             "fun4_1"            
    ## [31] "fun4_2"             "fun4_3"             "fun5_1"            
    ## [34] "fun5_2"             "fun5_3"             "fun6_1"            
    ## [37] "fun6_2"             "fun6_3"             "dog_extra1r"       
    ## [40] "dog_extra2"         "dog_extra3"         "dog_agree1"        
    ## [43] "dog_agree2r"        "dog_agree3"         "dog_cons1r"        
    ## [46] "dog_cons2"          "dog_neur1r"         "dog_neur2"         
    ## [49] "dog_open1r"         "dog_open2"          "cat_extra1r"       
    ## [52] "cat_extra2"         "cat_extra3"         "cat_agree1"        
    ## [55] "cat_agree2r"        "cat_agree3"         "cat_cons1r"        
    ## [58] "cat_cons2"          "cat_neur1r"         "cat_neur2"         
    ## [61] "cat_open1r"         "cat_open2"          "age"               
    ## [64] "gender"             "gender_fr"          "lehigh_status"     
    ## [67] "check_manipulation" "condition"

As part of importing data, R sometimes makes choices about what kind of data type each variable is (e.g., numeric, string, etc.), so the code below will ensure that the variables I *want* to be numeric are actually recognized as numeric by R. We'll do this using the now-familiar pipe operator (`%>%`) and `mutate_at()` ([source](https://twitter.com/SuzanBaert/status/984054401374523392)). For this example, I'm going to make everything numeric except `$gender_fr`, a free response item on which participants could specify their gender.

``` r
# coerce variables to numeric using column index
data <- data %>% 
  mutate_at(vars(1:64, 66:68), as.numeric)
```

Exclude subjects
----------------

For the current study, we will exclude any participants who **did not complete the study**. First, use `table()` to look at the frequencies of `$progress`. (*Note*: `table()` is my go-to function for getting frequencies; e.g., gender)

``` r
# frequency of progress
table(data$progress)
```

    ## 
    ##   1   2  31  35  40  44  59  75  91  99 100 
    ##   1   1   3   2   2   1  10   6   1  12 105

105 participants completely finished (`$progress == 100`), 12 got 99% of the way through, and then there's a smattering of others. This is something you'll have to decide for your studies; for now, exclude anyone who was below 99% completion.

Notice that the code below uses the pipe operator `%>%` from the `dplyr` package again. However instead of using `select()` to identify variables/columns to keep/drop, use `filter()` to identify participants/rows to keep/drop:

``` r
# keep only participants with at least 99% completion
data <- data %>% 
  filter(progress >= 99)
```

Next, exclude any participants **failed the manipulation check**; participants were asked to identify which tweet condition they were exposed to. Before that, though, let's see how many participants failed the check using `nrow()` and indexing. Note that you must use `data$` before variables because we are not using `dplyr` for this:

``` r
# in how many rows does check_manipulation not equal condition?
nrow(data[data$check_manipulation != data$condition, ])
```

    ## [1] 7

Ok so `nrow(data[data$check_manipulation != data$condition, ])` check failures. Use `filter()` to only include those who passed the check (i.e., if `check_manipulation == condition`).

``` r
data <- data %>% 
  filter(check_manipulation == condition)
```

Add participant number
----------------------

Next, add a unique participant number variable. If your own study already includes a participant number variable or unique identifier, this would be unnecessary. But having a unique subject variable for each participant is crucial for calculating participant-level variables (e.g., scale means). You'll be using another `dplyr` verb: `mutate()`, which is used to compute new variables:

``` r
data <- data %>% 
  mutate(participant = 1:n())
```

Concatenate across between-subjects variables
---------------------------------------------

The last thing to do tidy this data is to deal with missing values (also known as `NA`s) that have arisen due to between-subjects variables. For example, participants in the Animal tweet condition made ratings about the animal tweets they saw (e.g., `$anim1_1`, `$anim1_2`) but did not answer--and therefore have missing values--for all tweet items corresponding to the Funny tweet condition (e.g., `$fun1_1`, `$fun1_2`). As a demonstration of this:

``` r
# sample of NA's across animal vs. funny tweet items
data[1:4, c(69, 3:5, 21:22)]
```

    ## # A tibble: 4 x 6
    ##   participant anim1_1 anim1_2 anim1_3 fun1_1 fun1_2
    ##         <int>   <dbl>   <dbl>   <dbl>  <dbl>  <dbl>
    ## 1           1      NA      NA      NA      2      1
    ## 2           2       4       4       2     NA     NA
    ## 3           3      NA      NA      NA     NA     NA
    ## 4           4       4       2       3     NA     NA

Here we can see that participant \#1 was in the Funny condition, participant \#2 was in the Animal condition, and participant \#3 was in the Baseline condition (`NA` for all tiems). To concatenate across these columns and create new variables, use `coalesce()` within the now-familiar `mutate()` function. The code below may look clunky, and I'm sure you could write a function to this more elegantly, but it definitely works.

``` r
data <- data %>% 
  mutate(tweet1_1 = coalesce(anim1_1, fun1_1), 
         tweet1_2 = coalesce(anim1_2, fun1_2),
         tweet1_3 = coalesce(anim1_3, fun1_3), 
         tweet2_1 = coalesce(anim2_1, fun2_1), 
         tweet2_2 = coalesce(anim2_2, fun2_2),
         tweet2_3 = coalesce(anim2_3, fun2_3), 
         tweet3_1 = coalesce(anim3_1, fun3_1), 
         tweet3_2 = coalesce(anim3_2, fun3_2),
         tweet3_3 = coalesce(anim3_3, fun3_3), 
         tweet4_1 = coalesce(anim4_1, fun4_1), 
         tweet4_2 = coalesce(anim4_2, fun4_2),
         tweet4_3 = coalesce(anim4_3, fun4_3), 
         tweet5_1 = coalesce(anim5_1, fun5_1), 
         tweet5_2 = coalesce(anim5_2, fun5_2),
         tweet5_3 = coalesce(anim5_3, fun5_3), 
         tweet6_1 = coalesce(anim6_1, fun6_1), 
         tweet6_2 = coalesce(anim6_2, fun6_2),
         tweet6_3 = coalesce(anim6_3, fun6_3))

# if you want to check out the variables
#colnames(data)
```

Tidy: Done!
-----------

**Okay the data is now pretty tidy**. Many of the operations above were done separately to help explain the logic, but you could easily string many of the dplyr commands together depending on your needs/wants/desires. The code below would produce the exact same tidied data set.

``` r
data <- raw %>% 
  select(progress = Progress,
         time_sec = Duration..in.seconds., 
         anim1_1:anim6_3, 
         contains("fun"), 
         dog_extra1r, dog_extra2, dog_extra3, 
         dog_agree1, dog_agree2r, dog_agree3, 
         dog_cons1r, dog_cons2, 
         dog_neur1r, dog_neur2, 
         dog_open1r, dog_open2, 
         cat_extra1r, cat_extra2, cat_extra3, 
         cat_agree1, cat_agree2r, cat_agree3, 
         cat_cons1r, cat_cons2, 
         cat_neur1r, cat_neur2, 
         cat_open1r, cat_open2, 
         age:condition) %>% 
  mutate_at(vars(1:64, 66:68), as.numeric) %>% 
  filter(progress >= 99) %>% 
  filter(check_manipulation == condition) %>% 
  mutate(participant = 1:n(), 
         tweet1_1 = coalesce(anim1_1, fun1_1), 
         tweet1_2 = coalesce(anim1_2, fun1_2),
         tweet1_3 = coalesce(anim1_3, fun1_3), 
         tweet2_1 = coalesce(anim2_1, fun2_1), 
         tweet2_2 = coalesce(anim2_2, fun2_2),
         tweet2_3 = coalesce(anim2_3, fun2_3), 
         tweet3_1 = coalesce(anim3_1, fun3_1), 
         tweet3_2 = coalesce(anim3_2, fun3_2),
         tweet3_3 = coalesce(anim3_3, fun3_3), 
         tweet4_1 = coalesce(anim4_1, fun4_1), 
         tweet4_2 = coalesce(anim4_2, fun4_2),
         tweet4_3 = coalesce(anim4_3, fun4_3), 
         tweet5_1 = coalesce(anim5_1, fun5_1), 
         tweet5_2 = coalesce(anim5_2, fun5_2),
         tweet5_3 = coalesce(anim5_3, fun5_3), 
         tweet6_1 = coalesce(anim6_1, fun6_1), 
         tweet6_2 = coalesce(anim6_2, fun6_2),
         tweet6_3 = coalesce(anim6_3, fun6_3))
```

Calculate Variables
===================

Now, it's time to calculate some variables:

-   BFI: Extraversion (cat and dog)
-   BFI: Openness to Experience (cat and dog)
-   BFI: Conscientiousness (cat and dog)
-   BFI: Neuroticism (cat and dog)
-   BFI: Agreeableness (cat and dog)
-   Positive evaluation of tweets

BFI dimensions
--------------

**Extraversion** was measured for each animal using 3 items:

-   `$dog_extra1r` (reverse-coded)
-   `$dog_extra2`
-   `$dog_extra3`

Below, for each participant, calculate the mean dog extraversion by averaging these three items in a new variable `$dog_extra`. Some things to note:

-   `group_by()` is used to ensure that whatever comes afterwards is applied separately to each "group" specified. In other words, if you did not include `group_by(participant)`, you would calculate the grand dog extra version of mean across all participants (i.e., everyone would have the same score). By including `group_by(participant)`, you make sure that the mean calculation occurs for every "group" (participant). By this logic, you could also use `group_by(condition)` and this code would give you the mean dog extraversion for each of the three tweet conditions.
-   `(6 - dog_extra1r)` refers to the reverse-scoring of that item
-   `na.rm = TRUE` tells R to ignore missing values. Otherwise, if a participant happened to skip one dog extraversion item, no mean would be calculated and `NA` would be returned for that participant. This way, R calculates the mean from all available data.
-   The code below also calculates cat extraversion, `$cat_extra`, using the same logic; remmber, multiple variables can be created inside `mutate()` as long as you separate with commas!

``` r
data <- data %>% 
  group_by(participant) %>% 
  mutate(dog_extra = mean((6 - dog_extra1r), dog_extra2, dog_extra3, na.rm = TRUE), 
         cat_extra = mean((6 - cat_extra1r), cat_extra2, cat_extra3, na.rm = TRUE))
```

**Neuroticism** was measured for each animal using 2 items:

-   `$dog_neur1r` (reverse-coded)
-   `$dog_neur2`

Create means in the exact same way:

``` r
data <- data %>% 
  group_by(participant) %>% 
  mutate(dog_neur = mean((6 - dog_neur1r), dog_neur2, na.rm = TRUE), 
         cat_neur = mean((6 - cat_neur1r), cat_neur2, na.rm = TRUE))
```

...you could then easily do the same thing for the rest of the BFI variables:

``` r
data <- data %>% 
  group_by(participant) %>% 
  mutate(dog_agree = mean((6 - dog_agree2r), dog_agree1, dog_agree3, na.rm = TRUE), 
         cat_agree = mean((6 - cat_agree2r), cat_agree1, cat_agree3, na.rm = TRUE), 
         dog_cons = mean((6 - dog_cons1r), dog_cons2, na.rm = TRUE), 
         cat_cons = mean((6 - cat_cons1r), cat_cons2, na.rm = TRUE), 
         dog_open = mean((6 - dog_open1r), dog_open2, na.rm = TRUE), 
         cat_open = mean((6 - cat_open1r), cat_open2, na.rm = TRUE))
```

Tweet evaluation
----------------

``` r
data <- data %>% 
  group_by(participant) %>% 
  mutate(tweet_eval = mean(tweet1_1, tweet1_2, tweet1_3, 
                           tweet2_1, tweet2_2, tweet2_3, 
                           tweet3_1, tweet3_2, tweet3_3, 
                           tweet4_1, tweet4_2, tweet4_3, 
                           tweet5_1, tweet5_2, tweet5_3, 
                           tweet6_1, tweet6_2, tweet6_3, na.rm = TRUE))
```

Re-order variables
------------------

This is not strictly necessary, but I like to do one last "tidying" measure before starting with analyses: remove now-unnecessary variables (e.g., between-subjects items that are useless now that we've used `coalesce()` to concatenate; progress) and order variables to my liking. Remember, the order of variables passed through `select()` is the order they will be placed into resulting data.

``` r
data <- data %>% 
  select(participant, condition, 
         # all calculated variables
         dog_extra:tweet_eval, 
         # demographics, etc.
         time_sec, age:lehigh_status, 
         # individual scale items (retain for reliability analyses)
         tweet1_1:cat_open)
```

Take a moment to apreciate your clean data set:

``` r
colnames(data)
```

    ##  [1] "participant"   "condition"     "dog_extra"     "cat_extra"    
    ##  [5] "dog_neur"      "cat_neur"      "dog_agree"     "cat_agree"    
    ##  [9] "dog_cons"      "cat_cons"      "dog_open"      "cat_open"     
    ## [13] "tweet_eval"    "time_sec"      "age"           "gender"       
    ## [17] "gender_fr"     "lehigh_status" "tweet1_1"      "tweet1_2"     
    ## [21] "tweet1_3"      "tweet2_1"      "tweet2_2"      "tweet2_3"     
    ## [25] "tweet3_1"      "tweet3_2"      "tweet3_3"      "tweet4_1"     
    ## [29] "tweet4_2"      "tweet4_3"      "tweet5_1"      "tweet5_2"     
    ## [33] "tweet5_3"      "tweet6_1"      "tweet6_2"      "tweet6_3"

Optional: Export Data
=====================

At this point, you may want to export `data` in it's cleaned form, perhaps as a .csv file. You can use the `write.csv()` function for that ([source](http://rprogramming.net/write-csv-in-r/)). Something to remember is to set `row.names = FALSE` to prevent R from creating a new column with row numbers in your new .csv file. Besides, we already have `$participant` anyway!

Below is an example of how you might go about it:

``` r
write.csv(data,
          file = "2018-12-05 clean data.csv", 
          row.names = fALSE)
```

Analyze
=======

Descriptives
------------

There are few of ways to get descriptive statistics for your variables. I'll demonstrate a couple of them here.

You can get them "individually", like so:

``` r
mean(data$age)
```

    ## [1] 32.06364

``` r
sd(data$age)
```

    ## [1] 10.05971

You can use functions to return several statistics. For example, the `stat.desc()` function in the `pastecs` package:

``` r
#install.packages("pastecs")
require(pastecs)
stat.desc(data$age)
```

    ##      nbr.val     nbr.null       nbr.na          min          max 
    ##  110.0000000    0.0000000    0.0000000   20.0000000   72.0000000 
    ##        range          sum       median         mean      SE.mean 
    ##   52.0000000 3527.0000000   30.0000000   32.0636364    0.9591556 
    ## CI.mean.0.95          var      std.dev     coef.var 
    ##    1.9010153  101.1977481   10.0597091    0.3137420

A fun thing about arrays like the one `stat.desc()` spits out is that you can use `[]` to index and pull out specific values. For example, if you wanted to *just* return the mean of `$age`, you know that mean is the 9th thing in that object, so just pull it out!

``` r
# see what's "inside" stat.desc(data$age)
names(stat.desc(data$age))
```

    ##  [1] "nbr.val"      "nbr.null"     "nbr.na"       "min"         
    ##  [5] "max"          "range"        "sum"          "median"      
    ##  [9] "mean"         "SE.mean"      "CI.mean.0.95" "var"         
    ## [13] "std.dev"      "coef.var"

``` r
# pull out mean
stat.desc(data$age)[9]
```

    ##     mean 
    ## 32.06364

Another way is using the `describe()` function from the `psych` package:

``` r
require(psych)
```

    ## Loading required package: psych

    ## 
    ## Attaching package: 'psych'

    ## The following objects are masked from 'package:ggplot2':
    ## 
    ##     %+%, alpha

``` r
describe(data$age)
```

    ##    vars   n  mean    sd median trimmed  mad min max range skew kurtosis
    ## X1    1 110 32.06 10.06     30   30.09 2.97  20  72    52 1.99     3.71
    ##      se
    ## X1 0.96

**And the last way** demonstrated here is using our old friend `dplyr`, but now using the `summarize()` function. (As a fun twist, I've added `group_by(condition)`, which shows us the mean age by condition. Not that, since `<-` does not appear anywhere in the code below, nothing is saved or overwritten; the results are only printed.

``` r
# mean age for each condition
data %>% 
  group_by(condition) %>% 
  summarize(age_m = mean(age, na.rm = TRUE))
```

    ## # A tibble: 3 x 2
    ##   condition age_m
    ##       <dbl> <dbl>
    ## 1         1  31.1
    ## 2         2  31.6
    ## 3         3  33.3

ANOVA
-----

Run a one-way ANOVA on perceived extraversion: *Did participants in different Tweet conditions perceive more extraversion in animals, averaging across dogs and cats?*

There are *many* ways to conduct all types of analyses in R, but here we'll use the simple `aov()` function ([source](http://www.sthda.com/english/wiki/one-way-anova-test-in-r)).

First, I'm going to calculate a new variable that is mean extra version **across** dogs and cats.

``` r
data <- data %>% 
  group_by(participant) %>% 
  mutate(all_extra = mean(dog_extra, cat_extra, na.rm = TRUE))
```

Then use `aov()`, whose basic form is `aov(dv ~ iv, data = name_of_data)` and save the results in an object named `aov_extra`; this will let us continue to use that object for future analyses.

``` r
# save object
aov_extra <- aov(all_extra ~ condition, data = data)

# get summary of results
summary(aov_extra)
```

    ##              Df Sum Sq Mean Sq F value Pr(>F)
    ## condition     1   0.31  0.3095    0.24  0.625
    ## Residuals   108 139.01  1.2871

``` r
# tip: code below would simultaneously save and get summary
#summary(aov_extra <- aov(all_extra ~ condition, data = data))
```

So it doesn't look like there is a main effect of `$condition` on extraversion, averaged across dog and cat. We can see this is the case by looking at the descriptives across conditions using `group_by(condition)` and `summarize()`:

``` r
data %>% 
  group_by(condition) %>% 
  summarize(extra = mean(all_extra, na.rm = TRUE), 
            extra_sd = sd(all_extra, na.rm = TRUE))
```

    ## # A tibble: 3 x 3
    ##   condition extra extra_sd
    ##       <dbl> <dbl>    <dbl>
    ## 1         1  3.76     1.16
    ## 2         2  3.66     1.23
    ## 3         3  3.88     1.03

<sup>^</sup> Not surprising that the ANOVA was non-significant.

T-test
------

Now do a paired samples *t*-test to see if people differed in their perceived extraversion of dogs versus cats. For this, use `t.test()`, whose basic form is `t.test(dv1, dv2, paired = TRUE)` ([source](https://www.statmethods.net/stats/ttest.html)).

The code below does not do this, but you could always save results to an object (e.g., `extra_t`) to mess with later.

``` r
t.test(data$dog_extra, data$cat_extra, paired = TRUE)
```

    ## 
    ##  Paired t-test
    ## 
    ## data:  data$dog_extra and data$cat_extra
    ## t = 6.5896, df = 109, p-value = 1.618e-09
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  0.6483757 1.2061698
    ## sample estimates:
    ## mean of the differences 
    ##               0.9272727

Definitely significant. And the group means:

``` r
mean(data$dog_extra)
```

    ## [1] 3.772727

``` r
mean(data$cat_extra)
```

    ## [1] 2.845455

Simple regression
-----------------

Now try a simple linear regression. Regress perceived cat openness (`$cat_open`) onto age; *do older/younger people perceive cats as having different openness to experience?*

Most regression uses the `lm()` function, whose basic form is `lm(dv ~ iv, data = data)`. For regression analyses (and most others), you specify the model and can save the results in an object that can be modfied/explored further. In this regression example, save the regression into the object `cat_open_lm` and examine using `summary()`.

``` r
# define model
cat_open_lm <- lm(cat_open ~ age, data = data)

# summarize / get results
summary(cat_open_lm)
```

    ## 
    ## Call:
    ## lm(formula = cat_open ~ age, data = data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.95119 -0.94228  0.06233  1.06110  2.10288 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  2.975764   0.448674   6.632 1.35e-09 ***
    ## age         -0.001229   0.013357  -0.092    0.927    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1.403 on 108 degrees of freedom
    ## Multiple R-squared:  7.836e-05,  Adjusted R-squared:  -0.00918 
    ## F-statistic: 0.008464 on 1 and 108 DF,  p-value: 0.9269

Welp, not significant!

**ABSOLUTELY TANGENTIAL THING: Fun exploration of lm objects**

As a fun thing, you can "investigate" `lm()` objects like our `cat_open_lm`. Firstly, use `names()` to see the names of everything "inside" it.

``` r
names(cat_open_lm)
```

    ##  [1] "coefficients"  "residuals"     "effects"       "rank"         
    ##  [5] "fitted.values" "assign"        "qr"            "df.residual"  
    ##  [9] "xlevels"       "call"          "terms"         "model"

Cool! So all those things are kind of like "variables" inside the object `cat_open_lm`, which means we can use the `$` operator to "pull them out". For example, let's say I just wanted to get the list of regression coefficients, I could look only at `$coefficients` inside `cat_open_lm`.

``` r
cat_open_lm$coefficients
```

    ##  (Intercept)          age 
    ##  2.975764014 -0.001228818

Very cool! That gives us a matrix with the variable labels ("Intercept" and "age") in the first row, and their respective regression coefficients in the second row. OKAY but let's say I *only* wanted the regression coefficient for `$age`. I can use indexing (`[]`) to pull it out specifically by calling the label on top of that column, `"age"`.

``` r
cat_open_lm$coefficients["age"]
```

    ##          age 
    ## -0.001228818

``` r
# I could also use the column number for the same result
cat_open_lm$coefficients[2]
```

    ##          age 
    ## -0.001228818

**Plot regression line**. We can use R to plot the relationship between age and perceived cat openness ([source](https://www.statmethods.net/graphs/scatterplot.html)). There are more fancy ways to do this, but this is a simple (if not the prettiest) way to do it. First, plot a simple scatterplot of the points using the `plot()` function; then add the line of best fit to the scatterplot by using the `abline()` function. Notice that, inside the `abline()`, you should something similar to what we did when using `lm()` to create `cat_open_lm`: cat openness regressed onto age, `data$cat_open ~ data$age`.

``` r
plot(x = data$age, 
     y = data$cat_open, 
     xlab = "Age", 
     ylab = "Perceived Cat Openness")

abline(lm(data$cat_open ~ data$age), col = "blue")
```

![](qualtrics-import_files/figure-markdown_github/unnamed-chunk-39-1.png)

Advanced analyses, etc.
-----------------------

As you've no doubt noticed, a more appropriate analysis would be 3x3 mixed ANOVA with **Tweet** and **Animal** as the factors...however, mixed designs require you to restructure your data into "long format", which is something for another day. But you would probably use something like the `gather()` function in the `tidyr` package. ([source](http://www.cookbook-r.com/Manipulating_data/Converting_data_between_wide_and_long_format/))([source](https://tidyr.tidyverse.org/))

Once your data is in that format, I would probably use `ezANOVA()` function in the `ez` package. ([source](https://www.rdocumentation.org/packages/ez/versions/3.0-1/topics/ezANOVA))

Conclusion
==========

Well... that's it. It was a lot, and not very in-depth, but you should now have a pretty good idea of how to clean/examine/analyze your data in the future!

Pretty soon, you too can fully exert your will on technology, much like the goal-scoring robot below.

![](robot_soccer.gif)
