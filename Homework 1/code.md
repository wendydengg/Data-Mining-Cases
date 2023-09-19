---
title: " Modern Data Mining, HW 1"
author:
- Wendy Deng
- Ruolan Li
- Kira Nightingale
date: 'Due: 11:59PM,  Jan. 29th, 2023'
output:
  html_document:
    code_folding: show
    highlight: haddock
    number_sections: yes
    theme: lumen
    toc: yes
    toc_depth: 4
    toc_float: yes
  pdf_document:
    number_sections: yes
    toc: yes
    toc_depth: '4'
  word_document:
    toc: yes
    toc_depth: '4'
urlcolor: blue
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, fig.width=8, fig.height=4, warning = FALSE)
options(scipen = 0, digits = 3)  # controls base R output
if(!require('pacman')) {
  install.packages('pacman')
}
pacman::p_load(ISLR, readxl, tidyverse, magrittr, dplyr, ggplot2, gridExtra, ggrepel, plotly, skimr, tidytext, pander, ggpubr) 
```


\pagebreak

# Overview

This is a fast-paced course that covers a lot of material. There will be a large amount of references. You may need to do your own research to fill in the gaps in between lectures and homework/projects. It is impossible to learn data science without getting your hands dirty. Please budget your time evenly. Last-minute work ethic will not work for this course. 

Homework in this course is different from your usual homework assignment as a typical student. Most of the time, they are built over real case studies.  While you will be applying methods covered in lectures, you will also find that extra teaching materials appear here.  The focus will be always on the goals of the study, the usefulness of the data gathered, and the limitations in any conclusions you may draw. Always try to challenge your data analysis in a critical way. Frequently, there are no unique solutions. 

Case studies in each homework can be listed as your data science projects (e.g. on your CV) where you see fit. 



## Objectives 

- Get familiar with `R-studio` and `RMarkdown`
- Hands-on R 
- Learn data science essentials 
    - gather data
    - clean data
    - summarize data 
    - display data
    - conclusion
- Packages
    - `dplyr`
    - `ggplot`

##  Instructions

- **Homework assignments can be done in a group consisting of up to three members**. Please find your group members as soon as possible and register your group on our Canvas site.

- **All work submitted should be completed in the R Markdown format.** You can find a cheat sheet for R Markdown [here](https://www.rstudio.com/wp-content/uploads/2015/02/rmarkdown-cheatsheet.pdf) For those who have never used it before, we urge you to start this homework as soon as possible. 

- **Submit the following files, one submission for each group:**  (1) Rmd file, (2) a compiled  HTML or pdf version, and (3) all necessary data files if different from our source data. You may directly edit this .rmd file to add your answers. If you intend to work on the problems separately within your group, compile your answers into one Rmd file before submitting. We encourage that you at least attempt each problem by yourself before working with your teammates. Additionally, ensure that you can 'knit' or compile your Rmd file. It is also likely that you need to configure Rstudio to properly convert files to PDF. [**These instructions**](http://kbroman.org/knitr_knutshell/pages/latex.html#converting-knitrlatex-to-pdf) might be helpful.

- In general, be as concise as possible while giving a fully complete answer to each question. All necessary datasets are available in this homework folder on Canvas. Make sure to document your code with comments (written on separate lines in a code chunk using a hashtag `#` before the comment) so the teaching fellows can follow along. R Markdown is particularly useful because it follows a 'stream of consciousness' approach: as you write code in a code chunk, make sure to explain what you are doing outside of the chunk. 

- A few good or solicited submissions will be used as sample solutions. When those are released, make sure to compare your answers and understand the solutions.


## Review materials

- Study Basic R Tutorial
- Study Advanced R Tutorial (to include `dplyr` and `ggplot`)
- Study lecture 1: Data Acquisition and EDA


# Case study 1: Audience Size

How successful is the Wharton Talk Show [Business Radio Powered by the Wharton School](https://businessradio.wharton.upenn.edu/)  


**Background:** Have you ever listened to [SiriusXM](https://www.siriusxm.com/)? Do you know there is a **Talk Show** run by Wharton professors in Sirius Radio?  Wharton launched a talk show called [Business Radio Powered by the Wharton School](https://businessradio.wharton.upenn.edu/) through the Sirius Radio station in January of 2014. Within a short period of time the general reaction seemed to be overwhelmingly positive. To find out the audience size for the show, we designed a survey and collected a data set via MTURK in May of 2014. Our goal was to **estimate the audience size**. There were 51.6 million Sirius Radio listeners then. One approach is to estimate the proportion of the Wharton listeners to that of the Sirius listeners, $p$, so that we will come up with an audience size estimate of approximately 51.6 million times $p$. 

To do so, we launched a survey via Amazon Mechanical Turk ([MTurk](https://www.mturk.com/)) on May 24, 2014 at an offered price of \$0.10 for each answered survey.  We set it to be run for 6 days with a target maximum sample size of 2000 as our goal. Most of the observations came in within the first two days. The main questions of interest are "Have you ever listened to Sirius Radio" and "Have you ever listened to Sirius Business Radio by Wharton?". A few demographic features used as control variables were also collected; these include Gender, Age and Household Income.  

We requested that only people in United States answer the questions. Each person can only fill in the questionnaire once to avoid duplicates. Aside from these restrictions, we opened the survey to everyone in MTurk with a hope that the sample would be more randomly chosen. 

The raw data is stored as `Survey_results_final.csv` on Canvas.

## Data preparation

1.  We need to clean and select only the variables of interest.

Select only the variables Age, Gender, Education Level, Household Income in 2013, Sirius Listener?, Wharton Listener? and Time used to finish the survey.

```{r}
survey_results <- read.csv("data/Survey_results_final.csv")
```

Change the variable names to be "age", "gender", "education", "income", "sirius", "wharton", "worktime".

```{r, cleaning}
results <- survey_results %>% 
  # select only the variables Age, Gender, Education Level, Household Income in 2013, Sirius Listener?, Wharton Listener? and Time used to finish the survey
  select(Answer.Age, Answer.Gender, Answer.Education, Answer.HouseHoldIncome, Answer.Sirius.Radio, Answer.Wharton.Radio, WorkTimeInSeconds) %>% 
  # change the variable names to be "age", "gender", "education", "income", "sirius", "wharton", "worktime"
  rename(age = Answer.Age, gender = Answer.Gender, education = Answer.Education, income = Answer.HouseHoldIncome, sirius = Answer.Sirius.Radio, wharton = Answer.Wharton.Radio, worktime = WorkTimeInSeconds) %>% 
  # omit any row with NA because they will introduce noise into our data
  na.omit()

names(results)
str(results)

# get datatype of each column so we can convert them back to original datatype later on
print(class(results$age))
print(class(results$gender))
print(class(results$education))
print(class(results$income))
print(class(results$sirius))
print(class(results$wharton))
print(class(results$worktime))
```

```{r, factoring}
# we want to see what kinds of results are available for each variable
results$age <- as.factor(results$age)
results$gender <- as.factor(results$gender)
results$education <- as.factor(results$education)
results$income <- as.factor(results$income)
results$sirius <- as.factor(results$sirius)
results$wharton <- as.factor(results$wharton)
results$worktime <- as.factor(results$worktime)

summary(results)
```

2. Handle missing/wrongly filled values of the selected variables

As in real world data with user input, the data is incomplete, with missing values, and has incorrect responses. There is no general rule for dealing with these problems beyond “use common sense.” In whatever case, explain what the problems were and how you addressed them. Be sure to explain your rationale for your chosen methods of handling issues with the data. Do not use Excel for this, however tempting it might be.

Tip: Reflect on the reasons for which data could be wrong or missing. How would you address each case? For this homework, if you are trying to predict missing values with regression, you are definitely overthinking. Keep it simple.

**We should remove rows with empty entries due to possible nonresponse bias from incomplete data, but we will take a look at those rows first and decide how trustworthy they are. As we can see, most rows with only one or two answers missing seem to be credible, but others with multiple answers missing seem like the person did not put effort into answering the survey. Since our sample size is sufficiently large compared to the 17 rows with incomplete data, we can omit these rows from our analysis. Moreover, we should removes rows with incorrect/non-sensical entries, and we can display those using summary() on each of the factors. Examples of incorrect data in this case include "select one" under education, and those who selected "Yes" to Wharton but "No" to Sirius because that is not possible when the Wharton radio is one of the channels of Sirius.**

```{r, remove incomplete and incorrect data}
results %>%
  # view rows with incomplete data
  filter(age == "" | gender == "" | education == "" | income == "" | sirius == "" | wharton == "" | worktime == "")

results <- results %>%
  # keep only the rows that are not empty
  filter(age != "", gender != "", education != "", income != "", sirius != "", wharton != "", worktime != "") %>% 
  
  # remove rows with incorrect data entry
  filter(education != "select one") %>% # remove "select one" under education
  filter(!(wharton == "Yes" & sirius == "No")) %>% # remove inconsistent answers
  filter(age != "female") %>% # remove incorrect entry
  filter(age != "223") %>% # remove incorrect entry
  filter(age != "4") %>% # remove incorrect entry
  mutate(age = recode(age, "Eighteen (18)" = "18")) %>% # change "Eighteen (18)" to "18" under age
  mutate(age = recode(age, "27`" = "27")) # change "27`" to "27" under age

results <- droplevels(results) # drop levels in the factors after removing data
summary(results)

summary(results$age) # to see the (Other) levels under age and change the incorrect data entries accordingly
```

3. Brief summary 

Write a brief report to summarize all the variables collected. Include both summary statistics (including sample size) and graphical displays such as histograms or bar charts where appropriate. Comment on what you have found from this sample. (For example - it's very interesting to think about why would one work for a job that pays only 10cents/each survey? Who are those survey workers? The answer may be interesting even if it may not directly relate to our goal.)

**After cleaning the data, we have 1725 individuals in our sample. There are 729 female and 996 male (ratio of 0.42 : 0.58), and age ranges from 18 to 76. As seen from the histograms of age, education, and income: the distribution of age is right-skewed with a mean of 30.3 and a median of 28, 88.9% of the sample graduated from college with a Bachelor's, Associate's, or Graduate degree, and income distribution tells us that people who earn above $150,000 participated the least - this makes intuitive sense as the survey paid only 10 cents and it is unlikely that someone making a high income would be motivated to complete it. The scatterplot of age, education, and whether if one listens to either channel shows that one's age and education level do not affect their response to listening to the channel or not.**

```{r summary statistics and graphs}
# convert character to integer for datatype of age and worktime for calculation
results$age = as.integer(as.character(results$age)) 
results$worktime <- as.integer(as.character(results$worktime)) 

skimr::skim(results)
summary(results)

# distribution of age, education, and income in our sample -- histograms
age <- ggplot(results) + 
  geom_histogram(aes(x = age), bins = 7, fill = "darkblue") +
  labs( title = "Age of Respondents", x = "Age" , y = "Frequency")

education <- results %>%
      mutate(education = fct_relevel(education, "Less than 12 years; no high school diploma", "High school graduate (or equivalent)", "Some college, no diploma; or Associate’s degree", "Bachelor’s degree or other 4-year degree", "Graduate or professional degree", "Other")) %>% 
  ggplot() + 
  geom_histogram(aes(x = education), stat="count", fill = "blue") +
  labs(title = "Education Level of Respondents", x = "Education" , y = "Frequency") +
  theme(axis.text.x = element_text(angle = -60)) 

college_prop = (611+745+177)/1725

income <- results %>%
  mutate(income = fct_relevel(income, "Less than $15,000", "$15,000 - $30,000", "$30,000 - $50,000", "$50,000 - $75,000", "$75,000 - $150,000", "Above $150,000")) %>%
  ggplot() + 
  geom_histogram(aes(x = income), stat="count", fill = "lightblue") +
  labs( title = "Income Level of Respondents", x = "Income" , y = "Frequency") +
  theme(axis.text.x = element_text(angle = -60)) 

grid.arrange(age, education, income, ncol = 3) 

# scatterplot on sirius and wharton
sirius <- results %>%
  ggplot(aes(x = sirius, y = age)) + 
  # geometric options with aes mapping: 
  # color, size, alpha as a function of a variable 
  geom_point(aes(color = education), size = 3) + 
  labs(title = "Age vs. Listens to Sirius or no", 
       x = "Listens to Sirius?", 
       y = "Age") +
  theme_bw() +
  theme(legend.position = c(0.5, 0.5))

wharton <- results %>%
  ggplot(aes(x = wharton, y = age)) + 
  # geometric options with aes mapping: 
  # color, size, alpha as a function of a variable 
  geom_point(aes(color = education), size = 3) + 
  labs(title = "Age vs. Listens to Wharton or no", 
       x = "Listens to Wharton?", 
       y = "Age") +
  theme_bw() +
  theme(legend.position = c(0.5, 0.5))

grid.arrange(sirius, wharton, ncol = 2) 
```

## Sample properties

The population from which the sample is drawn determines where the results of our analysis can be applied or generalized. We include some basic demographic information for the purpose of identifying sample bias, if any exists. Combine our data and the general population distribution in age, gender and income to try to characterize our sample on hand.

1. Does this sample appear to be a random sample from the general population of the USA?
**Since our study was conducted in 2014, according to US Census Bureau, the median age in US in 2014 was 37.9, which is around 10 years older than the median age in our sample. According to "Population Distribution by Age" from KFF, the third quartile in the US is within the range of age 55-64, while the third quartile in our sample is age 34. Overall, most of the people's age in our sample are concentrated within a smaller, younger age range than the general US population.**

2. Does this sample appear to be a random sample from the MTURK population?
**According to an article published in 2016 by Pew Research Center, the ratio of female to male Turkers is 0.49 : 0.51, which is very close to the ratio in our sample. As for income, 68% of Turkers earn within the range of $10k to $75k, which is also pretty representative of our sample ((358+420+371)/1725 = 67% earn within the range of $15k to $75k in our sample). As for age, 88% of Turkers are within the age range of 18-49. Since the third quartile in our sample is age 34, it is reasonable to say that around 88% of Turkers in our sample also falls within that age range. Moreover, 87% of Turkers have college degree, which is about the same as the college education proportion in our sample as well. In short, this sample does appear to be a random sample from the MTURK population given that a lot of the statistics are very similar.**

Note: You can not provide evidence by simply looking at our data here. For example, you need to find distribution of education in our age group in US to see if the two groups match in distribution. You may need to gather some background information about the MTURK population to have a slight sense if this particular sample seem to a random sample from there... Please do not spend too much time gathering evidence. 

## Final estimate

Give a final estimate of the Wharton audience size in January 2014. Assume that the sample is a random sample of the MTURK population, and that the proportion of Wharton listeners vs. Sirius listeners in the general population is the same as that in the MTURK population. Write a brief executive summary to summarize your findings and how you came to that conclusion.

To be specific, you should include:

1. Goal of the study
2. Method used: data gathering, estimation methods
3. Findings
4. Limitations of the study. 

```{r estimate Wharton audience size}
#One approach is to estimate the proportion of the Wharton listeners to that of the Sirius listeners, $p$, so that we will come up with an audience size estimate of approximately 51.6 million times $p$.

wharton_prop <-
results %>%
  filter(wharton == "Yes") %>%
  nrow() 

sirius_prop <-
results %>%
  filter(sirius == "Yes") %>%
  nrow()

p <- wharton_prop/sirius_prop
percent <- p*100
audience_size = 51600000 * p
```

**The goal of the study is to estimate the Wharton Radio audience size by calculating the proportion of Wharton listeners to Sirius listeners times the total Sirius listener base. Data was collected via a survey on Amazon Mechanical Turk and ran for a length of 6 days. A total of 1764 responses were collected, which was reduced to 1725 after removal of incomplete/inaccurate responses. Given the total number of Sirius users (51.6 million), and the proportion of Sirius listeners to Wharton listeners in our sample (5.01%), it is estimated that there are 2.58 million listeners of Wharton Radio. A summary of the participants' demographic information shows that people who took the survey are likely representative of the MTurk population, but they are not representative of the US population since people who take MTurk surveys are generally younger and more educated than the US population. Thus, the main limitation of this study is that we cannot extrapolate the results of the data to the general US population, but we can say that among the MTurk population, those who listen to Wharton radio is about 5.01% (p) of the population.**

## New task

Now suppose you are asked to design a study to estimate the audience size of Wharton Business Radio Show as of today: You are given a budget of $1000. You need to present your findings in two months. 

Write a proposal for this study which includes:

1. Method proposed to estimate the audience size.
2. What data should be collected and where it should be sourced from.
Please fill in the google form to list your platform where surveys will be launched and collected [HERE](https://forms.gle/8SmjFQ1tpqr6c4sa8) 

A good proposal will give an accurate estimation with the least amount of money used. 

**Our new study will involve the use of multiple websites, social media platforms, or even hand-delivered mails to reach a wider and more representative audience and reduce selection bias. Such methods include websites like MTurk or SurveyMonkey or social media for the younger, educated population; and other methods include surveys through partnership with mails or newspaper for the older population. According to an article by Fit Small Business, an ad measuring four columns wide by 10 inches high in a standard newspaper would cost $480, which is more than enough space for our study and also fits within our budget. To reach the younger population, we can use multiple websites such as SurveyMonkey and Google Opinion Rewards along with Mturk, and use social media platforms such as Facebook. To increase the incentive to response on social media (since people who use social media are usually not looking to answer some surveys), we will offer raffle tickets into bigger prizes whenever they respond, which we can use around $200 for such prize.**

**As for the type of data to collect, we will use the same variables in the original survey with the addition of region and career fields to determine how representative of the US population will our sample be. Assuming we will have n participants after collecting and cleaning the data, we want to find the audience size X of Wharton. We can think of X as a random variable denoting the audience size of Wharton, and it follows a binomial distribution with probability of success p that the participants listens to Wharton. Then, p can be calculated using the original method, which is the proportion of Wharton listeners to that of Sirius listeners. We can now find the estimated audience size of Wharton, which is expectation of X from the population size of the US.**

# Case study 2: Women in Science


Are women underrepresented in science in general? How does gender relate to the type of educational degree pursued? Does the number of higher degrees increase over the years? In an attempt to answer these questions, we assembled a data set (`WomenData_06_16.xlsx`) from [NSF](https://ncses.nsf.gov/pubs/nsf19304/digest/field-of-degree-women) about various degrees granted in the U.S. from 2006 to 2016. It contains the following variables: Field (Non-science-engineering (`Non-S&E`) and sciences (`Computer sciences`, `Mathematics and statistics`, etc.)), Degree (`BS`, `MS`, `PhD`), Sex (`M`, `F`), Number of degrees granted, and Year.

Our goal is to answer the above questions only through EDA (Exploratory Data Analyses) without formal testing. We have provided sample R-codes in the appendix to help you if needed. 


## Data preparation  

1. Understand and clean the data

Notice the data came in as an Excel file. We need to use the package `readxl` and the function `read_excel()` to read the data `WomenData_06_16.xlsx` into R. 

a). Read the data into R.
```{r load data, echo = TRUE}
women <- read_xlsx("C:/Users/nighki01/Box Sync/Course Materials/2023 Spring_Data Mining/Homework 1/data/WomenData_06_16.xlsx")
```

b). Clean the names of each variables. (Change variable names to  `Field`,`Degree`, `Sex`, `Year` and `Number` )
```{r variable names, echo = TRUE}
#Degree, sex, and year are already named as desired. We therefore only rename the "field and sex" and "degrees awarded" variables.
women <- women %>% rename("Field" = "Field and sex", "Number" = "Degrees Awarded")
```

c). Set the variable natures properly. 
```{r variable types, echo = TRUE, results='markup', warning=FALSE}
#For this dataset, we would like field to be character, degree and sex to be factor, and year and number to be numeric.
str(women)

women$Degree <- as.factor(women$Degree)
women$Sex <- as.factor(women$Sex)

str(women)
  
```


d). Any missing values?
**There are no missing values in this dataset**
```{r missing, echo = TRUE, results = 'markup'}
summary(women)
colSums(is.na(women))
```


2. Write a summary describing the data set provided here. 
```{r summary, echo = TRUE, results='markup'}
summary(women)
table(women$Field)
table(women$Year)

```

a). How many fields are there in this data?

b). What are the degree types? 

c). How many year's statistics are being reported here? 

**This is a dataset that lists the number of degrees conferred by year, sex, field, and degree type. There are five variables and 660 observations. The data includes ten fields of study, three degree types (BS, MS, and PhD), and encompasses 11 years (2006-2016, inclusive).**


## BS degrees in 2015

Is there evidence that more males are in science-related fields vs `Non-S&E`? Provide summary statistics and a plot which shows the number of people by gender and by field. Write a brief summary to describe your findings.

**Based on our initial assessment, ignoring degree type and specific field/major, we can see that there are more females than males who obtain non-S&E degrees than males, while the two sexes are fairly even when it comes to S&E degrees. When looking at the specific field, it appears that there are more females than males who obtain degrees in biology and non- science/engineering fields as well as in psychology and social sciences. Greater numbers of males obtain degrees in computer sciences, engineering, math/statistics, and physical sciences. There seems to be somewhat equal numbers of males and females who receive degrees in agriculture and earth science.**

```{r 2015 field and sex, echo = TRUE, message = FALSE, results = 'markup'}
#Grouping by field and sex, creating summary statistic that combines number of all degrees, subsetting to include only data from 2015. Creating separate subsetted dataset which combines all S&E fields.
subset_2015 <- women %>% 
  group_by(Field, Sex) %>% 
  mutate(all_degs = sum(Number)/10000, SE = ifelse(Field != "Non-S&E", "S&E", "Non-S&E")) %>%
  filter(Year == 2015)

subset_2015_SE <- subset_2015 %>% 
  group_by(SE, Sex) %>% 
  summarize(SE_number = sum(all_degs), SE)

#Summary statistics - distinct field, SE versus non-SE
subset_2015 %>% group_by(Field, Sex) %>% summarize(unique(all_degs))
subset_2015_SE %>% group_by(SE, Sex) %>% summarise(unique(SE_number))

#Plots
plot_2015_SE <- ggplot(subset_2015_SE, aes(x=SE, y=SE_number, fill=Sex)) + 
  geom_bar(stat="identity", position="dodge") +
  scale_x_discrete(labels = function(x) str_wrap(x, width = 10)) +
  labs(y = "Number of Degrees (10,000)", title = "Number of Degrees Conferred for S&E Fields versus non-S&E Fields, by Sex") +
  theme_light()
plot_2015_SE

plot_2015 <- ggplot(subset_2015, aes(x=Field, y=all_degs, fill=Sex)) + 
  geom_bar(stat="identity", position="dodge") +
  scale_x_discrete(labels = function(x) str_wrap(x, width = 10)) +
  labs(y = "Number of Degrees (10,000)", title = "Number of Degrees Conferred by Degree Type, Field, and Sex") +
  theme_light()
plot_2015
```


## EDA bringing type of degree, field and gender in 2015

Describe the number of people by type of degree, field, and gender. Do you see any evidence of gender effects over different types of degrees? Again, provide graphs to summarize your findings.

**Trends across degree types are easiest to observe when we consider science & engineering degrees versus non-science & engineering degrees. Across all degree types, there are more females than males who obtain degrees in non-science & engineering fields. However, whereas at the BS level the number of males and females obtaining science & engineering degrees is similar, slightly more males obtain MS degrees and even more males obtain PhD degrees compared to females.**

**As in our first assessment, a larger number of females as compared to males receive degrees in non- science/engineering fields, regardless of degree type. Notably, whereas we can now see slightly less of a difference in the number of males and females who received Bachelor's degrees in the "hard sciences", the difference between males and females is much more apparent when considering doctoral degrees in the "hard sciences". As an illustration, there are approximately equal numbers of males and females who receive BS and MS degrees in Physical Sciences, but almost double the number of males receive a PhD in that field compared to females.**


```{r 2015 field sex and degree, echo = TRUE, message = FALSE, results = 'markup'}
#Grouping by field and sex, creating summary statistic that combines number of all degrees, subsetting to include only data from 2015. Creating separate subsetted dataset which combines all S&E fields.
subset_2015_degs <- women %>% 
  group_by(Field, Sex) %>% 
  mutate(degree_scaled = Number/10000, SE = ifelse(Field != "Non-S&E", "S&E", "Non-S&E")) %>%
  filter(Year == 2015) 

subset_2015_degs_SE <- subset_2015_degs %>% 
  group_by(SE, Sex, Degree) %>% 
  summarize(SE_number = sum(degree_scaled), SE, Degree)

#Summary statistics
subset_2015_degs %>% group_by(Field, Sex) %>% summarize(Degree, degree_scaled)
subset_2015_degs_SE %>% group_by(SE, Sex, Degree) %>% summarize(unique(SE_number))

#Plot
plot_2015_degs_SE <- ggplot(subset_2015_degs_SE, aes(x=SE, y=SE_number, fill=Sex)) + 
  geom_bar(stat="identity", position="dodge") +
  scale_x_discrete(labels = function(x) str_wrap(x, width = 10)) + 
  facet_grid(Degree~., scales = "free_y") +
  labs(y = "Number of Degrees (10,000)", title = "Number of S&E versus non-S&E Degrees Conferred by Degree Type and Sex") +
  theme_light()
plot_2015_degs_SE

plot_2015_degs <- ggplot(subset_2015_degs, aes(x=Field, y=degree_scaled, fill=Sex)) + 
  geom_bar(stat="identity", position="dodge") +
  scale_x_discrete(labels = function(x) str_wrap(x, width = 10)) + 
  facet_grid(Degree~., scales = "free_y") +
  labs(y = "Number of Degrees (10,000)", title = "Number of Degrees Conferred by Degree Type, Field, and Sex") +
  theme_light()
plot_2015_degs
```


## EDA bring all variables 

In this last portion of the EDA, we ask you to provide evidence numerically and graphically: Do the number of degrees change by gender, field, and time? 
**Trends over time indicate that ignoring sex, the number of BS and MS degrees conferred each year has increased over time, while the number of PhD degrees conferred has remained relatively stable. The proportion of women in science and engineering fields has decreased over time, and effect that is relatively consistent for BS, MS, and PhD degrees.**

```{r field sex degree and year, echo = TRUE, message = FALSE, results = 'markup'}
#Proportion female in SE/non-SE fields by year
women_prop <- women %>% 
  mutate(SE = ifelse(Field!="Non-S&E" , "S&E", "Non-S&E")) %>%
  group_by(SE, Sex, Year) %>%
  summarise(SE_number = sum(Number)) %>%
  group_by(SE, Year) %>%
  mutate(ratio = SE_number / sum(SE_number)) %>%
  filter(Sex == "Female")

pander(women_prop)

plot_propf <- women_prop %>% ggplot(aes(x = Year, y = ratio, color = SE)) +
  geom_point() + geom_line() +
  ggtitle("Proportion of females obtaining SE/non-SE degrees over time")
plot_propf

#Proportion female in SE/non-SE fields by degree and year
women_prop_deg <- women %>% 
  mutate(SE = ifelse(Field!="Non-S&E" , "S&E", "Non-S&E")) %>%
  group_by(SE, Sex, Degree, Year) %>%
  summarise(SE_number = sum(Number)) %>%
  group_by(SE, Degree, Year) %>%
  mutate(ratio = SE_number / sum(SE_number)) %>%
  filter(Sex == "Female")

pander(women_prop_deg)

plot_propf_deg <- women_prop_deg %>% ggplot(aes(x = Year, y = ratio, color = SE)) +
  geom_point() + geom_line() + 
  facet_grid(Degree~., scales = "free_y") +
  ggtitle("Proportion of females obtaining SE/non-SE degrees over time, by degree type")
plot_propf_deg

#Both sexes, SE/non-SE fields over time by degree and year. Number of degrees scaled to 10,000
women_deg <- women %>% 
  mutate(SE = ifelse(Field!="Non-S&E" , "S&E", "Non-S&E"), num_scaled = Number/10000) %>%
  group_by(SE, Sex, Degree, Year) %>%
  summarise(SE_number = sum(num_scaled)) 

pander(women_deg)

plot_deg <- women_deg %>% ggplot(aes(x = Year, y = SE_number, fill = Sex)) +
  geom_bar(stat = "identity", position = "dodge") +
  facet_grid(SE~Degree, scales = "free_y") +
    labs(y = "Number of Degrees (10,000)", title = "Number of Degrees Conferred by Degree Type, Field, and Sex Over Time")
plot_deg


```


## Women in Data Science

Finally, is there evidence showing that women are underrepresented in data science? Data science is an interdisciplinary field of computer science, math, and statistics. You may include year and/or degree.

**Based on our descriptive analysis, it appears that women are underrepresented in data science, with less that 30% of data science degrees being conferred to women in any given year. We can see that the proportion of MS in data science degrees conferred to women has increased over time, while there was an initial decline in the proportion of BS degrees which is increasing again. The proportion of PhD degrees conferred to women has remained somewhat constant over time with fluctuations from year to year. Overall, there are larger numbers of data science degrees being conferred, most notably for MS degrees.**

```{r women in data science, echo = TRUE, message = FALSE, results = 'markup'}
#Proportion of women in data science (combines computer science and math/statistics)
women_datasci <- women %>% 
  mutate(datasci = ifelse(Field == "Computer sciences", "yes", ifelse(Field == "Mathematics & statistics", "yes", "no"))) %>%
  group_by(datasci, Sex, Year, Degree) %>%
  summarise(datasci_number = sum(Number)) %>%
  group_by(datasci, Year, Degree) %>%
  mutate(ratio = datasci_number / sum(datasci_number)) %>%
  filter(Sex == "Female" & datasci == "yes")

plot_women_datasci <- women_datasci %>%
  ggplot(aes(x = Year, y = ratio, color = Degree)) + 
  geom_point() + geom_line() + 
  labs(title = "Proportion of Women Obtaining Data Science Degrees Over Time")
plot_women_datasci

#Number of degrees in data science by sex
datasci <- women %>%
    mutate(datasci = ifelse(Field == "Computer sciences", "yes", ifelse(Field == "Mathematics & statistics", "yes", "no"))) %>%
  group_by(datasci, Sex, Year, Degree) %>%
  summarise(datasci_number = sum(Number)) %>%
  group_by(datasci, Year, Degree) %>%
  mutate(ratio = datasci_number / sum(datasci_number)) %>%
  filter(datasci == "yes")

plot_datasci <- datasci %>%
  mutate(datasci_scaled = datasci_number/10000) %>%
  ggplot(aes(x = Year, y = datasci_scaled, fill = Sex)) + 
    geom_bar(stat = "identity", position = "dodge") +
  facet_grid(Degree~., scales = "free_y") +
    labs(y = "Number of Degrees (10,000)", title = "Number of Data Science Degrees Conferred by Degree Type and Sex Over Time")
plot_datasci

```


## Final brief report

Summarize your findings focusing on answering the questions regarding if we see consistent patterns that more males pursue science-related fields. Any concerns with the data set? How could we improve on the study?

**In general, we observe evidence that more males than females obtain science and engineering degrees, and this is most notable at the PhD level. Although the total number of science and engineering degrees appear to be increasing over time, the proportion of degrees conferred to females in these fields has remained constant or slightly decreased. One exception to this trend is the field of data science, where we observe increasing numbers of MS degrees being conferred to females over time.**

**One potential concern with the data set is that it is limited to BS, MS, and PhD degrees - it is possible that inclusion of other degree types (such as BA and MA) could impact the results. It could also be of interest to incorporate career information, as receipt of a science/engineering degree does not necessarily mean the individual ended up working in that field (and vice-versa). Finally, it would be helpful to further specify the research question to specific fields, or better define what fields count as "science", as we can see when looking at individual fields that trends are not consistent across fields.**


# Case study 3: Major League Baseball

We would like to explore how payroll affects performance among Major League Baseball teams. The data is prepared in two formats record payroll, winning numbers/percentage by team from 1998 to 2014. 

Here are the datasets:

-`MLPayData_Total.csv`: wide format
-`baseball.csv`: long format

Feel free to use either dataset to address the problems. 

## EDA: Relationship between payroll changes and performance

Payroll may relate to performance among ML Baseball teams. One possible argument is that what affects this year's performance is not this year's payroll, but the amount that payroll increased from last year. Let us look into this through EDA. 

Create increment in payroll

a). To describe the increment of payroll in each year there are several possible approaches. Take 2013 as an example:

    - option 1: diff: payroll_2013 - payroll_2012
    - option 2: log diff: log(payroll_2013) - log(payroll_2012)

Explain why the log difference is more appropriate in this setup.

```{r data input, include=FALSE}
datapay_wide <- read.csv("data/MLPayData_Total.csv")
datapay_long <- read.csv("data/baseball.csv")
```

```{r compare difference and log difference, echo=FALSE}
#plot variable payroll directly
p1 <- datapay_long %>%
  ggplot(aes(x = payroll)) +
  geom_density()

#plot log transformed payroll
p2 <- datapay_long %>%
  ggplot(aes(x = log(payroll))) +
  geom_density()

ggarrange(p1,p2)

```

**1. As the density plot shown, the variable payroll is right skewed. After the log transformation, the density plot is much less skewed. Therefore, it is more appropriate to use log difference.**

**2. Because of the mathematical properties of logarithm: $log(a)-log(b)=log(\frac{a}{b})$, the logarithm transformation converts absolute difference into relative difference.**

b). Create a new variable `diff_log=log(payroll_2013) - log(payroll_2012)`. Hint: use `dplyr::lag()` function.

```{r create diff_log variable, echo=FALSE}
#log transform difference
datapay_long = datapay_long %>%
  group_by(team) %>%
  mutate(diff_log = log(payroll) - log(lag(payroll)))
head(datapay_long)
```

c). Create a long data table including: team, year, diff_log, win_pct

```{r create long table, echo=FALSE}
datapay_long = datapay_long %>%
  select(team, year, diff_log, win_pct)
head(datapay_long)
```



## Exploratory questions

a). Which five teams had highest increase in their payroll between years 2010 and 2014, inclusive?

```{r highest increase in payroll between years 2010 and 2014, echo=FALSE}
payroll <- datapay_long %>%
  group_by(team) %>%
  filter(year %in% c(2010:2014)) %>%
  summarise(diff_sum = sum(diff_log)) %>% #log(2014)-log(2010) = sum of log(2014)-log(2013)+log(2013)-log(2012)...
  arrange(desc(diff_sum))
head(payroll)
```

**Los Angeles Dodgers, Washington Nationals, San Diego Padres, Texas Rangers and San Francisco Giants had highest increase in their payroll between years 2010 and 2014.**

b). Between 2010 and 2014, inclusive, which team(s) "improved" the most? That is, had the biggest percentage gain in wins?

```{r, echo=FALSE}
win <- datapay_wide %>%
  rename("team" = names(.)[1]) %>%
  select(team, paste0("X", c(2014,2010),".pct")) %>%
  group_by(team) %>%
  mutate(pct = X2014.pct - X2010.pct) %>% #difference of percentage
  arrange(desc(pct))
head(win)
```

**Pittsburgh Pirates had the biggest percentage gain in wins between 2010 and 2014.**

## Do log increases in payroll imply better performance? 

Is there evidence to support the hypothesis that higher increases in payroll on the log scale lead to increased performance?

Pick up a few statistics, accompanied with some data visualization, to support your answer. 

```{r test higher increases in payroll lead to increased performance, echo=FALSE}
data_final <- datapay_long %>%
  group_by(team) %>%
  summarise(payroll_mean = mean(diff_log, na.rm = TRUE),
            win_pct_mean = mean(win_pct)) 

data_final %>%
  ggplot(aes(x = payroll_mean, y = win_pct_mean)) +
  geom_point(color = "blue", size = 3) +
  geom_text(aes(label = team), size = 3) +
  labs(title = "Win Percentage vs. Log Increase in Payroll",
       x = "Log Increase in Payroll",
       y = "Proportion of Wins")
```


```{r, echo=FALSE}
#Least Squared Lines
data_final %>%
  ggplot(aes(x = payroll_mean, y = win_pct_mean))+
  geom_point(size = 3)+
  geom_smooth(method = "lm", formula = y~x, color = "blue")+
  labs(title = "Win Percentage vs. Log Increase in Payroll",
       x = "Log Increase in Payroll",
       y = "Proportion of Wins")+
  theme_bw()
```

**No, there is no strong evidence to support the hypothesis that higher increases in payroll on the log scale lead to increased performance.**

**In the scatter plot, the data points do not cluster tightly.**

```{r, echo=FALSE}
datapay_long %>%
  ggplot(aes(x=diff_log, y=win_pct, group = year, color=team)) +
  geom_point()+
  geom_smooth(method="lm", formula=y~x, se=F,color = "red")+
  facet_wrap(~year) + 
  theme_bw() +
  theme(legend.position = 0)
```

**The plot indicates that log increases in payroll and performance is not consistently positively related in some years.**

```{r, echo=FALSE}
#linear relationship
fit1 = lm(win_pct_mean~payroll_mean, data = data_final)
summary(fit1)

fit2 = lm(win_pct ~ diff_log+year+year*diff_log, data = datapay_long)
summary(fit2)

#correaltion
cor(data_final$payroll_mean, data_final$win_pct_mean)
```

**R-squared in the linear model for average win percentage and average log increases in payroll is relatively small, showing the linear regression model did not fit the data well and there was no strong linear relationship between log increases in payroll and win percentage. Same for the linear regression model when considering year and interaction between year and log increases in payroll.**

**The correlation coefficient is -0.0946, indicating a low correlation.**

## Comparison

Which set of factors are better explaining performance? Yearly payroll or yearly increase in payroll? What criterion is being used? 

**I think yearly increase in payroll can better explaining performance, because the baseline payroll for each team is different. For instance, there is one team got relatively low payroll in the previous year, and the payroll increased greatly for this team in the next year. The overall yearly payroll for this team is still not as high as another team who got high payroll in the previous year though less increment in the next year. The relative increase is better to analyze the performance of one team.**




