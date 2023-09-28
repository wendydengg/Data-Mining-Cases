# Overview

For the purpose of predictions, a nonlinear model or a model free approach could be beneficial. 

Neural networks are a good way to do prediction because they are capable of learning complex patterns and relationships within data, without requiring explicit programming of rules. They are able to recognize non-linear relationships and make predictions based on those patterns, which can be very difficult to accomplish with traditional statistical models. Neural networks can also be used for a wide range of prediction tasks, including image and speech recognition, natural language processing, and time series forecasting.

Neural networks have been around for several decades, but they have become more popular and widely used in recent years due to several factors. One reason is the increase in available data, which allows for more robust and accurate training of neural networks. Additionally, advancements in hardware technology have made it easier and more cost-effective to train large neural networks. Another reason is the development of new neural network architectures, such as convolutional neural networks (CNNs) and recurrent neural networks (RNNs), which have proven to be very effective for specific tasks. Finally, the rise of deep learning, which is a subfield of machine learning that utilizes deep neural networks with many layers, has further propelled the popularity of neural networks due to their ability to achieve state-of-the-art performance on a wide range of tasks.Neural Network is a natural extension of the linear models. 

On a model free side, a binary decision tree is the simplest, still interpretable and often provides insightful information between predictors and responses. To improve the predictive power we would like to aggregate many equations, especially uncorrelated ones. One clever way to have many free samples is to take bootstrap samples. For each bootstrap sample we  build a random tree by taking a randomly chosen number of variables to be split at each node. We then take average of all the random bootstrap trees to have our final prediction equation. This is RandomForest. 

Ensemble method can be applied broadly: simply take average or weighted average of many different equations. This may beat any single equation in your hand.


All the methods covered can handle both continuous responses as well as categorical response with multiple levels (not limited to binary response.)


## Objectives

- Understand a basic NN 
  + Be able to write an architecture (a model)
  + Understand the roles of 
    - Hidden layers
    - Neurons
    - Relu activation function
    
  + Be able to run keras to train and to use a NN. 

- Understand trees
    + single tree/displaying/pruning a tree
    + RandomForest
    + Ensemble idea

- R functions/Packages
    + `kears`
    + `tree`, `RandomForest`, `ranger`
    
- Json data format

- text mining
    + bag of words
  

Data needed:

+ `yelp_review_20k.json`
+ `IQ.Full.csv`


# Problem 0: Lectures

Please study all the lectures. Understand the main elements in each lecture and be able to run and compile the lectures

+ textmining
+ deep learning
+ trees
+ boosting



# Problem 1: Yelp challenge 2019

**Note:** This problem is rather involved. It covers essentially all the main materials we have done so far in this semester. It could be thought as a guideline for your final project if you want when appropriate. 

Yelp has made their data available to public and launched Yelp challenge. [More information](https://www.yelp.com/dataset/). It is unlikely we will win the $5,000 prize posted but we get to use their data for free. We have done a detailed analysis in our lecture. This exercise is designed for you to get hands on the whole process. 

For this case study, we downloaded the [data](https://www.yelp.com/dataset/download) and took a 20k subset from **review.json**. *json* is another format of a data. It is flexible and commonly-used for websites. Each item/subject/sample is contained in a brace *{}*. Data is stored as **key-value** pairs inside the brace. *Key* is the counterpart of column name in *csv* and *value* is the content/data. Both *key* and *value* are quoted. Each pair is separated by a comma. The following is an example of one item/subject/sample.

```{json}
{
  "key1": "value1",
  "key2": "value2"
}
```


**Data needed:** yelp_review_20k.json available in Canvas.

**yelp_review_20k.json** contains full review text data including the user_id that wrote the review and the business_id the review is written for. Here's an example of one review.

```{json}
{
    // string, 22 character unique review id
    "review_id": "zdSx_SD6obEhz9VrW9uAWA",

    // string, 22 character unique user id, maps to the user in user.json
    "user_id": "Ha3iJu77CxlrFm-vQRs_8g",

    // string, 22 character business id, maps to business in business.json
    "business_id": "tnhfDv5Il8EaGSXZGiuQGg",

    // integer, star rating
    "stars": 4,

    // string, date formatted YYYY-MM-DD
    "date": "2016-03-09",

    // string, the review itself
    "text": "Great place to hang out after work: the prices are decent, and the ambience is fun. It's a bit loud, but very lively. The staff is friendly, and the food is good. They have a good selection of drinks.",

    // integer, number of useful votes received
    "useful": 0,

    // integer, number of funny votes received
    "funny": 0,

    // integer, number of cool votes received
    "cool": 0
}
```

## Goal of the study

The goals are 

1) Try to identify important words associated with positive ratings and negative ratings. Collectively we have a sentiment analysis.  

2) To predict ratings using different methods. 

## JSON data and preprocessing data

i. Load *json* data

The *json* data provided is formatted as newline delimited JSON (ndjson). It is relatively new and useful for streaming.
```{json}
{
  "key1": "value1",
  "key2": "value2"
}
{
  "key1": "value1",
  "key2": "value2"
}
```

The traditional JSON format is as follows.
```{json}
[{
  "key1": "value1",
  "key2": "value2"
},
{
  "key1": "value1",
  "key2": "value2"
}]
```


We use `stream_in()` in the `jsonlite` package to load the JSON data (of ndjson format) as `data.frame`. (For the traditional JSON file, use `fromJSON()` function.)

```{r}
pacman::p_load(jsonlite)
yelp_data <- jsonlite::stream_in(file("data/yelp_review_20k.json"), verbose = F)
str(yelp_data)  
# different JSON format
# tmp_json <- toJSON(yelp_data[1:10,])
# fromJSON(tmp_json)
```

**Write a brief summary about the data:**

a) Which time period were the reviews collected in this data?
**These reviews were collected between 10/19/2004-10/4/2018.**
```{r summary}
yelp_data$date <- ymd_hms(yelp_data$date)
yelp_data$date <- as.Date(yelp_data$date)

summary(yelp_data$date)
```


b) Are ratings (with 5 levels) related to month of the year or days of the week? Only address this through EDA please. 
**Based on Figure 1, we don't observe any clear pattern regarding star rating and month. Although there are some months with fewer reviews overall (for example, November), the ratio between the star ratings is more or less consistent across months. The same is true of day of the week, with no particular day seeming to have a different distribution in ratings than any other, as seen in Figure 2.**
```{r eda, results = "markup"}
yelp_data <- yelp_data %>% 
  mutate(weekday = weekdays(date), month = month(date))

rating_months <- ggplot(data = yelp_data, aes(x=stars, fill=as.factor(stars))) +
  geom_bar(show.legend = FALSE) +
  facet_wrap(~month, scales = "free", labeller = labeller(month = \(.) month.abb[as.integer(.)])) + 
  theme_bw() +
  labs(title = "Figure 1: Star Rating by Month")
rating_months

rating_days <- ggplot(data = yelp_data, aes(x=stars, fill=as.factor(stars))) +
  geom_bar(show.legend = FALSE) +
  facet_wrap(~weekday, scales = "free") + 
  theme_bw() +
  labs(title = "Figure 2: Star Rating by Day of the Week")
rating_days

prop.table(table(yelp_data$stars, yelp_data$weekday), 1)
prop.table(table(yelp_data$stars, yelp_data$month), 1)
```


ii. Document term matrix (dtm) (bag of words)
 
 Extract document term matrix for texts to keep words appearing at least .5% of the time among all 20000 documents. Go through the similar process of cleansing as we did in the lecture. 
 
```{r bag of words}
yelp_text <- yelp_data$text
mycorpus1 <- VCorpus(VectorSource(yelp_text))

mycorpus_clean <- tm_map(mycorpus1, content_transformer(tolower))
mycorpus_clean <- tm_map(mycorpus_clean, removeWords, stopwords("english"))
mycorpus_clean <- tm_map(mycorpus_clean, removePunctuation)
mycorpus_clean <- tm_map(mycorpus_clean, removeNumbers)

dtm1 <- DocumentTermMatrix(mycorpus_clean)
dtm1 <- removeSparseTerms(dtm1, 1-.005)

inspect(dtm1)
inspect(dtm1[100, 405])
```
 

a) Briefly explain what does this matrix record? What is the cell number at row 100 and column 405? What does it represent?
**The sparsity matrix provides the number of times each word is used in a given review, where rows represent individual reviews and columns represent a word. The cell number for row 100 column 405 is "0", indicating that review number 100 used the word "event" 0 times.**

b) What is the sparsity of the dtm obtained here? What does that mean?
**The sparsity of the dtm after eliminating words that appear in less than 0.5% of reviews is 98%, which tells us that 98% of the cells in our matrix are 0 (i.e., the word is not used in the review).**

c) Set the stars as a two category response variable called rating to be “1” = 5,4 and “0”= 1,2,3. Combine the variable rating with the dtm as a data frame called data2. 
```{r revise ratings}
yelp_data <- yelp_data %>% mutate(rating = ifelse(stars == 5 | stars == 4, 1, 0))
table(yelp_data$stars, yelp_data$rating)
yelp_data_sub <- select(yelp_data, review_id, rating)

data2 <- data.frame(yelp_data_sub, as.matrix(dtm1)) 
dim(data2)
names(data2)[1:30]

```



## Analyses

Get a training data with 13000 reviews and the 5000 reserved as the testing data. Keep the rest (2000) as our validation data set. 
```{r splitting data}
set.seed(12345)
n <- nrow(data2)

idx_train <- sample(n, 13000)
idx_no_train <- (which(! seq(1:n) %in% idx_train))
idx_test <- sample(idx_no_train, 5000)
idx_val <- which(! idx_no_train %in% idx_test)

data2.test <- data2[idx_test, -c(1)]
data2.train <- data2[idx_train, -c(1)]
data2.val <- data2[idx_val, -c(1)]
dim(data2.train)
dim(data2.test)
dim(data2.val)
```


### LASSO

i. Use the training data to get Lasso fit. Choose lambda.1se. Label the the fit as fit.lasso. Comment on what tuning parameters are chosen at the end and why?
```{r lasso}
set.seed(12345)
Y <- as.matrix(data2.train$rating)
X <- sparse.model.matrix(rating~., data=data2.train)[, -1]
fit.lasso <- cv.glmnet(X, Y, alpha=.99, family="binomial")

coef.1se <- coef(fit.lasso, s="lambda.1se") 
lasso.words <- coef.1se@Dimnames[[1]] [coef.1se@i][-1]

```


ii. Feed the output from Lasso above, get a logistic regression and call this fit.glm
```{r glm from lasso}
sel_cols <- c("rating", lasso.words)
data2.train.lasso <- data2.train %>% select(all_of(sel_cols))

fit.glm <- glm(data = data2.train.lasso, rating ~ ., family = binomial())
  
```

	
a) Pull out all the positive coefficients and the corresponding words. Rank the coefficients in a decreasing order. Report the leading 2 words and the coefficients. Describe briefly the interpretation for those two coefficients. 
**The two leading words and their coefficients are refreshing (2.33) and perfection (1.82). The coefficients indicate that when a review contains the word refreshing or perfection, the review has a 3,587% or 3,027% increase, respectively, in the odds of being a positive versus a negative review.**
```{r positive words}

result.fit.coef <- coef(fit.glm)

good.glm <- result.fit.coef[which(result.fit.coef > 0)]
good.glm <- good.glm[-1]
good.glm <- sort(good.glm, decreasing = TRUE)
head(good.glm, 2)
exp(3.58)
exp(3.41)

```


b) Make a word cloud with the top 100 positive words according to their coefficients. Interpret the cloud briefly.
**The below word cloud shows us the top 100 words which are associated with positive (3, 4, or 5 stars) reviews. The size of the word indicates the strength of the likelihood that a review is good if that word is used; we can see that word such as "knowledge," "accommodating," and "perfection" are very strongly associated with good reviews. Other positive words, like "happy," "loved," and "favorite" are less strongly associated with positive reviews.**
```{r positive word cloud, output = "markup"}
set.seed(12345)
good.fre <- sort(good.glm, decreasing = TRUE)
good.word <- names(good.fre) 

pos.colors <- brewer.pal(7,"Set2")
wordcloud(good.word[1:100], good.fre[1:100], colors=pos.colors, ordered.colors=F, min.freq = 0, c(2,.25))

```

c) Repeat a) and b) above for the bag of negative words.
```{r negative words}
bad.glm <- result.fit.coef[which(result.fit.coef < 0)]
bad.glm <- bad.glm[-1]
bad.glm <- sort(bad.glm, decreasing = TRUE)
tail(bad.glm, 2)
exp(-3.59)
exp(-4.66)
```
```{r negative word cloud, output = "markup"}
set.seed(12345)
bad.fre <- sort(-bad.glm, decreasing = TRUE)
bad.word <- names(bad.fre) 

neg.colors <- brewer.pal(7,"Set1")
wordcloud(bad.word[1:100], bad.fre[1:100], colors=neg.colors, ordered.colors=F, min.freq = 0, c(2,.25))
```


d) Summarize the findings. 
**The two most negative words and their coefficients are poor (-2.16) and disgusting (-2.49). This tells us that a review that uses the word "poor" is 97.3% less likely to be positive than a review that doesn't use that word, and a review that uses the word "disgusting" is 99.1% less likely to be positive than a review that doesn't use that word. The below word cloud summarizes the 100 most negative words according to our analysis. We can see that both refund and worse show up in large font in agreement with our above assessment; other words, such as "disgusting," "disappointing," and "poor" are also strongly associated with bad reviews.**

iii. What are the major differences among the two methods used so far: Lasso and glm
**GLM may not run when we have a large number of parameters, particularly when the number of parameters is larger than the sample size. It may also overfit the data. LASSO is therefore a valuable method because by imposing constraints on the coefficients, we are able to eliminate coefficients which may not be meaningful in our model. LASSO also involves some randomness through K-fold cross-validation, which means that while a model obtained from GLM will be the same every time, a model obtained via LASSO will not be (unless the same seed is used each time).**

iv.  Using majority votes find the testing errors
	a) From `fit.lasso`
  b) From `fit.glm`
	c) Which one is smaller?
**The testing error for glm is 0.19 whereas the testing error for LASSO is 0.13; the testing error for LASSO is smaller.**
```{r testing errors for lasso and glm}
predict.glm <- predict(fit.glm, data2.test, type = "response")
class.glm <- ifelse(predict.glm > .5, "1", "0")
testerror.glm <- mean(data2.test$rating != class.glm)
testerror.glm 

predict.lasso.p <- predict(fit.lasso, as.matrix(data2.test[, -1]), type = "response", s="lambda.1se")
predict.lasso <- predict(fit.lasso, as.matrix(data2.test[, -1]), type = "class", s="lambda.1se")
testerror.lasso <- mean(data2.test$rating != predict.lasso)
testerror.lasso
```


### Neural network

i. Let's specify an architecture with the following specifications
  a) One hidden layers with 20 neurons
  b) Relu activation function
  c) Softmax output
  d) Explain in a high level what is the model? How many unknown weights (parameters are there)
**A neural network is a form of deep learning which uses a layered structure to improve data processing. Our basic model here includes three layers - the input layer, one hidden layer, and the output layer. Our input layer includes 354 neurons (the predictive words included), we have specified that our hidden layer will include 20 neurons, and our output includes 2 neurons because we are predicting a binary outcome. Given this, there are 7,142 parameters in our model.**
```{r archetecture}
set.seed(12345)

data2_xtrain <- data2.train[-1]
data2_ytrain <- data2.train[1]   
data2_xtrain <- as.matrix(data2_xtrain)
data2_ytrain <- as.matrix(data2_ytrain)

p <- dim(data2_xtrain)[2]
model <- keras_model_sequential() %>%
  layer_dense(units = 20, activation = "relu", input_shape = c(p)) %>% 
  layer_dense(units = 2, activation = "softmax") # output
print(model)
```

ii. Train your model and call it `fit.nn` 
  a) using the training data
  b) split 85% vs. 15% internally
  c) find the optimal epoch
**Based on our plot of fit.nn, it appears that loss bottoms out around epoch 10; we will therefore use 10 epochs.**
```{r fit.nn}
model %>% compile(
  optimizer = "rmsprop",
  loss = "sparse_categorical_crossentropy",
  metrics = c("accuracy")
)

fit.nn <- model %>% fit(
  data2_xtrain,
  data2_ytrain,
  epochs = 20, 
  batch_size = 512,
  validation_split = .15
)
plot(fit.nn) #Assess appropriate number of epochs. Looks like loss bottoms out around epoch 10.

```

iii. Report the testing errors using majority vote.
**Our accuracy for this model is 87.9%, which equates to a misclassification error of 12.1%. The loss for this model is 0.32.**
```{r neural network prediction}
set_random_seed(12345)

data2_xtest <- data2.test[-1]
data2_ytest <- data2.test[1]   
data2_xtest <- as.matrix(data2_xtest)
data2_ytest <- as.matrix(data2_ytest)

p <- dim(data2_xtrain)[2] # number of input variables

model <- keras_model_sequential() %>%
  layer_dense(units = 20, activation = "relu", input_shape = c(p)) %>% 
  layer_dense(units = 2, activation = "softmax")

model %>% compile(
  optimizer = "rmsprop",
  loss = "sparse_categorical_crossentropy",
  metrics = c("accuracy")
)

model %>% fit(data2_xtrain, data2_ytrain, epochs = 10, batch_size = 512)

results <- model %>% evaluate(data2_xtest, data2_ytest)
results
```


### Random Forest  

i. Briefly summarize the method of Random Forest
**The structures of the trees are very similar to that of using the whole dataset. To build uncorrelated trees, Random Forest builds trees based on bootstrap samples and variables that are randomly chosen each time. For regression trees, the predicted value is the average of the predicted values each time. For classification trees, the predicted label is based on majority vote.**

ii. Now train the data using the training data set by RF and call it `fit.rf`. 
```{r RF, echo=T, results=T}
# train the data using RF
#fit.rf <- randomForest(as.factor(rating)~., data2.train, mtry=sqrt(ncol(data2.train)-1), ntree=500)  This would take about 20 mins, I saved it as .rds file
#saveRDS(fit.rf, "fit.rf.rds")
fit.rf <- readRDS("fit.rf.rds")
plot(fit.rf)
legend("topright", colnames(fit.rf$err.rate), col = 1:3, cex=0.8, fill=1:3)
```
  a) Explain how you tune the tuning parameters (`mtry` and `ntree`). 
    **I used the default settings for classification trees, where `mtry` is the square root of number of variables and `ntree` is 500.**
    
  b) Get the testing error of majority vote. 
```{r testing error of RF, echo=T, results=T}
predict.rf.y <- predict(fit.rf, newdata=data2.test) 
rf.test.err <- mean(data2.test$rating != predict.rf.y)
rf.test.err
```
**The testing error is 0.1416.**

###  PCA first

i. Perform PCA (better to do sparse PCA) for the input matrix first. Decide how many PC's you may want to take and why.
```{r, echo=T, results=T}
# sparse pca takes very long time, so I just used normal pca
pca.all <- prcomp(data2[, -c(1,2)], center = TRUE, scale = TRUE)
# PVE plot
plot(summary(pca.all)$importance[2, ], pch=16,
     ylab="PVE",
     xlab="Number of PCs",
     main="PVE scree plot of PCA")

#  plot of CPVE
plot(summary(pca.all)$importance[3, ], pch=16,
     ylab="Cumulative PVE",
     xlab="Number of PCs",
     main="Scree plot of Cumulative PVE ")
```
**From the two plots above, we can see 80% of the total variability are explained by 1,000 PCs **

ii. Pick up one of your favorite methods above and build the predictive model with PC's. Say you use RandomForest.
```{r use RF to build predictive model based on PCs, echo=T, results=T}
# create a new df with PCs
data.pca <- data.frame(pca.all$x[, c(1:1000)]) #1000 PCs
rating <- data2$rating
data.pca <- cbind(rating, data.pca)

# split data.pca
set.seed(12345)
n <- nrow(data.pca)

idx_train_pca <- sample(n, 13000)
idx_no_train_pca <- (which(! seq(1:n) %in% idx_train_pca))
idx_test_pca <- sample(idx_no_train_pca, 5000)

data2.train.pca <- data.pca[idx_train_pca,]
data2.test.pca <- data.pca[idx_test_pca,]



# use RF to build the model
# fit.rf.pca <- randomForest(as.factor(rating)~., data2.train.pca, mtry=sqrt(ncol(data2.train.pca)-1), ntree=500) This would also take a while, also saved it as .rds file
# saveRDS(fit.rf.pca, "fit.rf.pca.rds")
fit.rf.pca <- readRDS("fit.rf.pca.rds")
```

iii. What is the testing error? Is this testing error better than that obtained using the original x's? 
```{r testing error of RF based on pca, echo=T, results=T}
predict.rf.y.pca <- predict(fit.rf.pca, newdata=data2.test.pca) 
rf.test.err.pca <- mean(data2.test.pca$rating != predict.rf.y.pca)
rf.test.err.pca
```
**Testing error is 0.223. The testing error of original data is 0.1416. Using orginal data would return better testing error compared to use 1,000 PCs.**

### Ensemble model

i. Take average of some of the  models built above (also try all of them) and this gives us the fit.em. Report it's testing error. (Do you have more models to be bagged, try it.)
```{r ensemble, echo=T, results=T}
# get predicted value from each method
variable_mtx <- as.matrix(data2.test[,-1])
lasso_pred <- predict(fit.lasso, variable_mtx, s = fit.lasso$lambda.1se, type="response")
glm_pred <- predict.glm
nn_pred <- model %>% predict(variable_mtx)
nn_pred <- nn_pred[,2]
rf_pred <- predict(fit.rf, newdata=data2.test, type="prob")[,2]


# take average
fit.em <- (lasso_pred + glm_pred + nn_pred + rf_pred) / 4
predict.em.y <- ifelse(fit.em > 0.5, 1, 0)
em.test.err <- mean(data2.test$rating != predict.em.y)
em.test.err
```
**The testing error of the ensemble model is 0.1204.**

## Final model

Which classifier(s) seem to produce the least testing error? Are you surprised? Report the final model and accompany the validation error. Once again this is THE only time you use the validation data set.  For the purpose of prediction, comment on how would you predict a rating if you are given a review (not a tm output) using our final model? 

```{r final model for validation, echo=T, results=T}
# get predicted value from each method
variable_mtx <- as.matrix(data2.val[,-1])
lasso_pred <- predict(fit.lasso, variable_mtx, s = fit.lasso$lambda.1se, type="response")
glm_pred <- predict(fit.glm, data2.val, type = "response")
nn_pred <- model %>% predict(variable_mtx)
nn_pred <- nn_pred[,2]
rf_pred <- predict(fit.rf, newdata=data2.val, type="prob")[,2]


# take average
fit.em <- (lasso_pred + glm_pred + nn_pred + rf_pred) / 4
predict.em.y <- ifelse(fit.em > 0.5, 1, 0)
em.val.err <- mean(data2.val$rating != predict.em.y)
em.val.err
```

**The ensemble model produces the least testing error. This is an expected result, since we are trying to bag uncorrelated predictive equations to reduce the variance. The final model should be the average of four predictive equations, which include lasso, glm, nn and random forest. The validation error is 0.093. In final model, we calculate the average predicted value from four predictions for each review, when the average predicted value is greater than 0.5, we would say the rating is 1, which indicates good reviews. When the average predicted value is less than 0.5, the rating would be 0, which means bad reviews.**



# Problem 2: IQ and successes

## Background: Measurement of Intelligence

Case Study:  how intelligence relates to one's future successes?

**Data needed: `IQ.Full.csv`**

ASVAB (Armed Services Vocational Aptitude Battery) tests have been used as a screening test for those who want to join the army or other jobs.

Our data set IQ.csv is a subset of individuals from the 1979 National Longitudinal Study of
Youth (NLSY79) survey who were re-interviewed in 2006. Information about family, personal demographic such as gender, race and education level, plus a set of ASVAB (Armed Services Vocational Aptitude Battery) test scores are available. It is STILL used as a screening test for those who want to join the army! ASVAB scores were 1981 and income was 2005.

**Our goals:**

+ Is IQ related to one's successes measured by Income?
+ Is there evidence to show that Females are under-paid?
+ What are the best possible prediction models to predict future income?


**The ASVAB has the following components:**

+ Science, Arith (Arithmetic reasoning), Word (Word knowledge), Parag (Paragraph comprehension), Numer (Numerical operation), Coding (Coding speed), Auto (Automative and Shop information), Math (Math knowledge), Mechanic (Mechanic Comprehension) and Elec (Electronic information).
+ AFQT (Armed Forces Qualifying Test) is a combination of Word, Parag, Math and Arith.
+ Note: Service Branch requirement: Army 31, Navy 35, Marines 31, Air Force 36, and Coast Guard 45,(out of 100 which is the max!)

**The detailed variable definitions:**

Personal Demographic Variables:

 * Race: 1 = Hispanic, 2 = Black, 3 = Not Hispanic or Black
 * Gender: a factor with levels "female" and "male"
 * Educ: years of education completed by 2006

Household Environment:

* Imagazine: a variable taking on the value 1 if anyone in the respondent’s household regularly read
	magazines in 1979, otherwise 0
* Inewspaper: a variable taking on the value 1 if anyone in the respondent’s household regularly read
	newspapers in 1979, otherwise 0
* Ilibrary: a variable taking on the value 1 if anyone in the respondent’s household had a library card
	in 1979, otherwise 0
* MotherEd: mother’s years of education
* FatherEd: father’s years of education

Variables Related to ASVAB test Scores in 1981 (Proxy of IQ's)

* AFQT: percentile score on the AFQT intelligence test in 1981
* Coding: score on the Coding Speed test in 1981
* Auto: score on the Automotive and Shop test in 1981
* Mechanic: score on the Mechanic test in 1981
* Elec: score on the Electronics Information test in 1981

* Science: score on the General Science test in 1981
* Math: score on the Math test in 1981
* Arith: score on the Arithmetic Reasoning test in 1981
* Word: score on the Word Knowledge Test in 1981
* Parag: score on the Paragraph Comprehension test in 1981
* Numer: score on the Numerical Operations test in 1981

Variable Related to Life Success in 2006

* Income2005: total annual income from wages and salary in 2005. We will use a natural log transformation over the income.


**Note: All the Esteem scores shouldn't be used as predictors to predict income**

## 1. EDA: Some cleaning work is needed to organize the data.

```{r EDA}
IQ <- read.csv("data/IQ.Full.csv")
#str(IQ)
#names(IQ)
#summary(IQ)
#dim(IQ)

# The first variable is the label for each person. Take that out.
IQ <- IQ[,-1]

# Set categorical variables as factors
IQ$Race <- as.factor(IQ$Race)
IQ$Gender <- as.factor(IQ$Gender)
IQ$Imagazine <- as.factor(IQ$Imagazine)
IQ$Inewspaper <- as.factor(IQ$Inewspaper)
IQ$Ilibrary <- as.factor(IQ$Ilibrary)

# Make log transformation for Income and take the original Income out
IQ <- IQ %>%
  mutate(LogIncome = log(Income2005)) %>%
  select(-Income2005)

# Take the last person out of the dataset and label it as **Michelle**.
Michelle <- tail(IQ, 1)
IQ <- IQ %>%
  filter(row_number() <= n()-1)

# When needed, split data to three portions: training, testing and validation (70%/20%/10%)
set.seed(1)
n <- nrow(IQ)
validation.index <- sample(n, n*0.10) # 10% of IQ is validation data
test.idx <- sample(setdiff(1:n, validation.index), n*0.20)
train.idx <- setdiff(setdiff(1:n, validation.index), test.idx)
IQ.val <- IQ[validation.index, ]
IQ.test <- IQ[test.idx, ]
IQ.train <- IQ[train.idx, ]

#summary(IQ.val)
#summary(IQ.test)
#summary(IQ.train)
```

+ The first variable is the label for each person. Take that out.
+ Set categorical variables as factors.
+ Make log transformation for Income and take the original Income out
+ Take the last person out of the dataset and label it as **Michelle**.
+ When needed, split data to three portions: training, testing and validation (70%/20%/10%)
  - training data: get a fit
  - testing data: find the best tuning parameters/best models
  - validation data: only used in your final model to report the accuracy.


## 2. Factors affect Income

We only use linear models to answer the questions below.

i. To summarize ASVAB test scores, create PC1 and PC2 of 10 scores of ASVAB tests and label them as
ASVAB_PC1 and ASVAB_PC2. Give a quick interpretation of each ASVAB_PC1 and ASVAB_PC2 in terms of the original 10 tests.

```{r PC1 PC2, echo=F}
pc.tests <- prcomp(IQ[,  c(10:19)], scale=TRUE)
pc.tests.loading <- data.frame(tests=row.names(pc.tests$rotation), pc.tests$rotation)

pc.tests.loading %>% select(PC1) %>% arrange(-PC1)
pc.tests.loading %>% select(PC2) %>% arrange(-PC2)

ASVAB_PC1 <- pc.tests$x[,1]
ASVAB_PC2 <- pc.tests$x[,2]
```

ii. Is there any evidence showing ASVAB test scores in terms of ASVAB_PC1 and ASVAB_PC2, might affect the Income?  Show your work here. You may control a few other variables, including gender.

```{r PC1 PC2 Income}
pc.model <- lm(LogIncome ~ ASVAB_PC1 + ASVAB_PC2 + Gender, data = IQ)
summary(pc.model)
```
**Since the p-values for the two PCs and gender are small regardless of significance level, we can conclude that there is evidence showing ASVAB scores in terms of the two PCs might affect income.**

iii. Is there any evidence to show that there is gender bias against either male or female in terms of income in the above model?

```{r}
car::Anova(pc.model)
```
**According to Anova results, since gender has a p-value of < 2.2e-16, we reject the null hypothesis, meaning that there is gender bias against either male or female in terms of income in the above model.**

We next build a few models for the purpose of prediction using all the information available. From now on you may use the three data sets setting (training/testing/validation) when it is appropriate.

## 3. Trees

i. fit1: tree(Income ~ Educ + Gender, data.train) with default set up

```{r fit1, echo=F}
fit1 <- tree(LogIncome ~ Educ + Gender, IQ.train)
fit1
data.table::data.table(fit1$frame)
```

    a) Display the tree

```{r fit1 plot, echo=F}
plot(fit1)
text(fit1)
```

    b) How many end nodes? Briefly explain how the estimation is obtained in each end nodes and describe the prediction equation

**There are four end nodes, meaning prediction for LogIncome using Educ and Gender as the two predictors only takes one of the four values. For example, if Gender is male and years of Education completed before 2006 is less than 15.5, then the predicted LogIncome for that person is 11.22.**

    c) Does it show interaction effect of Gender and Educ over Income?

**Since Educ has a split after Gender, there is interaction effect of Gender and Educ over Income.**

    d) Predict Michelle's income

```{r fit1 predict Michelle, echo=F}
predict(fit1, Michelle)
```

**Michelle's LogIncome is predicted to be 9.97.**

ii. fit2: fit2 <- rpart(Income2005 ~., data.train, minsplit=20, cp=.009)

```{r fit2, echo=F}
fit2 <- rpart(LogIncome ~., IQ.train, minsplit=20, cp=.009)
fit2
#summary(fit2)
```

    a) Display the tree using plot(as.party(fit2), main="Final Tree with Rpart")

```{r fit2 plot, echo=F}
plot(as.party(fit2), main="Final Tree with Rpart")
```

    b) A brief summary of the fit2

**The model fit2 has seven end node, and this tree takes all of the predictor variables into consideration and splits only if there are at least 20 observations in a node. Out of all of the predictor variables, Gender has the greatest importance, hence it is the root of the tree. For example, if a person is male and has completed at least 15.5 years of Education, then his predicted LogIncome is 11.22.**

    c) Compare testing errors between fit1 and fit2. Is the training error from fit2 always less than that from fit1? Is the testing error from fit2 always smaller than that from fit1?

```{r testing errors, echo=F}
train.error.fit1 <- mean((predict(fit1, IQ.train) - IQ.train$LogIncome)^2)
train.error.fit2 <- mean((predict(fit2, IQ.train) - IQ.train$LogIncome)^2)
train.error.fit1
train.error.fit2

test.error.fit1 <- mean((predict(fit1, IQ.test) - IQ.test$LogIncome)^2)
test.error.fit2 <- mean((predict(fit2, IQ.test) - IQ.test$LogIncome)^2)
test.error.fit1
test.error.fit2
```
**Since fit2 uses all of the predictor variables, we would expect its training error to be always less than that from fit1 because fit2 should provide a better fit to the training data. We can see that the training error for fit2 (0.7437005) is less than that of fit1 (0.7900584). However, we cannot expect the testing error of fit2 to be always less than that from fit1 because using more predictor variables doesn't always result in a better model, and it is possible that fit2 overfits the training data, which can result in a higher testing error. As we can see, the testing error for fit2 (0.6823709) is also less than that of fit1 (0.7059736).**

    d) You may prune the fit2 to get a tree with small testing error.

```{r fit2 prune, echo=F}
# fit a tree with large size
fit2.t <- tree(LogIncome ~., IQ.train,
          control=tree.control(nobs=nrow(IQ.train),
                minsize = 4,
                mindev=0.007))
fit2.p <- prune.tree(fit2.t, best=4)
#plot the best subtrees
par(mfrow=c(1, 2), cex=0.5)
plot(fit2.t)
title(main="Fit2 main tree")
text(fit2.t) # main tree
plot(fit2.p)
title(main="Fit2 best size 4 pruned")
text(fit2.p) # pruned tree
```

iii. fit3: bag two trees

    a) Take 2 bootstrap training samples and build two trees using the
    rpart(Income2005 ~., data.train.b, minsplit=20, cp=.009). Display both trees.

```{r bootstrap trees, echo=F}
# bootstrap tree 1
par(mfrow=c(1, 2), cex=0.5)
n=2067
set.seed(1)
index1 <- sample(n, n, replace = TRUE)
IQ.train2 <- IQ.train[index1, ]  # IQ.train2 here is a bootstrap sample
boot.1.single.full <- rpart(LogIncome ~., IQ.train2, minsplit=20, cp=.009)
plot(boot.1.single.full)
title(main = "First bootstrap tree")
text(boot.1.single.full, pretty=0)

# bootstrap tree 2
set.seed(2)
index1 <- sample(n, n, replace = TRUE)
IQ.train2 <- IQ.train[index1, ]  # IQ.train2 here is a bootstrap sample
boot.2.single.full <- rpart(LogIncome ~., IQ.train2, minsplit=20, cp=.009)
plot(boot.2.single.full)
title(main = "Second bootstrap tree")
text(boot.2.single.full, pretty=0)
par(mfrow=c(1,1), cex=0.5)
```

    b) Explain how to get fitted values for Michelle by bagging the two trees obtained above. Do not use the predict().

**We can get fitted values for Michelle by taking the average of fitted equations from the two trees, and her fitted value is 9.84.**

```{r fitted value bagged trees Michelle, echo=F}
fit3 <- (predict(boot.1.single.full, Michelle) + predict(boot.2.single.full, Michelle))/2
data.frame(fitted=fit3,  obsy=Michelle["LogIncome"])
```

    c) What is the testing error for the bagged tree. Is it guaranteed that the testing error by bagging the two tree always smaller that either single tree?

```{r testing error bagged tree}
test.error.fit3 <- mean((fit3 - Michelle$LogIncome)^2)
test.error.fit3
```

**The testing error for the bagged tree is 0.7647221, and it is not guaranteed that the testing error by bagging the two tree always smaller that either single tree because if the single trees already have low variance and are well-tuned, bagging would not offer much benefit and could even increase the testing error.**

iv. fit4: Build a best possible RandomForest

    a) Show the process how you tune mtry and number of trees. Give a very high level explanation how fit4 is built.

**To build fit4 using randomForest, we would first take B many bootstrap samples, then build a deep random tree for each sample using m as the parameter for number of randomly chosen predictors at each split. We want to choose B such that it settles the OOB testing errors, and it seems like we may need 250 trees according to Figure 3.1. Since we have 30 predictors for LogIncome, m ranges from 1 to 30, in which m=30 gives us bagging estimates of non-random trees. We would want to tune the parameter m using OOB testing errors. As seem from Figure 3.2, it seems like the best value for m is 10.**

```{r fit4}
set.seed(1)
#names(IQ) # we have 30 predictor values

# number of trees
fit.rf <- randomForest(LogIncome~., IQ.train, mtry=10, ntree=250) # after changing ntree
plot(fit.rf, col="red", pch=16, type="p",
     main="Figure 3.1: Testing error vs number of trees")

# testing error vs mtry plot
rf.error.p <- 1:30
for (p in 1:30)
{
  fit.rf <- randomForest(LogIncome~., IQ.train, mtry=p, ntree=250)
  #plot(fit.rf, col= p, lwd = 3)
  rf.error.p[p] <- fit.rf$mse[250]
}
rf.error.p # oob mse

plot(1:30, rf.error.p, pch=16,
     main = "Figure 3.2: Testing errors of mtry with 250 trees",
     xlab="mtry",
     ylab="OOB mse of mtry")
lines(1:30, rf.error.p)

# mtry = 9
fit4 <- randomForest(LogIncome~., IQ.train, mtry=10, ntree=250)
plot(fit4)
fit4$mse[250]
```

    b) Compare the oob errors from fit4 to the testing errors using your testing data. Are you convinced that oob errors estimate testing error reasonably well.

```{r oob errors vs testing errors, echo=F}
fit4.testing <- randomForest(LogIncome~., IQ.train,
                             xtest=IQ.test[, -31], ytest=IQ.test[,31], mtry=10, ntree=250)
plot(1:250, fit4.testing$mse, col="red", pch=16,
     xlab="number of trees",
     ylab="mse",
     main="Figure 3.3: fit4 mse's of RF: blue=oob errors, red=testing errors")
points(1:250, fit4$mse, col="blue", pch=16)
```
**As seem from Figure 3.3, the oob errors estimate testing errors pretty well.**

    c) What is the predicted value for Michelle?

```{r fit4 predicted value Michelle}
predict(fit4, Michelle)
```

**The predicted LogIncome value for Michelle is 9.81.**


v. Now you have built so many predicted models (fit1 through fit4 in this section). What about build a fit5 which bags fit1 through fit4. Does fit5 have the smallest testing error?

```{r fit5, echo=F}
fit5 <- (predict(fit1, Michelle) + predict(fit2, Michelle) + fit3 + predict(fit4, Michelle))/4
test.error.fit5 <- mean((fit5 - Michelle$LogIncome)^2)
test.error.fit5
```

**The testing error for fit5 is 0.7395131, which is not the smallest testing error as fit2 has the smallest testing error of 0.6823709 out of all of the models.**

vi.  Summarize the results and nail down one best possible final model you will recommend to predict income. Explain briefly why this is the best choice. Finally for the first time evaluate the prediction error using the validating data set.

```{r val data, echo=F}
val.error.fit2 <- mean((predict(fit2, IQ.val) - IQ.val$LogIncome)^2)
val.error.fit2
```

**The best possible final model is fit2, which is just one tree that uses all of the predictor variables. This is because we are predicting LogIncome on Michelle, who is just one person, so out of all of the models, fit2 just turns out to be the one that predicts the best for her. If we are predicting on a larger group of people, then it is more likely that the bagged trees will have a lower testing error because there are more people in the sample for the trees to predict on. The validation error for fit2 is 0.8504427.**

vii. Use your final model to predict Michelle's income.

```{r}
data.frame(fitted=fit5,  obsy=Michelle["LogIncome"])
```
**Fit5 predicts Michelle's income to be 9.85.**






