---
layout: default
title: R examples
nav_order: 10
---

#Datasets for today's workshop

We are working with two years (<a href="https://hdl.handle.net/11272.1/AB2/UYC0Z8" target="_blank">2020</a> and <a href="https://hdl.handle.net/11272.1/AB2/PWWFK3" target="_blank">2022</a>) of the Canadian Tobacco, Alcohol and Drugs Survey available on abacus.

We will start with the 2022 dataset, then combine it with the 2020 dataset later to practice working with two datasets.

## Reading in plain text data into R

``` r
## load packages
library(tidyverse)
library(plyr)

# 2020 survey data
tab2020 = read.csv("ctns.csv")
# 2022 survey data
tab2022 = read.csv("CTNS2022_P.csv")
```

## Participant demographic data

Lets make an initial graph to look at the participant age breakdown of the survey.

``` r
ggplot(tab2022, aes(x=as.factor(AGEGROUP), group=GENDER, fill=as.character(GENDER)))+
  geom_bar(position = "dodge")+
  scale_fill_manual(values=c("#27E57A", "#7A27E5", "#E57A27"))+
  labs(x="Age Group", y="Number of Responses", fill="Gender")
```

Is this the age breakdown of the Canadian population?

<a href="https://publications.gc.ca/Collection/Statcan/96F0030X/96F0030XIE2001002.pdf" target="_blank">No!</a> Often, survey data is analyzed by looking at statistical weight.

Another example of statistical weighting, the survey data we have loaded in has statistical weight by province included as a column. Depending on your study question, you may want to weight participant responses.

## Example questions

### Of the people who currently smoke cigarettes, is that the first smoking product they tried?

### What percentage of people aged 20 to 24 who vape, vape cannabis?

## Working with two years of data.

Often, your data will be in multiple separate files that need to be combined prior to analysis. The UBC libraries offers <a href="https://ubc-library-rc.github.io/relational-data-r/" target="_blank">workshops</a> on how to join datasets in R.

Working with many years is challenging. The variable names and data format can change year to year, so it is important to read the user guide for each year and to re-format your data accordingly.

Here, we will demonstrate combining two datasets (2020 and 2022) and just work with the columns that merge properly without additional formatting.

```r
## add survey year indicator
tab2022$survey_year = 2022
tab2020$survey_year = 2020
## join tables (datasets)
tab = full_join(tab2022, tab2020)

## check no rows were missed or added
## rows in 2020
print(nrow(tab2020))
## rows in 2022
print(nrow(tab2022))
## total of new rows
nobservations = nrow(tab2020)+nrow(tab2022)
print(nobservations)

```

### How does the age group and gender breakdown compare between the two surveys?

```r
ggplot(tab, aes(x=as.factor(AGEGROUP), group=GENDER, fill=as.character(GENDER)))+
geom_bar(position = "dodge")+
scale_fill_manual(values=c("#27E57A", "#7A27E5", "#E57A27"))+
labs(x="Age Group", y="Number of Responses", fill="Gender")+
facet_grid(.~survey_year)

```

What do you notice about this plot?

-   Statistical weights

-   Overall participant numbers

-   Gender breakdown

Comparing datasets at this broad level allows us to set up our analyses correctly. For help on how to account or different sample sizes and other statistical questions, see the <a href="https://www.stat.ubc.ca/how-can-you-get-help-your-data" target="_blank">UBC statistics department.</a>

## Reading in sav files into R

``` r
## load packages
library(haven)

sav2022 = read_sav("ctns_2020_pumf_bsw_eng.sav")
```

### 
