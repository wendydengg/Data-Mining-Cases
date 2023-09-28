# Overview

Logistic regression is used for modeling categorical response variables. The simplest scenario is how to identify risk factors of heart disease? In this case the response takes a possible value of `YES` or `NO`. Logit link function is used to connect the probability of one being a heart disease with other potential risk factors such as `blood pressure`, `cholestrol level`, `weight`. Maximum likelihood function is used to estimate unknown parameters. Inference is made based on the properties of MLE. We use AIC to help nailing down a useful final model. Predictions in categorical response case is also termed as `Classification` problems. One immediately application of logistic regression is to provide a simple yet powerful classification boundaries. Various metrics/criteria are proposed to evaluate the quality of a classification rule such as `False Positive`, `FDR` or `Mis-Classification Errors`. 

LASSO with logistic regression is a powerful tool to get dimension reduction. 


## Objectives

- Understand the model
  - logit function
    + interpretation
  - Likelihood function
- Methods
    - Maximum likelihood estimators
        + Z-intervals/tests
        + Chi-squared likelihood ratio tests
- Metrics/criteria 
    - Sensitivity/False Positive
    - True Positive Prediction/FDR
    - Misclassification Error/Weighted MCE
    - Residual deviance
    - Training/Testing errors

- LASSO 

- R functions/Packages
    - `glm()`, `Anova`
    - `pROC`
    - `cv.glmnet`

## Review

Review the code and concepts covered in

* Module Logistic Regressions/Classification
* Module LASSO in Logistic Regression

## This homework

We have two parts in this homework. Part I is guided portion of work, designed to get familiar with elements of logistic regressions/classification. Part II, we bring you projects. You have options to choose one topic among either Credit Risk via LendingClub or Diabetes and Health Management. Find details in the projects. 


# Part I: Framingham heart disease study 

We will continue to use the Framingham Data (`Framingham.dat`) so that you are already familiar with the data and the variables. All the results are obtained through training data. 

Liz is a patient with the following readings: `AGE=50, GENDER=FEMALE, SBP=110, DBP=80, CHOL=180, FRW=105, CIG=0`. We would be interested to predict Liz's outcome in heart disease. 

To keep our answers consistent, use a subset of the data, and exclude anyone with a missing entry. For your convenience, we've loaded it here together with a brief summary about the data.

```{r data preparation, include=F}
# Notice that we hide the code and the results here
# Using `include=F` in the chunk declaration. 
#hd_data <- read.csv("/Users/ruolanli/Desktop/notes/data mining/Module_7_Logistic_Reg_Classification/data/Framingham.dat")
hd_data <- read.csv("data/Framingham.dat")
str(hd_data) 

### Renames, setting the variables with correct natures...
names(hd_data)[1] <- "HD"
hd_data$HD <- as.factor(hd_data$HD)
hd_data$SEX <- as.factor(hd_data$SEX)
str(hd_data)
#tail(hd_data, 1)    # The last row is for prediction
hd_data.new <- hd_data[1407,] # The female whose HD will be predicted.
hd_data <- hd_data[-1407,]  # take out the last row 
hd_data.f <- na.omit(hd_data)
```

We note that this dataset contains 311 people diagnosed with heart disease and 1095 without heart disease.
```{r table heart disease, echo = F, comment = " ", results = T}
# we use echo = F to avoid showing this R code
# notice the usage of comment = " " here in the header
table(hd_data$HD) # HD: 311 of "0" and 1095 "1" 
```

After a quick cleaning up here is a summary about the data:
```{r data summary, comment=" "}
# using the comment="     ", we get rid of the ## in the output.
summary(hd_data.f)
```

Lastly we would like to show five observations randomly chosen. 
```{r, results = T, comment=" "}
row.names(hd_data.f) <- 1:1393
set.seed(471)
indx <- sample(1393, 5)
hd_data.f[indx, ]
# set.seed(471)
# hd_data.f[sample(1393, 5), ]
```

## Identify risk factors

### Understand the likelihood function
Conceptual questions to understand the building blocks of logistic regression. All the codes in this part should be hidden. We will use a small subset to run a logistic regression of `HD` vs. `SBP`. 

i. Take a random subsample of size 5 from `hd_data_f` which only includes `HD` and `SBP`. Also set  `set.seed(471)`. List the five observations neatly below. No code should be shown here.
```{r random sample, results = T}
library(dplyr)
set.seed(471)
sample_df <- hd_data.f[sample(nrow(hd_data.f), 5), ]
sample_df <- sample_df %>%
  select(HD, SBP)
sample_df
```

ii. Write down the likelihood function using the five observations above.
**The likelihood function:**
$$
\begin{split}
L(\beta_0,\beta_1 \vert {\text Data}) 
&= P(HD=1|SBP=140) \times P(HD=0|SBP=110) \times \ldots P(HD=0|SBP=122)\\  
&=\frac{e^{\beta_0+140\beta_1}}{1+e^{\beta_0+140\beta_1}} \times \frac{1}{1+e^{\beta_0+110\beta_1}} \times \frac{e^{\beta_0+150\beta_1}}{1+e^{\beta_0+150\beta_1}} \times \frac{e^{\beta_0+260\beta_1}}{1+e^{\beta_0+260\beta_1}} \times \frac{1}{1+e^{\beta_0+122\beta_1}}
\end{split} 
$$

iii. Find the MLE based on this subset using glm(). Report the estimated logit function of `SBP` and the probability of `HD`=1. Briefly explain how the MLE are obtained based on ii. above.
```{r MLE, echo=T, results=T, warning=FALSE}
subset_fit <- glm(HD ~ SBP, data = sample_df, family =  "binomial")
summary(subset_fit)
```
- **Estimated logit function:**
$$logit(P(HD=1|SBP))=-334.96+2.56SBP$$
- **Probability of HD=1:**
$$P(HD=1|SBP) = \frac{e^{-334.96+2.56SBP}}{1+e^{-334.96+2.56SBP}}$$
- **After we obtain the likelihood function in ii., we take the log of the likelihood function and then estimate the $\beta_0$ and $\beta_1$ by maximizing the log likelihood function. To be more specific, we take the first derivative of log likelihood function and set it equal 0. Then we should also take second derivative to check maximum.**

iv. Evaluate the probability of Liz having heart disease. 
```{r predict in subset fit model, echo=T, results=T}
liz_prob <- predict(subset_fit, hd_data.new, type="response")
liz_prob
```
**The probability of Liz having heart disease is close to 0.**

### Identify important risk factors for `Heart.Disease.`

We focus on understanding the elements of basic inference method in this part. Let us start a fit with just one factor, `SBP`, and call it `fit1`. We then add one variable to this at a time from among the rest of the variables. For example
```{r, results='hide'}
fit1 <- glm(HD~SBP, hd_data.f, family=binomial)
summary(fit1)
fit1.1 <- glm(HD~SBP + AGE, hd_data.f, family=binomial)
summary(fit1.1)
# you will need to finish by adding each other variable 
# fit1.2...
```

i. Which single variable would be the most important to add?  Add it to your model, and call the new fit `fit2`.  
```{r add variable, echo=T, results=T}
# test p-value adding 
full_fit <- glm(HD ~ ., data = hd_data.f, family = "binomial")
Anova(full_fit)

# alternative: use forward selection
fit.forward <- regsubsets(HD ~., hd_data.f , nvmax=25, method="forward")
summary(fit.forward)

```


We will pick up the variable either with highest $|z|$ value, or smallest $p$ value. Report the summary of your `fit2` Note: One way to keep your output neat, we will suggest you using `xtable`. And here is the summary report looks like.
```{r the most important addition, results='asis', comment="   "}
library(xtable)
options(xtable.comment = FALSE)
fit2 <- glm(HD ~ SBP + SEX, data = hd_data.f, family = "binomial")
xtable(fit2)
```
**According to the p-value and forward selection, sex is the most important variable, therefore, we added sex variable to fit2.**

ii. Is the residual deviance of `fit2` always smaller than that of `fit1`? Why or why not?
**Yes. Smaller residual deviance indicates a better fit. Adding one variable makes the model fit the data better. Thus,the residual deviance of fit2 is always smaller than that of fit1. **  

iii. Perform both the Wald test and the Likelihood ratio tests (Chi-Squared) to see if the added variable is significant at the .01 level.  What are the p-values from each test? Are they the same? 
```{r Test for coefficient, echo=T, results=T}
# wald test calculated by hand
fit2_summary <- summary(fit2)
fit2_summary$coefficients
beta2 <- fit2_summary$coefficients[3,1]
se_beta2 <- fit2_summary$coefficients[3,2]
wald_test <- beta2/se_beta2
wald_pval <- 2*pnorm(wald_test, lower.tail = FALSE)
wald_pval

# LRT
Anova(fit2)
lrt_pval <- Anova(fit2)[2,3]
lrt_pval
```
**The p-value for coefficient of variable sex is 1.02e-10 in wald test, and the p-value is 3.83e-11 in likelihood ratio test. Both p-values indicate the added variable is significant at the .01 level. But the p-values obtained from two tests are different.**

###  Model building

Start with all variables. Our goal is to fit a well-fitting model, that is still small and easy to interpret (parsimonious).

i. Use backward selection method. Only keep variables whose coefficients are significantly different from 0 at .05 level. Kick out the variable with the largest p-value first, and then re-fit the model to see if there are other variables you want to kick out.
```{r backward selection, results = T}
# first fit full model
fit3 <- glm(HD ~ ., hd_data.f, family = "binomial")
Anova(fit3)

# kick DBP out
fit3.1 <- update(fit3, .~. -DBP)
Anova(fit3.1)

# kick FRW out
fit3.2 <- update(fit3.1, .~. -FRW)
Anova(fit3.2)

# kick CIG out
fit3.3 <- update(fit3.2, .~. -CIG)
Anova(fit3.3)
```
**In the final model using backward selection method, we keep AGE, SEX, SBP and CHOL as variables. **

ii. Use AIC as the criterion for model selection. Find a model with small AIC through exhaustive search. Does exhaustive search  guarantee that the p-values for all the remaining variables are less than .05? Is our final model here the same as the model from backwards elimination? 
```{r AIC, echo=T, results=T}
# Get the design matrix without 1's and HD
xy_design <- model.matrix(HD ~. + 0, hd_data.f)

# Attach HD as the last column
xy <- data.frame(xy_design, hd_data.f$HD)

# model selection
fit.all <- bestglm(xy, family = binomial, method = "exhaustive", IC="AIC", nvmax = 10)
fit.all$BestModels

# check significance for each variable
aic_fit <- glm(HD~AGE+SEX+SBP+CHOL+FRW+CIG, family=binomial, data=hd_data.f)
Anova(aic_fit)

# eliminate FRW
aic_fit1 <- update(aic_fit, .~. -FRW)
summary(aic_fit1)

# eliminate CIG
aic_fit_final <- update(aic_fit1, .~. -CIG)
summary(aic_fit_final)
```
**Exhaustive search can not guarantee that the p-values for all the remaining variables are less than .05. After kicking out the variables that are not significant, the final model includes AGE, SEX, SBP and CHOL. The final model here is the same as the model from backwards elimination.**

iii. Use the model chosen from part ii. as the final model. Write a brief summary to describe important factors relating to Heart Diseases (i.e. the relationships between those variables in the model and heart disease). Give a definition of “important factors”. 
```{r results='asis'}
xtable(aic_fit_final)
```

```{r OR, echo=T, results=T}
exp(coef(aic_fit_final))
```

**Summary** \par
- **The final model is:**
$$logit(P(HD=1|SBP))=-8.40872+0.05664AGE+0.98987SEX+0.01696SBP+0.00448CHOL$$
- **Interpretation: ** \par
1) age: The log odds of heart disease increases 0.05664 for each one unit increase in age, controlling for other variables. In terms of odds ratio, when age increases 1 unit, the odds ratio for age is 1.06. This means the odds of heart disease increases by 6% for each one unit increase in age controlling for other variables. \par
2) sex: The log odds of heart disease increases 0.98987 if the patient is male controlling for other variables. In terms of odds ratio, when sex is male, the odds ratio for age is 2.69. This means the odds of heart disease increase by 169% if the patient is male compared to female and controlling for other variables.\par
3) sbp: The log odds of heart disease increases 0.01696 for one unit increase in sSBP controlling for other variables. In terms of odds ration, when SBP increases 1 unit, the odds ratio for SBP is 1.017. This means the odds of heart disease increases by 1.7% for each one unit increase in SBP controlling for other variables. \par
3) chol: The log odds of heart disease increases 0.00448 for one unit increase in cholesterol controlling for other variables. In terms of odds ratio, when cholesterol increases 1 unit, the odds ratio for cholesterol is 1.004. This means the odds of heart disease increases by 0.4% for each one unit increase in cholesterol controlling for other variables. \par
- **Definition of “important factors”:**
We first found a model with smallest AIC through exhaustive search, indicating a better fit of the model to the data. Then we kicked out the variables that are not significant in this model and got the final model. The coefficients of the variables in the final model are significantly different from 0 at the alpha = 0.05 level, indicating the association between the variable and heart disease is significant controlling for other variables.

iv. What is the probability that Liz will have heart disease, according to our final model?
```{r predict using final model, echo=T, results=T}
liz_prob1 <- predict(aic_fit_final, hd_data.new, type="response")
liz_prob1
```
**According to the final model, the probability that Liz will have heart disease is 0.0336.**

##  Classification analysis

### ROC/FDR

i. Display the ROC curve using `fit1`. Explain what ROC reports and how to use the graph. Specify the classifier such that the False Positive rate is less than .1 and the True Positive rate is as high as possible.
```{r roc fit1, echo=F}
fit1.roc<- roc(hd_data.f$HD, fit1$fitted, plot=T, col="blue", sub = "Figure 1.1")

plot(fit1.roc$thresholds, 1-fit1.roc$specificities,  col="green", pch=16,  
     xlab="Threshold on prob",
     ylab="False Positive",
     main = "Thresholds vs. False Postive",
     sub = "Figure 1.2")
```
**The ROC curve (Figure 1.1) reports the sensitivity (True Positive rate) and specificity (1-False Positive rate) of different classifiers, and we want to choose a classifier on the curve such that it has both high sensitivity and high specificity at the same time. To do so, we would make one more graph for False Positive rate (1-specificity) against thresholds from 0 to 1 (Figure 1.2). Based on Figure 1.2, the threshold based on a False Positive rate of less than 0.1 is around 0.3. Thus, the classifier that has a False Positive rate less than .1 and True Positive rate as high as possible is around 0.3.**

ii. Overlay two ROC curves: one from `fit1`, the other from `fit2`. Does one curve always contain the other curve? Is the AUC of one curve always larger than the AUC of the other one? Why or why not?
```{r overlay roc fit1 and fit2, echo=F, message=FALSE}
fit1.roc<- roc(hd_data.f$HD, fit1$fitted)
fit2.roc<- roc(hd_data.f$HD, fit2$fitted)

plot(1-fit1.roc$specificities, 
     fit1.roc$sensitivities, col="blue", lwd=3, type="l",
     xlab="False Positive", 
     ylab="Sensitivity", 
     sub = "Figure 2")
lines(1-fit2.roc$specificities, fit2.roc$sensitivities, col="red", lwd=3)

legend("bottomright",
       c(paste0("fit1 AUC=", round(fit1.roc$auc,2)), 
         paste0("fit2 AUC=", round(fit2.roc$auc, 2))), 
       col=c("blue", "red"),
       lty=1)
```
**Yes, the curve that has a larger AUC will always contain the other curve because AUC measures how well a model fits a data. If a model has higher sensitivity for each specificity, it will have a larger AUC. In Figure 2, fit2 is better than fit1 because fit2 has a higher AUC.**

iii.  Estimate the Positive Prediction Values and Negative Prediction Values for `fit1` and `fit2` using .5 as a threshold. Which model is more desirable if we prioritize the Positive Prediction values?
```{r 0.5 threshold for fit1 and fit2, echo=F}
fit1.pred.5 <- ifelse(fit1$fitted > 1/2, "1", "0")
fit2.pred.5 <- ifelse(fit2$fitted > 1/2, "1", "0")

fit1.cm.5 <- table(fit1.pred.5, hd_data.f$HD)
fit2.cm.5 <- table(fit2.pred.5, hd_data.f$HD)
fit1.cm.5
fit2.cm.5

fit1.positive.pred <- fit1.cm.5[2,2] / sum(fit1.cm.5[2,]) 
fit1.negative.pred <- fit1.cm.5[1,1] / sum(fit1.cm.5[1,])
fit2.positive.pred <- fit2.cm.5[2,2] / sum(fit2.cm.5[2,])
fit2.negative.pred <- fit2.cm.5[1,1] / sum(fit2.cm.5[1,])
fit1.positive.pred
fit2.positive.pred
```
**Since fit2 as a higher Positive Prediction value of 0.472, fit2 is more desirable if we prioritize the Positive Prediction value, which measures the accuracy of the model.**

iv.  For `fit1`: overlay two curves,  but put the threshold over the probability function as the x-axis and positive prediction values and the negative prediction values as the y-axis.  Overlay the same plot for `fit2`. Which model would you choose if the set of positive and negative prediction values are the concerns? If you can find an R package to do so, you may use it directly.
```{r overlay pos and neg pred values fit1 and fit2, echo=F}
fit1.threshold <- coords(fit1.roc, ret="threshold")
fit2.threshold <- coords(fit2.roc, ret="threshold")
fit1.ppv <- coords(fit1.roc, ret="ppv")
fit1.npv <- coords(fit1.roc, ret="npv")
fit2.ppv <- coords(fit2.roc, ret="ppv")
fit2.npv <- coords(fit2.roc, ret="npv")

# ppv
plot(fit1.threshold$threshold, fit1.ppv$ppv, type="l", col="blue", pch=16,  
     xlab="Threshold on prob",
     ylab="ppv",
     main = "Thresholds vs. Positive Prediction Values",
     sub = "Figure 3.1")
lines(fit2.threshold$threshold, fit2.ppv$ppv, col="red", lwd=3)

legend("bottomright",
       c(paste0("fit1"), 
         paste0("fit2")), 
       col=c("blue", "red"),
       lty=1)

# npv
plot(fit1.threshold$threshold, fit1.npv$npv, type="l", col="blue", pch=16,  
     xlab="Threshold on prob",
     ylab="npv",
     main = "Thresholds vs. Negative Prediction Values",
     sub = "Figure 3.2")
lines(fit2.threshold$threshold, fit2.npv$npv, col="red", lwd=3)

legend("bottomright",
       c(paste0("fit1"), 
         paste0("fit2")), 
       col=c("blue", "red"),
       lty=1)
```

**If the set of ppv and npv are the concerns, then we would choose fit2 as our model.**
  
### Cost function/ Bayes Rule

Bayes rules with risk ratio $\frac{a_{10}}{a_{01}}=10$ or $\frac{a_{10}}{a_{01}}=1$. Use your final model obtained from Part 1 to build a class of linear classifiers.


i.  Write down the linear boundary for the Bayes classifier if the risk ratio of $a_{10}/a_{01}=10$.
$$logit(P(HD=1|SBP))=-8.40872+0.05664AGE+0.98987SEX+0.01696SBP+0.00448CHOL$$
$$\hat P(Y=1 \vert x) > \frac{\frac{a_{0,1}}{a_{1,0}}}{1 + \frac{a_{0,1}}{a_{1,0}}} = \frac{\frac{1}{10}}{1 + \frac{1}{10}} = \frac{0.1}{1+0.1}=0.09$$ 
$$logit > \log(\frac{0.09}{0.91})=-1.00479$$
**The linear boundary is**
$$7.40392>0.05664AGE+0.98987SEX+0.01696SBP+0.00448CHOL$$

ii. What is your estimated weighted misclassification error for this given risk ratio?
$$MCE=\frac{a_{1,0} \sum 1 \{\hat Y = 0 | Y=1\} + a_{0,1} \sum 1 \{\hat Y = 1 | Y=0\}}{n}$$
```{r weighted misclassification error, results=TRUE, echo=F}
fit.final.pred.5 <- as.factor(ifelse(aic_fit_final$fitted > .09, "1", "0"))
MCE.final.bayes <- (10*sum(fit.final.pred.5[hd_data.f$HD == "1"] != "1")
              + sum(fit.final.pred.5[hd_data.f$HD == "0"] != "0"))/length(hd_data.f$HD)
MCE.final.bayes
```
**Given this risk ratio, the estimated weighted misclassification error is 0.715.**

iii.  How would you classify Liz under this classifier?
```{r classify Liz, echo=F}
predict(aic_fit_final, hd_data.new, type="response")
```
**Under this classifier, we would classify Liz as not having heart disease.**

iv. Bayes rule gives us the best rule if we can estimate the probability of `HD-1` accurately. In practice we use logistic regression as our working model. How well does the Bayes rule work in practice? We hope to show in this example it works pretty well.
```{r Bayes rule, echo=F}
fit.final.roc<- roc(hd_data.f$HD, aic_fit_final$fitted, plot=T, col="green")

fit.final.threshold <- coords(fit.final.roc, ret="threshold")
fit.final.ppv <- coords(fit.final.roc, ret="ppv")
fit.final.npv <- coords(fit.final.roc, ret="npv")

# ppv
plot(fit.final.threshold$threshold, fit.final.ppv$ppv, type="l", col="green", pch=16,  
     xlab="Threshold on prob",
     ylab="ppv",
     main = "Thresholds vs. Positive Prediction Values",
     sub = "Figure 4.1")
lines(fit1.threshold$threshold, fit1.ppv$ppv, col="blue", lwd=3)
lines(fit2.threshold$threshold, fit2.ppv$ppv, col="red", lwd=3)

legend("bottomright",
       c(paste0("fitfinal"),
         paste0("fit1"), 
         paste0("fit2")), 
       col=c("green", "blue", "red"),
       lty=1)

# npv
plot(fit.final.threshold$threshold, fit.final.npv$npv, type="l", col="green", pch=16,  
     xlab="Threshold on prob",
     ylab="npv",
     main = "Thresholds vs. Negative Prediction Values",
     sub = "Figure 4.2")
lines(fit1.threshold$threshold, fit1.ppv$npv, col="blue", lwd=3)
lines(fit2.threshold$threshold, fit2.npv$npv, col="red", lwd=3)

legend("bottomright",
       c(paste0("fitfinal"),
         paste0("fit1"), 
         paste0("fit2")), 
       col=c("green", "blue", "red"),
       lty=1)
```
**Based on the threshold vs ppv and threshold vs npv graphs, the final fit has a higher ppv and npv for each threshold compared to fit1 and fit2, which shows that the Bayes rules does work well in practice.**

Now, draw two estimated curves where x = threshold, and y = misclassification errors, corresponding to the thresholding rule given in x-axis.
```{r threshold vs misclassification error, echo=F}
fit.final.error <- data.frame(threshold = fit.final.roc$thresholds, 
                           error.risk10 =(10*307*(1-fit.final.roc$sensitivities)
                                          +1*1086*(1-fit.final.roc$specificities))/length(hd_data.f$HD),
                           error.risk1 =(1*307*(1-fit.final.roc$sensitivities)
                                          +1*1086*(1-fit.final.roc$specificities))/length(hd_data.f$HD))

plot(fit.final.error$threshold, fit.final.error$error.risk10, col = "blue", pch = 16,
     xlab = "threshold", ylab = "weighted misclassfication error", main = "threshold vs weighted   
     misclassification error", ylim=c(0, 1))
lines(fit.final.error$threshold, fit.final.error$error.risk1, col="red", lwd=3)
```

v. Use weighted misclassification error, and set $a_{10}/a_{01}=10$. How well does the Bayes rule classifier perform? 
```{r misclassification error 10, echo=F}
fit.final.10 <- as.factor(ifelse(aic_fit_final$fitted > .09, "1", "0"))
MCE.10 <- (10*sum(fit.final.10[hd_data.f$HD == "1"] != "1")
              + sum(fit.final.10[hd_data.f$HD == "0"] != "0"))/length(hd_data.f$HD)
MCE.10
```
**When $a_{10}/a_{01}=10$, we have a misclassification error of 0.715.**

vi. Use weighted misclassification error, and set $a_{10}/a_{01}=1$. How well does the Bayes rule classifier perform? 
```{r misclassification error 1, echo=F}
fit.final.1 <- as.factor(ifelse(aic_fit_final$fitted > .09, "1", "0"))
MCE.1 <- (sum(fit.final.1[hd_data.f$HD == "1"] != "1")
              + sum(fit.final.1[hd_data.f$HD == "0"] != "0"))/length(hd_data.f$HD)
MCE.1
```
**When $a_{10}/a_{01}=1$, we have a misclassification error of 0.67, which is better than the case when  $a_{10}/a_{01}=10$.**

# Part II: Project

## Project Option 1 Credit Risk via LendingClub

## Project Option 2 Diabetes and Health Management

