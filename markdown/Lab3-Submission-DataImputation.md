---
editor_options: 
  markdown: 
    wrap: 72
---

# Business Intelligence Lab Submission Markdown

Acers Team 9th October 2023

-   [Student Details](#student-details)
-   [Setup Chunk](#setup-chunk)
-   [Understanding the Dataset (Exploratory Data Analysis
    (EDA))](#understanding-the-dataset-exploratory-data-analysis-eda)
    -   [Downloading the Dataset](#downloading-the-dataset)
        -   [Source:](#source)
        -   [Reference:](#reference)
-   [STEP 1 : Install and load all the
    packages](#step-1--install-and-load-all-the-packages)
    -   [Loading the Dataset](#loading-the-dataset)
-   [STEP 2: Create a subset of the
    variables/features](#step-2-create-a-subset-of-the-variablesfeatures)
-   [STEP 3: Confirm the "missingness" in the Dataset before
    Imputation](#step-3-confirm-the-missingness-in-the-dataset-before-imputation)
-   [STEP 4. Use the MICE package to perform data
    imputation](#step-4-use-the-mice-package-to-perform-data-imputation)
-   [STEP 5. Confirm the "missingness" in the Imputed
    Dataset](#step-5-confirm-the-missingness-in-the-imputed-dataset)

# Student Details {#student-details}

|                                                   |                                                              |
|---------------------------------------------------|--------------------------------------------------------------|
| **Student ID Numbers and Names of Group Members** | 134879 Tulienge Lesley                                       |
|                                                   | 133834 Mongare Sarah                                         |
|                                                   | 122790 Bwalley Nicholas                                      |
|                                                   | 124461 Gitonga Angela                                        |
|                                                   | 133928 Cheptoi Millicent                                     |
| **BBIT 4.2 Group**                                | Group C                                                      |
| **Course Code**                                   | BBT4206                                                      |
| **Course Name**                                   | Business Intelligence II                                     |
| **Program**                                       | Bachelor of Business Information Technology                  |
| **Semester Duration**                             | 21<sup>st</sup> August 2023 to 28<sup>th</sup> November 2023 |

# Setup Chunk {#setup-chunk}

**Note:** the following KnitR options have been set as the global
defaults: <BR>
`knitr::opts_chunk$set(echo = TRUE, warning = FALSE, eval = TRUE, collapse = FALSE, tidy = TRUE)`.

More KnitR options are documented here
<https://bookdown.org/yihui/rmarkdown-cookbook/chunk-options.html> and
here <https://yihui.org/knitr/options/>.

# Understanding the Dataset (Exploratory Data Analysis (EDA)) {#understanding-the-dataset-exploratory-data-analysis-eda}

## Downloading the Dataset {#downloading-the-dataset}

### Source: {#source}

The dataset that was used can be downloaded here:
*\<<https://drive.google.com/drive/folders/1-BGEhfOwquXF6KKXwcvrx7WuZXuqmW9q?usp=sharing>\>*

### Reference: {#reference}

*\<Cite the dataset here using APA\>\
Refer to the APA 7th edition manual for rules on how to cite datasets:
<https://apastyle.apa.org/style-grammar-guidelines/references/examples/data-set-references>*

# STEP 1 : Install and load all the packages

We installed all the packages that will enable us execute this lab.

``` r
## dplyr ----
if (!is.element("dplyr", installed.packages()[, 1])) {
  install.packages("dplyr", dependencies = TRUE,
                   repos = "https://cloud.r-project.org")
}
require("dplyr")
```

```         
## Loading required package: dplyr

## 
## Attaching package: 'dplyr'

## The following objects are masked from 'package:stats':
## 
##     filter, lag

## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

``` r
## naniar ----
# Documentation:
#   https://cran.r-project.org/package=naniar or
#   https://www.rdocumentation.org/packages/naniar/versions/1.0.0
if (!is.element("naniar", installed.packages()[, 1])) {
  install.packages("naniar", dependencies = TRUE,
                   repos = "https://cloud.r-project.org")
}
require("naniar")
```

```         
## Loading required package: naniar
```

``` r
## ggplot2 ----
# We require the "ggplot2" package to create more appealing visualizations
if (!is.element("ggplot2", installed.packages()[, 1])) {
  install.packages("ggplot2", dependencies = TRUE,
                   repos = "https://cloud.r-project.org")
}
require("ggplot2")
```

```         
## Loading required package: ggplot2
```

``` r
## MICE ----
# We use the MICE package to perform data imputation
if (!is.element("mice", installed.packages()[, 1])) {
  install.packages("mice", dependencies = TRUE,
                   repos = "https://cloud.r-project.org")
}
require("mice")
```

```         
## Loading required package: mice

## 
## Attaching package: 'mice'

## The following object is masked from 'package:stats':
## 
##     filter

## The following objects are masked from 'package:base':
## 
##     cbind, rbind
```

``` r
## Amelia ----
if (!is.element("Amelia", installed.packages()[, 1])) {
  install.packages("Amelia", dependencies = TRUE,
                   repos = "https://cloud.r-project.org")
}
require("Amelia")
```

```         
## Loading required package: Amelia

## Loading required package: Rcpp

## ## 
## ## Amelia II: Multiple Imputation
## ## (Version 1.8.1, built: 2022-11-18)
## ## Copyright (C) 2005-2023 James Honaker, Gary King and Matthew Blackwell
## ## Refer to http://gking.harvard.edu/amelia/ for more information
## ##
```

## Loading the Dataset {#loading-the-dataset}

``` r
student_performance_dataset <-
  readr::read_csv(
    "data/20230412-20230719-BI1-BBIT4-1-StudentPerformanceDataset.CSV", # nolint
    col_types =
      readr::cols(
        class_group =
          readr::col_factor(levels = c("A", "B", "C")),
        gender = readr::col_factor(levels = c("1", "0")),
        YOB = readr::col_date(format = "%Y"),
        regret_choosing_bi =
          readr::col_factor(levels = c("1", "0")),
        drop_bi_now =
          readr::col_factor(levels = c("1", "0")),
        motivator =
          readr::col_factor(levels = c("1", "0")),
        read_content_before_lecture =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        anticipate_test_questions =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        answer_rhetorical_questions =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        find_terms_I_do_not_know =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        copy_new_terms_in_reading_notebook =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        take_quizzes_and_use_results =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        reorganise_course_outline =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        write_down_important_points =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        space_out_revision =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        studying_in_study_group =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        schedule_appointments =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        goal_oriented =
          readr::col_factor(levels =
                              c("1", "0")),
        spaced_repetition =
          readr::col_factor(levels =
                              c("1", "2", "3", "4")),
        testing_and_active_recall =
          readr::col_factor(levels =
                              c("1", "2", "3", "4")),
        interleaving =
          readr::col_factor(levels =
                              c("1", "2", "3", "4")),
        categorizing =
          readr::col_factor(levels =
                              c("1", "2", "3", "4")),
        retrospective_timetable =
          readr::col_factor(levels =
                              c("1", "2", "3", "4")),
        cornell_notes =
          readr::col_factor(levels =
                              c("1", "2", "3", "4")),
        sq3r = readr::col_factor(levels =
                                   c("1", "2", "3", "4")),
        commute = readr::col_factor(levels =
                                      c("1", "2",
                                        "3", "4")),
        study_time = readr::col_factor(levels =
                                         c("1", "2",
                                           "3", "4")),
        repeats_since_Y1 = readr::col_integer(),
        paid_tuition = readr::col_factor(levels =
                                           c("0", "1")),
        free_tuition = readr::col_factor(levels =
                                           c("0", "1")),
        extra_curricular = readr::col_factor(levels =
                                               c("0", "1")),
        sports_extra_curricular =
          readr::col_factor(levels = c("0", "1")),
        exercise_per_week = readr::col_factor(levels =
                                                c("0", "1",
                                                  "2",
                                                  "3")),
        meditate = readr::col_factor(levels =
                                       c("0", "1",
                                         "2", "3")),
        pray = readr::col_factor(levels =
                                   c("0", "1",
                                     "2", "3")),
        internet = readr::col_factor(levels =
                                       c("0", "1")),
        laptop = readr::col_factor(levels = c("0", "1")),
        family_relationships =
          readr::col_factor(levels =
                              c("1", "2", "3", "4", "5")),
        friendships = readr::col_factor(levels =
                                          c("1", "2", "3",
                                            "4", "5")),
        romantic_relationships =
          readr::col_factor(levels =
                              c("0", "1", "2", "3", "4")),
        spiritual_wellnes =
          readr::col_factor(levels = c("1", "2", "3",
                                       "4", "5")),
        financial_wellness =
          readr::col_factor(levels = c("1", "2", "3",
                                       "4", "5")),
        health = readr::col_factor(levels = c("1", "2",
                                              "3", "4",
                                              "5")),
        day_out = readr::col_factor(levels = c("0", "1",
                                               "2", "3")),
        night_out = readr::col_factor(levels = c("0",
                                                 "1", "2",
                                                 "3")),
        alcohol_or_narcotics =
          readr::col_factor(levels = c("0", "1", "2", "3")),
        mentor = readr::col_factor(levels = c("0", "1")),
        mentor_meetings = readr::col_factor(levels =
                                              c("0", "1",
                                                "2", "3")),
        `Attendance Waiver Granted: 1 = Yes, 0 = No` =
          readr::col_factor(levels = c("0", "1")),
        GRADE = readr::col_factor(levels =
                                    c("A", "B", "C", "D",
                                      "E"))),
    locale = readr::locale())
```

# STEP 2: Create a subset of the variables/features {#step-2-create-a-subset-of-the-variablesfeatures}

We created a subset of the variables to be included in the new dataset.
We 14 features and 80 random observations to be included in the dataset.

``` r
# We select only the following 14 features to be included in the dataset:
student_dataset <- student_performance_dataset %>%
  select(class_group ,gender, YOB,regret_choosing_bi, drop_bi_now, motivator, friendships, spiritual_wellnes, financial_wellness,
        health, alcohol_or_narcotics,  mentor,  mentor_meetings,  sports_extra_curricular,  romantic_relationships)

# We then select 80 random observations to be included in the dataset
rand_ind <- sample(seq_len(nrow(student_dataset)), 80)
student_performance_dataset <-student_dataset[rand_ind, ]
```

# STEP 3: Confirm the "missingness" in the Dataset before Imputation {#step-3-confirm-the-missingness-in-the-dataset-before-imputation}

``` r
# Are there missing values in the dataset?
any_na(student_performance_dataset)
```

```         
## [1] TRUE
```

``` r
# How many?
n_miss(student_performance_dataset)
```

```         
## [1] 9
```

``` r
# What is the percentage of missing data in the entire dataset?
prop_miss(student_performance_dataset)
```

```         
## [1] 0.0075
```

``` r
# How many missing values does each variable have?
student_performance_dataset %>% is.na() %>% colSums()
```

```         
##             class_group                  gender                     YOB 
##                       0                       0                       0 
##      regret_choosing_bi             drop_bi_now               motivator 
##                       0                       0                       0 
##             friendships       spiritual_wellnes      financial_wellness 
##                       1                       1                       1 
##                  health    alcohol_or_narcotics                  mentor 
##                       1                       1                       1 
##         mentor_meetings sports_extra_curricular  romantic_relationships 
##                       1                       1                       1
```

``` r
# What is the number and percentage of missing values grouped by
# each variable?
miss_var_summary(student_performance_dataset)
```

```         
## # A tibble: 15 × 3
##    variable                n_miss pct_miss
##    <chr>                    <int>    <dbl>
##  1 friendships                  1     1.25
##  2 spiritual_wellnes            1     1.25
##  3 financial_wellness           1     1.25
##  4 health                       1     1.25
##  5 alcohol_or_narcotics         1     1.25
##  6 mentor                       1     1.25
##  7 mentor_meetings              1     1.25
##  8 sports_extra_curricular      1     1.25
##  9 romantic_relationships       1     1.25
## 10 class_group                  0     0   
## 11 gender                       0     0   
## 12 YOB                          0     0   
## 13 regret_choosing_bi           0     0   
## 14 drop_bi_now                  0     0   
## 15 motivator                    0     0
```

``` r
# What is the number and percentage of missing values grouped by
# each observation?
miss_case_summary(student_performance_dataset)
```

```         
## # A tibble: 80 × 3
##     case n_miss pct_miss
##    <int>  <int>    <dbl>
##  1    11      9       60
##  2     1      0        0
##  3     2      0        0
##  4     3      0        0
##  5     4      0        0
##  6     5      0        0
##  7     6      0        0
##  8     7      0        0
##  9     8      0        0
## 10     9      0        0
## # ℹ 70 more rows
```

``` r
# Which variables contain the most missing values?
gg_miss_var(student_performance_dataset)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20fourth%20Code%20Chunk-1.png)<!-- -->

``` r
# Where are missing values located (the shaded regions in the plot)?
vis_miss(student_performance_dataset) + theme(axis.text.x = element_text(angle = 80))
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20fourth%20Code%20Chunk-2.png)<!-- -->

``` r
# Which combinations of variables are missing together?
#gg_miss_upset(student_performance_dataset)

# Create a heatmap of "missingness" broken down by " gender"
# First, confirm that the " gender" variable categorical variable
is.factor(student_performance_dataset$ gender)
```

```         
## [1] TRUE
```

``` r
# Second, create the visualization
gg_miss_fct(student_performance_dataset, fct = gender)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20fourth%20Code%20Chunk-3.png)<!-- -->

``` r
# We can also create a heatmap of "missingness" broken down by " class_group"
# First, confirm that the " class_group" variable is a categorical variable
is.factor(student_performance_dataset$ class_group)
```

```         
## [1] TRUE
```

``` r
# Second, create the visualization
gg_miss_fct(student_performance_dataset, fct =  class_group)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20fourth%20Code%20Chunk-4.png)<!-- -->

# STEP 4. Use the MICE package to perform data imputation

``` r
# We can use the dplyr::mutate() function inside the dplyr package to add new
# variables that are functions of existing variables

# In this case, it is used to create a new variable called,
# "Median Arterial Pressure (MAP)"
# Further reading:
#   https://en.wikipedia.org/wiki/Mean_arterial_pressure

# Convert factor variables to numeric or integer
student_performance_dataset$mentor <- as.numeric(student_performance_dataset$mentor)
student_performance_dataset$mentor_meetings <- as.numeric(student_performance_dataset$mentor_meetings)

# Perform the arithmetic operation
student_performance_dataset <- student_performance_dataset %>%
  mutate(MAP = mentor + (1/3) * (mentor_meetings - mentor))

# MAP can be positively correlated with "BMI", unfortunately, BMI was reported
# to have approximately 4.8% missing values.

# MAP can also be negatively correlated with "PhysActiveDays" which had
# approximately 55% missing data.

# We finally begin to make use of Multivariate Imputation by Chained
# Equations (MICE). We use 11 multiple imputations.

# To arrive at good predictions for each variable containing missing values, we
# save the variables that are at least "somewhat correlated" (r > 0.3).
somewhat_correlated_variables <- quickpred(student_performance_dataset, mincor = 0.3) # nolint

# m = 11 Specifies that the imputation (filling in the missing data) will be
#         performed 11 times (multiple times) to create several complete
#         datasets before we pool the results to arrive at a more realistic
#         final result. The larger the value of "m" and the larger the dataset,
#         longer the data imputation will take.
# seed = 7 Specifies that number 7 will be used to offset the random number
#         generator used by mice. This is so that we get the same results
#         each time we run MICE.
# meth = "pmm" Specifies the imputation method. "pmm" stands for "Predictive
#         Mean Matching" and it can be used for numeric data.# Used when the various intregers,observations ad statements are implemented or evaluated all together 
#         Other methods include:
#         1. "logreg": logistic regression imputation; used
#            for binary categorical data
#         2. "polyreg": Polytomous Regression Imputation for unordered
#            categorical data with more than 2 categories, and
#         3. "polr": Proportional Odds model for ordered categorical
#            data with more than 2 categories.
student_performance_dataset_mice <- mice(student_performance_dataset, m = 11, method = "pmm",
                            seed = 7,
                            predictorMatrix = somewhat_correlated_variables)
```

```         
## 
##  iter imp variable
##   1   1  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   2  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   3  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   4  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   5  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   6  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   7  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   8  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   9  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   10  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   1   11  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   1  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   2  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   3  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   4  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   5  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   6  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   7  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   8  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   9  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   10  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   2   11  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   1  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   2  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   3  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   4  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   5  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   6  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   7  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   8  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   9  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   10  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   3   11  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   1  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   2  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   3  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   4  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   5  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   6  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   7  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   8  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   9  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   10  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   4   11  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   1  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   2  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   3  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   4  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   5  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   6  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   7  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   8  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   9  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   10  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
##   5   11  friendships  spiritual_wellnes  financial_wellness  health  alcohol_or_narcotics  mentor  mentor_meetings  sports_extra_curricular  romantic_relationships  MAP
```

``` r
# One can then train a model to predict MAP using BMI and PhysActiveDays or to
# identify the p-Value and confidence intervals between MAP and BMI and
# PhysActiveDays
  
# We can use multiple scatter plots (a.k.a. strip-plots) to visualize how
# random the imputed data is in each of the 11 datasets.#Blue dots is the existing data 
stripplot(student_performance_dataset_mice,
          MAP ~  health | .imp,
          pch = 20, cex = 1)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20fifth%20Code%20Chunk-1.png)<!-- -->

``` r
stripplot(student_performance_dataset_mice,#to confirm whether the existing data is randomized enough
          MAP ~  motivator | .imp,
          pch = 20, cex = 1)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20fifth%20Code%20Chunk-2.png)<!-- -->

``` r
#Disadvantage of data imputation-- The data should be randomized  but has to be realistic or accurate #
## Impute the missing data ----
# We then create imputed data for the final dataset using the mice::complete()
# function in the mice package to fill in the missing data.
student_performance_dataset_imputed <- mice::complete(student_performance_dataset_mice, 1)
```

# STEP 5. Confirm the "missingness" in the Imputed Dataset

``` r
# A textual confirmation that the dataset has no more missing values in any
# feature:
miss_var_summary(student_performance_dataset_imputed)
```

```         
## # A tibble: 16 × 3
##    variable                n_miss pct_miss
##    <chr>                    <int>    <dbl>
##  1 class_group                  0        0
##  2 gender                       0        0
##  3 YOB                          0        0
##  4 regret_choosing_bi           0        0
##  5 drop_bi_now                  0        0
##  6 motivator                    0        0
##  7 friendships                  0        0
##  8 spiritual_wellnes            0        0
##  9 financial_wellness           0        0
## 10 health                       0        0
## 11 alcohol_or_narcotics         0        0
## 12 mentor                       0        0
## 13 mentor_meetings              0        0
## 14 sports_extra_curricular      0        0
## 15 romantic_relationships       0        0
## 16 MAP                          0        0
```

``` r
# A visual confirmation that the dataset has no more missing values in any
# feature:
Amelia::missmap(student_performance_dataset_imputed)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20sixth%20Code%20Chunk-1.png)<!-- -->

``` r
#########################
# Are there missing values in the dataset?
any_na(student_performance_dataset_imputed)
```

```         
## [1] FALSE
```

``` r
# How many?
n_miss(student_performance_dataset_imputed)
```

```         
## [1] 0
```

``` r
# What is the percentage of missing data in the entire dataset?
prop_miss(student_performance_dataset_imputed)
```

```         
## [1] 0
```

``` r
# How many missing values does each variable have?
student_performance_dataset_imputed%>% is.na() %>% colSums()
```

```         
##             class_group                  gender                     YOB 
##                       0                       0                       0 
##      regret_choosing_bi             drop_bi_now               motivator 
##                       0                       0                       0 
##             friendships       spiritual_wellnes      financial_wellness 
##                       0                       0                       0 
##                  health    alcohol_or_narcotics                  mentor 
##                       0                       0                       0 
##         mentor_meetings sports_extra_curricular  romantic_relationships 
##                       0                       0                       0 
##                     MAP 
##                       0
```

``` r
# What is the number and percentage of missing values grouped by
# each variable?
miss_var_summary(student_performance_dataset_imputed)
```

```         
## # A tibble: 16 × 3
##    variable                n_miss pct_miss
##    <chr>                    <int>    <dbl>
##  1 class_group                  0        0
##  2 gender                       0        0
##  3 YOB                          0        0
##  4 regret_choosing_bi           0        0
##  5 drop_bi_now                  0        0
##  6 motivator                    0        0
##  7 friendships                  0        0
##  8 spiritual_wellnes            0        0
##  9 financial_wellness           0        0
## 10 health                       0        0
## 11 alcohol_or_narcotics         0        0
## 12 mentor                       0        0
## 13 mentor_meetings              0        0
## 14 sports_extra_curricular      0        0
## 15 romantic_relationships       0        0
## 16 MAP                          0        0
```

``` r
# What is the number and percentage of missing values grouped by
# each observation?
miss_case_summary(student_performance_dataset_imputed)
```

```         
## # A tibble: 80 × 3
##     case n_miss pct_miss
##    <int>  <int>    <dbl>
##  1     1      0        0
##  2     2      0        0
##  3     3      0        0
##  4     4      0        0
##  5     5      0        0
##  6     6      0        0
##  7     7      0        0
##  8     8      0        0
##  9     9      0        0
## 10    10      0        0
## # ℹ 70 more rows
```

``` r
# Which variables contain the most missing values?
gg_miss_var(student_performance_dataset_imputed)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20sixth%20Code%20Chunk-2.png)<!-- -->

``` r
# We require the "ggplot2" package to create more appealing visualizations

# Where are missing values located (the shaded regions in the plot)?
vis_miss(student_performance_dataset_imputed) + theme(axis.text.x = element_text(angle = 80))
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20sixth%20Code%20Chunk-3.png)<!-- -->

``` r
# Which combinations of variables are missing together?

# Note: The following command should give you an error stating that at least 2
# variables should have missing data for the plot to be created.
#gg_miss_upset(student_performance_dataset_imputed)

# Create a heatmap of "missingness" broken down by "gender"
# First, confirm that the "gender" variable is a categorical variable
is.factor(student_performance_dataset_imputed$gender)
```

```         
## [1] TRUE
```

``` r
# Second, create the visualization
gg_miss_fct(student_performance_dataset_imputed, fct = gender)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20sixth%20Code%20Chunk-4.png)<!-- -->

``` r
# We can also create a heatmap of "missingness" broken down by "class_group"
# First, confirm that the "class_group" variable is a categorical variable
is.factor(student_performance_dataset_imputed$class_group)
```

```         
## [1] TRUE
```

``` r
# Second, create the visualization
gg_miss_fct(student_performance_dataset_imputed, fct = class_group)
```

![](Lab3-Submission-DataImputation_files/figure-gfm/Your%20sixth%20Code%20Chunk-5.png)<!-- -->
