---
layout: default
title: R examples
nav_order: 10
---

# Practical example of data analysis in R

## Reading in .sav files into R

Most of the time, survey data is provided as .sav files. Because .sav is not a standard format that R can read, an <strong>additional library is required to read in the .sav file </strong>. Here, we use the package `haven`.

## Datasets for today's workshop

We are working with two years (<a href="https://abacus.library.ubc.ca/file.xhtml?persistentId=hdl:11272.1/AB2/UYC0Z8/XVITQW&version=1.0" target="_blank">2020</a> and <a href="https://abacus.library.ubc.ca/file.xhtml?persistentId=hdl:11272.1/AB2/PWWFK3/4K96XZ&version=1.0" target="_blank">2022</a>) of the Canadian Tobacco, Alcohol and Drugs Survey available on abacus.

We will start with the 2022 dataset, then combine it with the 2020 dataset later to practice working with two datasets.

Note, you are encouraged to code along in R with this section, but it is not required. If you are unfamiliar with R and RStudio, you may want to go over the Beginner R <a href="https://ubc-library-rc.github.io/Beginner_R_Part1/" target="_blank">part 1</a> and <a href="https://ubc-library-rc.github.io/Beginner_R_Part2/" target="_blank">part 2</a> workshops.

We also cover material that we present more in depth in the <a href="https://ubc-library-rc.github.io/ggplot2_intro_workshop/" target="_blank">ggplot2</a>, <a href="https://ubc-library-rc.github.io/relational-data-r/" target="_blank">joining tables</a>, and <a href="https://ubc-library-rc.github.io/intro_to_tidyverse/" target="_blank">tidyverse</a> workshops. 

## Set up the analysis

First, we will install (if needed) and load the packages we need for the workshop.
Then we will download the files from abacus, unzip them, and read them into R.

``` r
## Install packages (only needs to be run once on your computer, then never again).
install.packages("tidyverse")
install.packages("haven")

## load packages
library(tidyverse) 
library(haven)

## download and read in 2020 survey data
download.file("https://abacus.library.ubc.ca/api/access/datafile/:persistentId?persistentId=hdl:11272.1/AB2/UYC0Z8/XVITQW","CTNS_2020.zip", mode="wb")
unzip("CTNS_2020.zip")
tab2020 = read_sav("ctns_2020_pumf_eng.sav")

## download and read in 2022 data
download.file("https://abacus.library.ubc.ca/api/access/datafile/:persistentId?persistentId=hdl:11272.1/AB2/PWWFK3/4K96XZ","CTNS_2022.zip", mode="wb")
unzip("CTNS_2022.zip")
tab2022 = read_sav("ctns_2022_pumf.sav")
```

## Reducing clutter

Use the select function to only keep the columns of interest. This is not necessary, but it increases the readability of the dataframe for you and also can also save computing power and disk storage space down the line.

``` r
tab2022 = select(tab2022, "AGEGROUP", "GENDER", "TBC_05A", "FIRSTTRR", "DV_VC30B", "WTPP")
```

## Participant demographic data (2022)

Lets make an initial graph to look at the participant age breakdown of the survey.

``` r
ggplot(tab2022, aes(x=as.factor(AGEGROUP), group=GENDER, fill=as.character(GENDER)))+
  geom_bar(position = "dodge")+
  scale_fill_manual(values=c("#27E57A", "#7A27E5", "#E57A27"))+
  labs(x="Age Group", y="Number of Responses", fill="Gender")
```

<div style="margin-left: 5%; margin-top: 20px; margin-bottom: 60px">
<img src="images/age_breakdown.png" alt="unweigthed bargraph" width="50%"/>
</div>

<div style="margin-left: 5%; margin-top: 20px; margin-bottom: 60px">
<img src="images/AGEGROUP.png" alt="outline" width="60%"/>
</div>

Is this the actual age breakdown of the Canadian population?

<a href="https://publications.gc.ca/Collection/Statcan/96F0030X/96F0030XIE2001002.pdf" target="_blank">No!</a> Often, survey data is analyzed by looking at statistical weight.

The WTPP variable is the participant survey weight variable. We will use this variable to correctly weigh the survey responses to answer our questions for the rest of the workshop.

Also, notice how one of the gender responses is NA? R is treating these non-responses are missing data. This is not necessarily a problem, but it can be when doing data manipulation and analysis.

For the sake of showing how to change NA values in datasets to a specific value, lets change the musing values (NA) in the gender column to 3. *Note: blank cells and non-italicized NA in cells are not counted as missing values, they are counted as characters, so they are handled differently by R and what we are about to do will not work in these two cases.*

``` r
tab2022$GENDER[is.na(tab2022$GENDER)]<-3
```

Now, run the plot code from before. What difference do you notice?

``` r
ggplot(tab2022, aes(x=as.factor(AGEGROUP), group=GENDER, fill=as.character(GENDER)))+
  geom_bar(position = "dodge")+
  scale_fill_manual(values=c("#27E57A", "#7A27E5", "#E57A27"))+
  labs(x="Age Group", y="Number of Responses", fill="Gender")
```

## Example questions

### Of the people who currently smoke cigarettes, what percent of them first tried cigarettes?

``` r
## only keep current smokers
cig = subset(tab2022, tab2022$TBC_05A==1)

## answer our question
cig.sum <- cig %>%
  group_by(FIRSTTRR) %>% # group by the first smoking device tried
  summarize(Population = sum(WTPP)) %>% #get weighted totals
  mutate(Percentage = 100*Population/sum(Population)) #calculate percentages
  
## plot to visualize the data in the table below  
ggplot(cig.sum, aes(x=as.factor(FIRSTTRR), y=Percentage)) +
  geom_col() +
  coord_flip()  
```

<div style="margin-left: 5%; margin-top: 20px; margin-bottom: 60px">
<img src="images/pooled_first_try.png" alt="outline" width="40%"/>
</div>

There is a problem here. <a href="https://en.wikipedia.org/wiki/Electronic_cigarette" target="_blank">E-cigarettes only really entered the market in 2003</a>, so older age group did not have e-cigarettes available as a "first try option". Let's break down the data by age group.

``` r
## make a summary table by the first thing participants tried smoking by age group

cig.age.sum <- cig %>%
  group_by(FIRSTTRR, AGEGROUP) %>% # group by the first smoking device tried
  summarize(Population = sum(WTPP), .groups = 'drop') %>% #get weighted totals
  mutate(Percentage = 100*Population/sum(Population)) #calculate percentages
  
ggplot(cig.age.sum, aes(x=as.factor(FIRSTTRR), y=Percentage)) +
  geom_col() +
  coord_flip()+
  facet_grid(AGEGROUP~.)
```

<div style="margin-left: 5%; margin-top: 20px; margin-bottom: 60px">
<img src="images/try_by_age.png" alt="outline" width="90%"/>
</div>
<em>Note the different x-axes between these two plots.</em>

## Working with two years of data.

Often, your data will be in multiple separate files that need to be combined prior to analysis. The UBC libraries offers <a href="https://ubc-library-rc.github.io/relational-data-r/" target="_blank">workshops</a> on how to join datasets in R.

Working with many years is challenging. The variable names and data format can change year to year, so it is important to read the user guide for each year and to re-format your data accordingly.

Here, we will demonstrate combining two datasets (2020 and 2022) and just work with the columns that merge properly without additional formatting.

``` r
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

``` r
## Plot A
ggplot(tab, aes(x=as.factor(AGEGROUP), fill=as.factor(survey_year)))+
  geom_bar(position = "dodge")+
  labs(x="Age Group", y="Number of Responses", fill="Survey year")+
  facet_wrap(.~GENDER)+
  scale_fill_manual(values=c("#27E57A", "#7A27E5"))

## Plot B
ggplot(tab, aes(x=as.factor(AGEGROUP), fill=as.factor(survey_year)))+
  geom_bar(position = "dodge")+
  labs(x="Age Group", y="Number of Responses", fill="Survey year")+
  facet_wrap(.~GENDER, scales="free_y") ##"free" lets both axes vary. We use "free_y" to only let the y-axis vary
  scale_fill_manual(values=c("#27E57A", "#7A27E5"))
```

![](images/both_years_age.png)

What do you notice about this plot?
-   Statistical weights: Does this plot represent the actual age and gender distribution of Canada?
-   Gender breakdown: Which gender identities are likely underrepresented in the data. Is this more of a problem in some age groups than others?
-   Overall participant numbers between years
-   What breakdown would you like to see before analyzing the data? Maybe, socioeconomic status, race, or marital status?

Comparing datasets at this broad level allows us to set up our analyses correctly. For help on how to account or different sample sizes and other statistical questions, see the <a href="https://www.stat.ubc.ca/how-can-you-get-help-your-data" target="_blank">UBC statistics department.</a>
