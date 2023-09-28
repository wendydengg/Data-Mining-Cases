# Overview 

Principle Component Analysis is widely used in data exploration, dimension reduction, data visualization. The aim is to transform original data into uncorrelated linear combinations of the original data while keeping the information contained in the data. High dimensional data tends to show clusters in lower dimensional view. 

Clustering Analysis is another form of EDA. Here we are hoping to group data points which are close to each other within the groups and far away between different groups. Clustering using PC's can be effective. Clustering analysis can be very subjective in the way we need to summarize the properties within each group. 

Both PCA and Clustering Analysis are so called unsupervised learning. There is no response variables involved in the process. 

For supervised learning, we try to find out how does a set of predictors relate to some response variable of the interest. Multiple regression is still by far, one of the most popular methods. We use a linear models as a working model for its simplicity and interpretability. It is important that we use domain knowledge as much as we can to determine the form of the response as well as the function format of the factors on the other hand. 


## Objectives

- PCA
- SVD
- Clustering Analysis
- Linear Regression

## Review materials

- Study Module 2: PCA
- Study Module 3: Clustering Analysis
- Study Module 4: Multiple regression

## Data needed

- `NLSY79.csv`
- `brca_subtype.csv`
- `brca_x_patient.csv`

# Case study 1: Self-eseteem 

Self-esteem generally describes a person's overall sense of self-worthiness and personal value. It can play significant role in one's motivation and success throughout the life. Factors that influence self-esteem can be inner thinking, health condition, age, life experiences etc. We will try to identify possible factors in our data that are related to the level of self-esteem. 

In the well-cited National Longitudinal Study of Youth (NLSY79), it follows about 13,000 individuals and numerous individual-year information has been gathered through surveys. The survey data is open to public [here](https://www.nlsinfo.org/investigator/). Among many variables we assembled a subset of variables including personal demographic variables in different years, household environment in 79, ASVAB test Scores in 81 and Self-Esteem scores in 81 and 87 respectively. 

The data is store in `NLSY79.csv`.



Here are the description of variables:

**Personal Demographic Variables**

* Gender: a factor with levels "female" and "male"
* Education05: years of education completed by 2005
* HeightFeet05, HeightInch05: height measurement. For example, a person of 5'10 will be recorded as HeightFeet05=5, HeightInch05=10.
* Weight05: weight in lbs.
* Income87, Income05: total annual income from wages and salary in 2005. 
* Job87 (missing), Job05: job type in 1987 and 2005, including Protective Service Occupations, Food Preparation and Serving Related Occupations, Cleaning and Building Service Occupations, Entertainment Attendants and Related Workers, Funeral Related Occupations, Personal Care and Service Workers, Sales and Related Workers, Office and Administrative Support Workers, Farming, Fishing and Forestry Occupations, Construction Trade and Extraction Workers, Installation, Maintenance and Repairs Workers, Production and Operating Workers, Food Preparation Occupations, Setters, Operators and Tenders,  Transportation and Material Moving Workers
 
 
**Household Environment**
 
* Imagazine: a variable taking on the value 1 if anyone in the respondent’s household regularly read magazines in 1979, otherwise 0
* Inewspaper: a variable taking on the value 1 if anyone in the respondent’s household regularly read newspapers in 1979, otherwise 0
* Ilibrary: a variable taking on the value 1 if anyone in the respondent’s household had a library card in 1979, otherwise 0
* MotherEd: mother’s years of education
* FatherEd: father’s years of education
* FamilyIncome78

**Variables Related to ASVAB test Scores in 1981**

Test | Description
--------- | ------------------------------------------------------
AFQT | percentile score on the AFQT intelligence test in 1981 
Coding | score on the Coding Speed test in 1981
Auto | score on the Automotive and Shop test in 1981
Mechanic | score on the Mechanic test in 1981
Elec | score on the Electronics Information test in 1981
Science | score on the General Science test in 1981
Math | score on the Math test in 1981
Arith | score on the Arithmetic Reasoning test in 1981
Word | score on the Word Knowledge Test in 1981
Parag | score on the Paragraph Comprehension test in 1981
Numer | score on the Numerical Operations test in 1981

**Self-Esteem test 81 and 87**

We have two sets of self-esteem test, one in 1981 and the other in 1987. Each set has same 10 questions. 
They are labeled as `Esteem81` and `Esteem87` respectively followed by the question number.
For example, `Esteem81_1` is Esteem question 1 in 81.

The following 10 questions are answered as 1: strongly agree, 2: agree, 3: disagree, 4: strongly disagree

* Esteem 1: “I am a person of worth”
* Esteem 2: “I have a number of good qualities”
* Esteem 3: “I am inclined to feel like a failure”
* Esteem 4: “I do things as well as others”
* Esteem 5: “I do not have much to be proud of”
* Esteem 6: “I take a positive attitude towards myself and others”
* Esteem 7: “I am satisfied with myself”
* Esteem 8: “I wish I could have more respect for myself”
* Esteem 9: “I feel useless at times”
* Esteem 10: “I think I am no good at all”

## Data preparation

Load the data. Do a quick EDA to get familiar with the data set. Pay attention to the unit of each variable. Are there any missing values? 

```{r EDA}
self_esteem <- read.csv("Data/NLSY79.csv")

#check dimension
dim(self_esteem)

#check variables
names(self_esteem)

#check data structure
str(self_esteem)

#check missing data
sapply(self_esteem, function(x) sum(is.na(x)))
sum(self_esteem$Income87 == 0)
sum(self_esteem$Income05 == 0)
sum(self_esteem$FamilyIncome78 == 0)
```
**There are 2431 people and 46 variables in the self-esteem data. Some values of income and family income are missing**

## Self esteem evaluation

Let concentrate on Esteem scores evaluated in 87. 

0. First do a quick summary over all the `Esteem` variables. Pay attention to missing values, any peculiar numbers etc. How do you fix problems discovered if there is any? Briefly describe what you have done for the data preparation. 

```{r get Esteem scores evaluated in 87}
library(dplyr)
#select Esteem scores in 87
data_esteem <- self_esteem %>%
  dplyr::select(paste0("Esteem87_",c(1:10)))

#data summary
summary(data_esteem)
sum(is.na(data_esteem))
```
- **There are no missing values or peculiar numbers in Esteem scores.**
- **If there are missing values, we should first check the reason for missing data. If the values are missing at random, and the number of missing values is small, we can delete the observations directly. However, if the values are missing not at random, deleting observations can cause bias. We can also consider using imputation to generate values for missing values, for example, we can use the mean to replace the missing values.**
- **If there are peculiar numbers, we should first check if they are data entry errors. If not, we can use data transformation to eliminate the outliers.**

1. Reverse Esteem 1, 2, 4, 6, and 7 so that a higher score corresponds to higher self-esteem. (Hint: if we store the esteem data in `data.esteem`, then `data.esteem[,  c(1, 2, 4, 6, 7)]  <- 5 - data.esteem[,  c(1, 2, 4, 6, 7)]` to reverse the score.)
```{r Reverse}
data_esteem[, c(1, 2, 4, 6, 7)] <- 5 - data_esteem[, c(1, 2, 4, 6, 7)]
head(data_esteem)
```


2. Write a brief summary with necessary plots about the 10 esteem measurements.
```{r 10 esteem measurement}
library(ggplot2)
library(tidyverse)
#bar plots
data_esteem1 <- data_esteem %>%
  mutate(across(everything(), as.character))

x <- as.data.frame(t(data_esteem1))
x <- tibble::rownames_to_column(x,"measurement")

x %>% 
  pivot_longer(cols = contains("V"),
               names_to = "subject",
               values_to = "score") %>%
  ggplot(aes(x = score)) +
  geom_bar()+
  facet_wrap( ~ measurement) +
  labs(title = "Figure 1.1:",
       subtitle = "Histogram of 10 esteem measurements")
```

**Since we reverse the order in 1.2.1, higher score corresponds to higher self-esteem. We can see the scores are high in the measurements associated with positive attitude from the bar plots. For the measurements associated with negative attitude, the scores are also mainly 3 or 4, indicating most of individuals disagree with the negative measurements.**

3. Do esteem scores all positively correlated? Report the pairwise correlation table and write a brief summary.
```{r correlation, warning=FALSE}
library(kableExtra)
library(magrittr)
cor_esteem <- cor(data_esteem)
knitr::opts_chunk$set(echo = TRUE)
knitr::kable(cor_esteem, format = "latex", booktabs = TRUE) %>%
  kable_styling(latex_options = "scale_down") %>%
  kable_styling(latex_options = "HOLD_position")
  
```
**Esteem scores are all positively correlated after reversing the order in part 1.2.1. Esteem 1, 2, 4, 6, and 7 indicate positive attitude, therefore, they are more closely correlated.**

4. PCA on 10 esteem measurements. (centered but no scaling)

    a) Report the PC1 and PC2 loadings. Are they unit vectors? Are they orthogonal? 
```{r PC1 and PC2 loadings}
#PC1 and PC2 loadings
pc <- prcomp(data_esteem, scale = F)
pc12 <- pc$rotation[,1:2]
knitr::kable(pc12) %>%
  kable_styling(latex_options = "HOLD_position")

#check the loadings are unit 1
colSums((pc12)^2)

#check orthogonality
round(cor(pc12),2)

```
- **The PC1 and PC2 loadings are shown in the table.**
- **The PC1 and PC2 loadings are unit vectors, but they are not orthogonal.**

    b) Are there good interpretations for PC1 and PC2? (If loadings are all negative, take the positive loadings for the ease of interpretation)
    
  **Interpretations for PC1: The values of ten loadings are approximately proportional to the sum of esteem scores. The absolute values are approximately around 0.3. Higher PC1 indicates higher self-esteem**
**Interpretations for PC2: The values and signs of ten loadings can be interpreted as the difference between sum of esteem scores indicating strongly negative attitude and sum of esteem scores indicating slightly negative/positive attitude.**

    c) How is the PC1 score obtained for each subject? Write down the formula.
```{r pc1 score}
#calculate by hand
library(geometry)
pc$rotation[,1] #PC1 loading
scale(data_esteem,scale = F)[1,] #centered original score
pc1_score = dot(pc$rotation[,1],scale(data_esteem,scale = F)[1,])
pc1_score

#compare result
round(pc1_score,3) == round(pc$x[1,1],3) #PC1 score for first ind
```
**Take the first observation as an example, to get PC1 score:** $$PC1 = 0.235*centered\_Esteem87\_1+0.244*centered\_Esteem87\_1_...+0.376*centered\_Esteem87\_10$$

    d) Are PC1 scores and PC2 scores in the data uncorrelated? 
```{r PC1 scores and PC2 scores}
round(cor(pc$x),2)[1:2,1:2]
```
**PC1 and PC2 scores are uncorrelated.**

    e) Plot PVE (Proportion of Variance Explained) and summarize the plot. 
```{r PVE}
#plot PVE
plot(summary(pc)$importance[2,],
     xlab = "Number of PCs",
     ylab = "PVE",
     main = "Figure 1.2: PVE plot")
```

**The plot indicates 4 leading PCs should be enough. Among those PCs, first PC can explain most proportion of the variance.**

    f) Also plot CPVE (Cumulative Proportion of Variance Explained). What proportion of the variance in the data is explained by the first two principal components?
```{r CPVE}
#plot CPVE
plot(summary(pc)$importance[3,],
     xlab = "Number of PCs",
     ylab = "CPVE",
     main = "Figure 1.3: CPVE plot")

#summary
summary(pc)
```

**First two principal components explained 59.3% variance in the original data.**
   
    g) PC’s provide us with a low dimensional view of the self-esteem scores. Use a biplot with the first two PC's to display the data.  Give an interpretation of PC1 and PC2 from the plot. (try `ggbiplot` if you could, much prettier!)
    
```{r install ggbiplot, include=FALSE}
install.packages("htmltools")

devtools::install_github("vqv/ggbiplot")
```

```{r biplot, warning=FALSE, message=FALSE}
library(ggbiplot)
ggbiplot(pc, obs.scale = 1, var.scale = 1)+
  geom_hline(yintercept = 0,color = "blue")+
  geom_vline(xintercept = 0, color = "blue")+
  labs(title = "Figure 1.4:", subtitle = "Biplot")+
  theme_bw()
```
**Interpretation: All esteem measurements are positively associated with PC1. Esteem 8,9 and 10 are positively associated with PC2. **

5. Apply k-means to cluster subjects on the original esteem scores

    a) Find a reasonable number of clusters using within sum of squared with elbow rules.
```{r number of clusters}
library(factoextra)
set.seed(83)
fviz_nbclust(data_esteem, kmeans, method = "wss")+
  labs(title = "Figure 1.5:", subtitle = "Optimal number of clusters")
```

**Two clusters are a reasonable number based on the elbow rules.**

    b) Can you summarize common features within each cluster?
```{r clusters}
#k means
esteem_kmeans <- kmeans(data_esteem, centers = 2)
str(esteem_kmeans)
```

**We do not know any info from clustering results since it is unsupervised. We do not know the label of samples.**

    c) Can you visualize the clusters with somewhat clear boundaries? You may try different pairs of variables and different PC pairs of the esteem scores.
```{r visualize the clusters}
#get other variables
esteem <- data_esteem %>%
  mutate(cluster = as.factor(esteem_kmeans$cluster)) %>%
  arrange(cluster)

#try positive attitude esteem scores
ggplot(data = esteem, aes(x = Esteem87_1, y = Esteem87_2, col = cluster))+
  geom_point()+
  ggtitle("Clustering over positive attitude esteem scores") #categorical variables

#try pca
data.frame(pc1 = pc$x[,1],
           pc2 = pc$x[,2],
           cluster = as.factor(esteem_kmeans$cluster)) %>%
ggplot(aes(x = pc1, y = pc2, col = cluster))+
  geom_point()+
  labs(title = "Figure 1.6:", subtitle = "Clustering over PCs")
```

**Because esteem scores are categorical, clusters can not be visualized with clear boundaries. By using PCs, we can get clear boundaries of two groups.**


6. We now try to find out what factors are related to self-esteem? PC1 of all the Esteem scores is a good variable to summarize one's esteem scores. We take PC1 as our response variable. 

    a) Prepare possible factors/variables:
    
      - EDA the data set first. 
```{r EDA data}
#check data structure
str(self_esteem)

#summary table
self_esteem %>%
  dplyr::select(-paste0("Esteem81_", c(1:10)),-paste0("Esteem87_", c(1:10))) %>%
  summary(self_esteem)
```

**Check data structure and summary table first.**   

      - Personal information: gender, education (05), log(income) in 87, job type in 87. Weight05 (lb) and HeightFeet05 together with Heightinch05. One way to summarize one's weight and height is via Body Mass Index which is defined as the body mass divided by the square of the body height, and is universally expressed in units of kg/m². Note, you need to create BMI first. Then may include it as one possible predictor. 
```{r Personal information}
personal_info <- self_esteem %>%
  filter(Income87 != 0,
         Income05 != 0,
         FamilyIncome78 != 0) %>%
  mutate(log_income = log(Income87),
         Weight_kg = 0.45*Weight05, #convert to kg
         height_m = 0.3048*HeightFeet05 + 0.0254*HeightInch05, #convert to m
         BMI = Weight_kg/(height_m^2)) %>%
  dplyr::select(Subject, Gender, Education05, log_income, BMI)


#remove NaNs
not_null_val <- which(!is.na(personal_info$log_income))
personal_info <- personal_info %>%
  filter(log_income != "NaN")
```
          
      - Household environment: Imagazine, Inewspaper, Ilibrary, MotherEd, FatherEd, FamilyIncome78. Do set indicators `Imagazine`, `Inewspaper` and `Ilibrary` as factors. 
```{r Household environment}
h_env <- self_esteem %>%
  mutate(Imagazine = as.factor(Imagazine),
         Inewspaper = as.factor(Inewspaper),
         Ilibrary = as.factor(Ilibrary)) %>%
  dplyr::select(Subject, Imagazine, Inewspaper, Ilibrary, MotherEd, FatherEd, FamilyIncome78)
```
     
      - You may use PC1 of ASVAB as level of intelligence
```{r Final data}
#PC1 of esteem score
pc1_esteem <- pc$x[,1][not_null_val]

#PC1 of ASVAB
ASVAB <- self_esteem[,16:26]
pc_ASVAB <- prcomp(ASVAB, scale = F)
pc1_ASVAB <- pc_ASVAB$x[,1][not_null_val]

#Final data
esteem_final <- personal_info %>%
  inner_join(h_env , by = c("Subject" = "Subject")) %>%
  mutate(Intelligence = pc1_ASVAB,
         Esteem = pc1_esteem)
head(esteem_final)
```
        
    b)   Run a few regression models between PC1 of all the esteem scores and suitable variables listed in a). Find a final best model with your own criterion. 

      - How did you land this model? Run a model diagnosis to see if the linear model assumptions are reasonably met. 
        
      - Write a summary of your findings. In particular, explain what and how the variables in the model affect one's self-esteem. 
      
```{r regression models, echo=FALSE}
#Backward elimination
fit <- lm(Esteem ~ Gender+Education05+log_income + BMI+Imagazine+ Inewspaper+ Ilibrary+ MotherEd+ FatherEd+ FamilyIncome78+Intelligence, data = esteem_final)
library(MASS)
stepAIC(fit, direction = "backward")
```        

```{r model diagnosis}
#Final Model
fit_final <- lm(formula = Esteem ~ Intelligence, data = esteem_final)
summary(fit_final)
plot(fit_final, sub = "Figure 1.7", cex.main = 0.5)
```

**To find the final model, we first need to consider the type of regression model. Since we take PC1 as our response variable, which is continuous, we should use linear regression. To select variables in the model, I used stepwise variable selection algorithms. More specifically, I included all variables in a complex model.(Interaction terms were not considered here.) Backward elimination begins with this complex model and sequentially removes terms. To select an optimal model, AIC is used as criteria. Because of the bias/variance trade off, It is not automatically best to choose the model with more parameters. The AIC penalizes a model for each additional parameter. The final model should result in a lowest AIC. Eventually, the final model with lowest AIC only includes Intelligence. Based on the plots, we can see linear model assumptions are reasonably met. The coeffient of Intelligence is 0.011649. It indicates Intelligence is positively related with Esteem score. Esteem score is expected to increase 0.012 for every unit increase in Intelligence score.** 

# Case study 2: Breast cancer sub-type


[The Cancer Genome Atlas (TCGA)](https://www.cancer.gov/about-nci/organization/ccg/research/structural-genomics/tcga), a landmark cancer genomics program by National Cancer Institute (NCI), molecularly characterized over 20,000 primary cancer and matched normal samples spanning 33 cancer types. The genome data is open to public from the [Genomic Data Commons Data Portal (GDC)](https://portal.gdc.cancer.gov/).
 
In this study, we focus on 4 sub-types of breast cancer (BRCA): basal-like (basal), Luminal A-like (lumA), Luminal B-like (lumB), HER2-enriched. The sub-type is based on PAM50, a clinical-grade luminal-basal classifier. 

* Luminal A cancers are low-grade, tend to grow slowly and have the best prognosis.
* Luminal B cancers generally grow slightly faster than luminal A cancers and their prognosis is slightly worse.
* HER2-enriched cancers tend to grow faster than luminal cancers and can have a worse prognosis, but they are often successfully treated with targeted therapies aimed at the HER2 protein. 
* Basal-like breast cancers or triple negative breast cancers do not have the three receptors that the other sub-types have so have fewer treatment options.

We will try to use mRNA expression data alone without the labels to classify 4 sub-types. Classification without labels or prediction without outcomes is called unsupervised learning. We will use K-means and spectrum clustering to cluster the mRNA data and see whether the sub-type can be separated through mRNA data.

We first read the data using `data.table::fread()` which is a faster way to read in big data than `read.csv()`. 

```{r}
brca <- fread("data/brca_subtype.csv")

# get the sub-type information
brca_subtype <- brca$BRCA_Subtype_PAM50
brca <- brca[,-1]
```

1. Summary and transformation

    a) How many patients are there in each sub-type? 
    
```{r}
brca <- fread("data/brca_subtype.csv")

dim(brca)
names(brca)[1:20]
table(brca$BRCA_Subtype_PAM50)
brca$BRCA_Subtype_PAM50 <- as.factor(brca$BRCA_Subtype_PAM50)

# get the sub-type information
brca_subtype <- brca$BRCA_Subtype_PAM50
brca <- brca[,-1]
```
     **Basal: 208**
     **Her2: 91**
     **LumA: 628**
     **LumB: 233**

    b) Randomly pick 5 genes and plot the histogram by each sub-type.

```{r, echo=TRUE, warning=FALSE, message=FALSE}
num_gene <- ncol(brca)

# randomly select 5 gene
set.seed(5)
sample_idx <- sample(num_gene, 5)  

# plot count number histogram for each gene
brca %>%
  dplyr::select(all_of(sample_idx)) %>%      # select column by index
  pivot_longer(cols = everything()) %>%     # for facet(0)
  ggplot(aes(x = value, y = ..density..)) +
  geom_histogram(aes(fill = name)) +
  facet_wrap(~name, scales = "free") +
  theme_bw() +
  theme(legend.position = "none") +
  labs(title = "Figure 2.1:",
       subtitle = "Histogram of 5 Random Genes")
  
```

    c) Remove gene with zero count and no variability. Then apply logarithmic transform.
    
```{r}
# remove genes with 0 counts
sel_cols <- which(colSums(abs(brca)) != 0)
brca_sub <- brca[, sel_cols, with=F]
dim(brca_sub) 

# log
brca_sub <- log2(as.matrix(brca_sub+1e-10)) 
```

2. Apply kmeans on the transformed dataset with 4 centers and output the discrepancy table between the real sub-type `brca_subtype` and the cluster labels.

```{r}
brca_sub_kmeans <- kmeans(x = brca_sub, 4)
table(brca_subtype, brca_sub_kmeans$cluster)
```

3. Spectrum clustering: to scale or not to scale?

    a) Apply PCA on the centered and scaled dataset. How many PCs should we use and why? You are encouraged to use `irlba::irlba()`.
    
```{r}
# center and scale the data
brca_sub_scaled_centered <- scale(as.matrix(brca_sub), center = T, scale = T)
svd_ret <- irlba::irlba(brca_sub_scaled_centered, nv = 10)

# Approximate the PVE
svd_var <- svd_ret$d^2/(nrow(brca_sub_scaled_centered)-1)
pve_apx <- svd_var/num_gene
plot(pve_apx, type="b", pch = 19, frame = FALSE, main = "Figure 2.2: PVE plot")
```

**With the elbow rule on Figure 1.2, we should use 4 PCs.**
    
    b) Plot PC1 vs PC2 of the centered and scaled data and PC1 vs PC2 of the centered but unscaled data side by side. Should we scale or not scale for clustering process? Why? (Hint: to put plots side by side, use `gridExtra::grid.arrange()` or `ggpubr::ggrrange()` or `egg::ggrrange()` for ggplots; use `fig.show="hold"` as chunk option for base plots)
    
```{r, echo=FALSE, results='hide'}
pca_scaled <- prcomp(brca_sub, center = T, scale. = T)
pca_unscaled <- prcomp(brca_sub, center = T, scale. = F)

pca_scaled$rotation[, 1:2]
pca_unscaled$rotation[, 1:2]
```

```{r}
p1 <- data.frame(
          pc1 = pca_scaled$x[, 1], 
          pc2 = pca_scaled$x[, 2],
          group = as.factor(brca_sub_kmeans$cluster)) %>%
  ggplot(aes(x = pc1, y = pc2, col = group)) +
  geom_point() +
  labs(title = "Figure 2.3:",
       subtitle = "Clustering over PC1 and PC2 of Scaled Data")

p2 <- data.frame(
          pc1 = pca_unscaled$x[, 1], 
          pc2 = pca_unscaled$x[, 2],
          group = as.factor(brca_sub_kmeans$cluster)) %>%
  ggplot(aes(x = pc1, y = pc2, col = group)) +
  geom_point() +
  labs(title = "Figure 2.4:",
       subtitle = "Clustering over PC1 and PC2 of Unscaled Data")

gridExtra::grid.arrange(p1, p2)
```

**We should use unscaled data because the clusters in Figure 1.4 are more distinct compared to the clusters in Figure 1.3.**

4. Spectrum clustering: center but do not scale the data

    a) Use the first 4 PCs of the centered and unscaled data and apply kmeans. Find a reasonable number of clusters using within sum of squared with the elbow rule.

```{r}
# using plot 
plot(pca_unscaled)

# using wss
set.seed(0)

# function to compute total within-cluster sum of square 
wss <- function(df, k) {
  kmeans(df, k, nstart = 10)$tot.withinss
}

k.values <- 2:15

wss_values <- sapply(k.values, function(k) kmeans(brca_sub, centers = k)$tot.withinss)

plot(k.values, wss_values,
       type="b", pch = 19, frame = FALSE, 
       xlab="Number of clusters K",
       ylab="Total within-clusters sum of squares",
       main = "Figure 2.5: WSS plot")

pca_unscaled_kmeans <- kmeans(pca_unscaled$x[, 1:4], centers = 4)
```
    
    b) Choose an optimal cluster number and apply kmeans. Compare the real sub-type and the clustering label as follows: Plot scatter plot of PC1 vs PC2. Use point color to indicate the true cancer type and point shape to indicate the clustering label. Plot the kmeans centroids with black dots. Summarize how good is clustering results compared to the real sub-type.
    
```{r}
# color indicates the true cancer type
# shape indicates the cluster results
p3 <- data.table(x = pca_unscaled$x[,1], 
                 y = pca_unscaled$x[,2], 
                 col = as.factor(brca_subtype), 
                 cl = as.factor(pca_unscaled_kmeans$cluster), 
                 centers_x = pca_unscaled_kmeans$centers[,1], 
                 centers_y = pca_unscaled_kmeans$centers[,2])  %>%
  ggplot() +
  geom_point(aes(x = x, y = y, col = col, shape = cl)) +
  scale_color_manual(labels = c("Basal", "Her", "LumA", "LumB"),
                     values = scales::hue_pal()(4)) +
  scale_shape_manual(labels = c("Clulster 1", "Cluster 2", "Cluster 3", "Cluster 4"),
                     values = c(4, 16, 12, 11)) +
  geom_point(aes(x = centers_x,
                 y = centers_y, size = 1)) +
  theme_bw() +
  labs(color = "Cancer type", shape = "Cluster", size = "Centroid") +
  xlab("PC1") +
  ylab("PC2") + 
  labs(title = "Figure 1.6:",
       subtitle = "Clustering over PC1 and PC2")
p3
```

**The clustering results are not as good when using kmeans because some clusters, especially cluster 3 in Figure 1.6, contain all of the cancer sub-types, which is not the optimal result for clustering analysis since we want the cluster to group distinct elements together.**
    
    c) Compare the clustering result from applying kmeans to the original data and the clustering result from applying kmeans to 4 PCs. Does PCA help in kmeans clustering? What might be the reasons if PCA helps?
    
```{r}
p4 <- data.frame(x = pca_unscaled$x[,1],
                 y = pca_unscaled$x[,2],
                 col = as.factor(brca_sub_kmeans$cluster),
                 cl = as.factor(pca_unscaled_kmeans$cluster)) %>%
  ggplot() +
  geom_point(aes(x = x, y = y, col = col, shape = cl)) +
  scale_color_manual(labels = c("Cluster 1 (full)", "Cluster 2 (full)", "Cluster 3 (full)", "Cluster 4 (full)"),
                     values = scales::hue_pal()(4)) +
  scale_shape_manual(labels = c("PC cluster 1", "PC cluster 2", "PC cluster 3", "PC cluster 4"),
                     values = c(4, 16, 12, 11)) +
  theme_bw() +
  labs(color = "Cancer type", shape = "Cluster") +
  xlab("PC1") +
  ylab("PC2") + 
  labs(title = "Figure 1.7:",
       subtitle = "Clustering over Original Data")
p4
```

**PCA does help in kmeans clustering because the clustering results are pretty much the same when kmeans are applied to the original data versus to the 4 PCs, which is likely because we can narrow down the causal genes for the 4 cancer subtypes to 4 main genes and clusters.**
    
    d) Now we have an x patient with breast cancer but with unknown sub-type. We have this patient's mRNA sequencing data. Project this x patient to the space of PC1 and PC2. (Hint: remember we remove some gene with no counts or no variablity, take log and centered) Plot this patient in the plot in iv) with a black dot. Calculate the Euclidean distance between this patient and each of centroid of the cluster. Can you tell which sub-type this patient might have? 
    
```{r}
x_patient <- fread("data/brca_x_patient.csv")
x_patient <- log2(as.matrix(x_patient+1e-10)) 
x_patient <- data.frame(x_patient)
x_patient_sub <- data.frame()

i = 1
while (i <= length(colnames(brca_sub))) {
  if (colnames(brca_sub)[i] %in% colnames(x_patient) == TRUE) {
    x_patient_sub[1,i] = x_patient[,colnames(x_patient)[i]] #if same gene as brca_sub, append gene name and gene counts directly
    colnames(x_patient_sub)[i] = colnames(x_patient)[i]
  } else {
    x_patient_sub[1,i] = 0
    colnames(x_patient_sub)[i] = colnames(brca_sub)[i] #if x_patient info does not contain same gene as brca_sub, append gene name and set counts = 0
  }
  i = i+1
}

pc_score_x <- scale(x_patient_sub, pca_unscaled$center, pca_unscaled$scale) %*% pca_unscaled$rotation

p5 <- data.table(x = pca_unscaled$x[,1], 
                 y = pca_unscaled$x[,2], 
                 col = as.factor(brca_subtype), 
                 cl = as.factor(pca_unscaled_kmeans$cluster), 
                 x_pc = pc_score_x[,1], 
                 y_px = pc_score_x[,2])  %>%
  ggplot() +
  geom_point(aes(x = x, y = y, col = col, shape = cl)) +
  scale_color_manual(labels = c("Basal", "Her", "LumA", "LumB"),
                     values = scales::hue_pal()(4)) +
  scale_shape_manual(labels = c("Clulster 1", "Cluster 2", "Cluster 3", "Cluster 4"),
                     values = c(4, 16, 12, 11)) +
  geom_point(aes(x = x_pc,
                 y = y_px, size = 1)) +
  geom_text(aes(x = x_pc,
                 y = y_px, label = "Patient X"), hjust=0, vjust=0) +
  theme_bw() +
  labs(color = "Cancer type", shape = "Cluster", size = "Patient X") +
  xlab("PC1") +
  ylab("PC2") +
  labs(title = "Figure 1.8:",
       subtitle = "Clustering over PC1 and PC2 with Patient X")
p5

# calculate distance
x <- c(pc_score_x[,1], pc_score_x[,2])
c1 <- c(pca_unscaled_kmeans$centers[,1][1], pca_unscaled_kmeans$centers[,2][1])
c2 <- c(pca_unscaled_kmeans$centers[,1][2], pca_unscaled_kmeans$centers[,2][2])
c3 <- c(pca_unscaled_kmeans$centers[,1][3], pca_unscaled_kmeans$centers[,2][3])
c4 <- c(pca_unscaled_kmeans$centers[,1][4], pca_unscaled_kmeans$centers[,2][4])

euclidean_dist <- function(x, y) sqrt(sum((x - y)^2))
euclidean_dist(x, c1)
euclidean_dist(x, c2)
euclidean_dist(x, c3)
euclidean_dist(x, c4)
```
**According to Figure 1.8 and the Euclidean distances calculated between Patient X and each of the four centroids, Patient X is closest to the fourth centroid, whose cluster contains all of the cancer subtypes. Thus, we cannot tell which cancer subtype this patient has.**

# Case study 3: Auto data set

This question utilizes the `Auto` dataset from ISLR. The original dataset contains 408 observations about cars. It is similar to the CARS dataset that we use in our lectures. To get the data, first install the package ISLR. The `Auto` dataset should be loaded automatically. We'll use this dataset to practice the methods learn so far. 
Original data source is here: https://archive.ics.uci.edu/ml/datasets/auto+mpg

Get familiar with this dataset first. Tip: you can use the command `?ISLR::Auto` to view a description of the dataset. 

## EDA
Explore the data, with particular focus on pairwise plots and summary statistics. Briefly summarize your findings and any peculiarities in the data.

**The auto dataset contains 392 observations of nine variables describing 301 different car models with model years ranging from 1970-1982. The region of origin with the greatest number of unique car models is America in all years except for 1980, when the greatest number of unique models came from Japan (Figure 3.1). Across all years, American cars had a lower average miles per gallon (MPG) than both European and Japanese cars, although MPG generally increased over time for all origins (Figure 3.2). While there appears to be a fair amount of variation, heavier cars generally have a shorter time to acceleration from 0 to 60 miles per hour. Heavier cars also usually have a greater number of cylinders (Figure 3.3). Cars with higher horsepower tend to get lower miles per gallon (Figure 3.4).**

```{r auto descriptives}
#Load auto data and get summary statistics
rm(list = ls())
data("Auto")

str(Auto)
summary(Auto)
colSums(is.na(Auto))
n_distinct(Auto$name)

#Origin should be treated as a factor: 1=American, 2=European, 3=Japanese. 
Auto$origin <- factor(Auto$origin, levels = c("1", "2", "3"), labels = c("American", "European", "Japanese"))

```


```{r auto descriptive graphs}
#Look at number of vehicles by region over time
Auto_names <- Auto %>%
  group_by(origin, year) %>%
  mutate(num=n_distinct(name)) %>%
  distinct(year, origin, .keep_all = TRUE)

number_time <-  ggplot(data = Auto_names, aes(x=year, y=num, fill=origin)) +
  geom_bar(stat = "identity") +
  labs(y="Number of unique models", x="Year", title="Figure 3.1:", subtitle="Number of unique car models over time by region of origin") +
  theme_bw()
number_time

#Look at average MPH over time by origin
Auto <- Auto %>%
  group_by(origin, year) %>%
  mutate(avg_mpg = mean(mpg))

mpg_time <- ggplot(Auto, aes(x=year, y=avg_mpg, color=origin)) + 
  geom_point() + geom_line() +
  labs(y="Average MPG", x="Year", title="Figure 3.2:", subtitle="Average MPG over time by region of origin") +
  scale_x_continuous(n.breaks = 13) + 
  theme_bw()
mpg_time

#Look at weight versus acceleration
weight_accel <- ggplot(Auto, aes(x=weight, y=acceleration, color=cylinders)) +
  geom_point() + 
  geom_smooth() +
  labs(y="Time to accelerate from 0 to 60 mph (seconds)", x="Vehicle weight (pounds)", title="Figure 3.3:", subtitle="Relationship between weight of car and acceleration, by number of cylinders") +
  theme_bw()
weight_accel

#Look at weight versus acceleration
mpg_horse <- ggplot(Auto, aes(x=horsepower, y=mpg)) +
  geom_point() + 
  geom_smooth() +
  labs(y="Miles per gallon", x="Horsepower", title="Figure 3.4:", subtitle="Relationship between horsepower and miles per gallon") +
  theme_bw()
mpg_horse

```


## What effect does `time` have on `MPG`?

a) Start with a simple regression of `mpg` vs. `year` and report R's `summary` output. Is `year` a significant variable at the .05 level? State what effect `year` has on `mpg`, if any, according to this model. 
**Year is significantly related to MPG at the 0.05 level (Table 3.1). Every 1-year increase in time is associated with an average MPG increase of 1.23 (p<0.001).**
```{r linear regression mpg year, warning=FALSE}
regress_mpgyr <- glm(data=Auto, mpg ~ year, family=gaussian)
summary(regress_mpgyr)
confint(regress_mpgyr)

stargazer(regress_mpgyr, type = "text", title = "Table 3.1: Model Regressing Year on MPG", ci = TRUE)
```


b) Add `horsepower` on top of the variable `year` to your linear model. Is `year` still a significant variable at the .05 level? Give a precise interpretation of the `year`'s effect found here. 
**After controlling for horsepower, year is still significantly related to mpg at the 0.05 level (Table 3.2). Holding horsepower constant, every 1-year increase in time is associated with an average 0.66 increase in mpg (p<0.001).**
```{r regression mgp year horsepower, warning=FALSE}
regress_mpgyrhorse <- glm(data=Auto, mpg ~ year + horsepower, family=gaussian)
summary(regress_mpgyrhorse)
confint(regress_mpgyrhorse)

stargazer(regress_mpgyrhorse, type = "text", title = "Table 3.2: Model Regressing Year and Horsepower on MPG", ci = TRUE)
```

c) The two 95% CI's for the coefficient of year differ among (i) and (ii). How would you explain the difference to a non-statistician?
**Regression models only account for the variables that are included in the model, and the effect estimates and confidence intervals are dependent on what other variables have been included in the model. The first model included year alone, which has a different effect on MPG than year after accounting for horsepower.**

d) Create a model with interaction by fitting `lm(mpg ~ year * horsepower)`. Is the interaction effect significant at .05 level? Explain the year effect (if any). 
**The interaction between year and horsepower is significant at the 0.05 level (Table 3.3). When horsepower is 0, every 1-year increase in time is associated with an average 2.19 increase in mpg (p<0.001). This interpretation is nonsensical in this context, as a car could not have a horsepower of 0. In this case, it would be better to center the variables, which would allow us to interpret the effect of year when horsepower is equal to the sample average. The interaction coefficient tells us the difference in the effect of time (measured in years) for each unit increase in horsepower.**
```{r regression w/ interaction, warning=FALSE}
regress_interaction <- glm(data=Auto, mpg ~ year * horsepower, family=gaussian)
summary(regress_interaction)

stargazer(regress_interaction, type = "text", title = "Table 3.3: Regression Model with a Year*Horsepower Interaction", ci = TRUE)
```


## Categorical predictors

Remember that the same variable can play different roles! Take a quick look at the variable `cylinders`, and try to use this variable in the following analyses wisely. We all agree that a larger number of cylinders will lower mpg. However, we can interpret `cylinders` as either a continuous (numeric) variable or a categorical variable.

a) Fit a model that treats `cylinders` as a continuous/numeric variable. Is `cylinders` significant at the 0.01 level? What effect does `cylinders` play in this model?
**Treating cylinders as a continuous variable, it is significant at the 0.01 level (Table 3.4). For each 1-cylinder increase in the number of cylinders, mpg decreases by 3.56 on average (p<0.001).**
```{r regression cylinder continuous, warning=FALSE}
regression_cylinder_cont <- lm(data = Auto, mpg ~ cylinders)
summary(regression_cylinder_cont)

stargazer(regression_cylinder_cont, type = "text", title = "Table 3.4: Regression Model Treating Cylinders as Continuous", ci = TRUE)
```

b) Fit a model that treats `cylinders` as a categorical/factor. Is `cylinders` significant at the .01 level? What is the effect of `cylinders` in this model? Describe the `cylinders` effect over `mpg`. 
**Treating the number of cylinders as categorical, we compare the effect of the number of cylinders on mpg as compared to our reference, which in this case is 3 cylinders. Our overall test of significance, the F-test, tells us that the number of cylinders is significantly related to mpg at the 0.01 level (p<0.001) (Table 3.5). Cars with 4 cylinders have on average 8.73 higher mpg than cars with 3 cylinders (p<0.001). At the p=0.01 level, there is no significant difference in mpg between cars with 3 cylinders compared to cars with 5, 6, or 8 cylinders.**
```{r regression cylinder categorical, warning=FALSE}
#Creating a factor version of cylinders
Auto$cylinders_fact <- as.factor(Auto$cylinders)

#Running regression
regression_cylinder_cat <- lm(data = Auto, mpg ~ cylinders_fact)
summary(regression_cylinder_cat)

stargazer(regression_cylinder_cat, type = "text", title = "Table 3.5: Regression Model Treating Cylinders as Categorical", ci = TRUE)

```


c) What are the fundamental differences between treating `cylinders` as a continuous and categorical variable in your models? 
**A fundamental difference between continuous variables and categorical variables in regression models is that continuous variables contribute one beta to the model, whereas categorical variables contribute N-1 betas to the model, where N=the number of levels of your variable. When looking at the effect of a change in cylinders on MPG, we interpret the categorical beta as the effect of a 1-unit increase in the number of cylinders on MPG. In contrast, for categorical variables, we always compare one category to the reference category, which can be set at whatever value the researcher prefers.**

d) Can you test the null hypothesis: fit0: `mpg` is linear in `cylinders` vs. fit1: `mpg` relates to `cylinders` as a categorical variable at .01 level?  
**The model including cylinders as a categorical variable is significantly better than the model treating cylinders as continuous at the 0.01 level (p<0.001).**
```{r compare cylinder models}
anova(regression_cylinder_cont, regression_cylinder_cat)
```


## Results

Final modeling question: we want to explore the effects of each feature as best as possible. You may explore interactions, feature transformations, higher order terms, or other strategies within reason. The model(s) should be as parsimonious (simple) as possible unless the gain in accuracy is significant from your point of view.
```{r test models}
#First, try a model with all variables includes as predictors. This model has a fairly high R2 but isn't parsimonious, and has non-significant terms.
model1 <- lm(data=Auto, mpg ~ cylinders + displacement + horsepower + weight + acceleration + year + origin)
summary(model1)

#Data exploration showed that cars from different regions have different MPG, and that MPG increased over time. The rates of MPG change over time look slightly different by region - trying an interaction term.
model2 <- lm(data=Auto, mpg ~ cylinders + displacement + horsepower + weight + acceleration + year*origin)
summary(model2)

#We also saw in data exploration that as weight increases, the number of cylinders increases. Weight is significantly related to MPG - maybe there is an interaction effect
model3 <- lm(data=Auto, mpg ~ cylinders*weight + displacement + horsepower + acceleration + year*origin)
summary(model3)

#Remove remaining non-significant variables
model4 <- lm(data=Auto, mpg ~ cylinders*weight + displacement + horsepower + year*origin)
summary(model4)

#Displacement and horsepower are significant, but have a small absolute effect estimate on MPG - try removing them and comparing this with the previous model.
model5 <- lm(data=Auto, mpg ~ cylinders*weight + year*origin)
summary(model5)
anova(model5, model4)
#While the results show that the model including displacement and horsepower is "better", removing those because it only increased the R2 by 0.005 which doesn't seem worth the two extra variables.

```
  
a) Describe the final model. Include diagnostic plots with particular focus on the model residuals and diagnoses.
**Our final model uses weight, number of cylinders, year, and region of origin, plus the interaction between weight and cylinders and the interaction between year and origin to predict the square root of miles per gallon.** 

**To select these parameters, we began with all possible variables as predictors of MGP then added interaction terms based on the descriptive graphs we generated in exploratory analysis. From there, we removed non-significant terms and terms which were significant but didn't contribute substantially to the final MPG prediction (e.g., the effect estimate was very small). From there, we tested the assumptions of linearity, homoscedasticity, and normality. We found that the residuals were non-normal, and therefore tested transformations of our outcome of MPG. Although normality was improved with transformation of the outcome, it was still not ideal; for ease of interpretation, we therefore decided to proceed with a model using the non-transformed MPG outcome. Finally, we centered year, weight, and cylinders to allow for ease of interpretation individual coefficients.**
```{r final cars model diagnostics hidden}
final_model <- lm(data=Auto, mpg ~ weight*cylinders + year*origin)
summary(final_model)

#Model diagnostics
#Linearity and homoscedasticity
plot(final_model$fitted, final_model$residuals, 
     pch  = 16,
     main = "Residual plot")
abline(h=0, lwd=4, col="red")
#Linearity assumption is satisfied. Slight funnel-shaped pattern opening to the right but probably OK to assume this is homoscedastic.
#Normality
qqnorm(final_model$residuals)
qqline(final_model$residuals, lwd=4, col="blue")
#Normality definitely not met. Will try transforming the outcome (mpg) - although this does complicate interpretation.

Auto <- Auto %>% mutate(mpg_log = log(mpg), mpg_sqrt = sqrt(mpg), year_center = year-76, weight_center = weight-2978, cylinders_center = cylinders-5.47)
final_model_2 <- lm(data=Auto, mpg_log ~ weight*cylinders + year*origin)
summary(final_model_2)
qqnorm(final_model_2$residuals)
qqline(final_model_2$residuals, lwd=4, col="blue")

final_model_3 <- lm(data=Auto, mpg_sqrt ~ weight*cylinders + year*origin)
summary(final_model_3)
qqnorm(final_model_3$residuals)
qqline(final_model_3$residuals, lwd=4, col="blue")


```

```{r final cars model visible, echo = TRUE, warning = FALSE}
final_model <- lm(data=Auto, mpg ~ weight_center*cylinders_center + year_center*origin)
stargazer(final_model, type = "text", title = "Table 3.6: Model for Predicting MPG", ci = TRUE) 
#Linearity and homoscedasticity
plot(final_model$fitted, final_model$residuals, 
     pch  = 16,
     main = "Figure 3.5: Residual plot",
     xlab = "Fitted Values", 
     ylab = "Residuals")
abline(h=0, lwd=4, col="red")
#Normality
qqnorm(final_model$residuals,
       main = "Figure 3.6: Q-Q Plot")
qqline(final_model$residuals, lwd=4, col="blue")

```

b) Summarize the effects found.
**The results of our final model can be found in Table 3.6. Holding all other variables constant, a 1-pound increase in weight is associated with an average 0.007 decrease in MPG (95% CI: -0.008 - -0.006) when the number of cylinders is 5.47 (the sample average). Each one-year increase is associated with a 0.649 increase in MPG for American-made cars, holding all other factors constant (95% CI: 0.534-0.764). Compared to American-made cars, European-made cars have 1.26 greater MPG when the year is 1976 and all other factors are held constant. The difference in average MPG between European- versus American-made cars for every 1 year increase is 0.553 (95% CI: 0.313-0.793). The difference in average MPG between Japanese- versus American-made cars for every 1 year increase is 0.348 (95% CI: 0.133-0.563). The effect of weight and cylinders are related at the 0.05 level. The effect of the number of cylinders is not significant at the 0.05 level, nor is the difference between MPG of Japanese- and American-made cars.**

c) Predict the `mpg` of the following car: A red car built in the US in 1983 that is 180 inches long, has eight cylinders, displaces 350 cu. inches, weighs 4000 pounds, and has a horsepower of 260. Also give a 95% CI for your prediction.
**Based on our model, this car would be expected to have 21.4 MPG (95% CI: 15.4-27.4).**
```{r cars prediction}
newcar <- Auto[1, ]
newcar[1] <- "NA"
newcar[3] <- 350
newcar[4] <- 260
newcar[5] <- 4000
newcar[6] <- "NA"
newcar[7] <- 83
newcar[9:16] <- "NA"
newcar <- newcar %>% mutate(year_center = year-76, weight_center = weight-2978, cylinders_center = cylinders-5.47)

predict(final_model, newcar,  interval = "predict", se.fit = TRUE)

```


# Simple Regression through simulations
    
## Linear model through simulations

This exercise is designed to help you understand the linear model using simulations. In this exercise, we will generate $(x_i, y_i)$ pairs so that all linear model assumptions are met.

Presume that $\mathbf{x}$ and $\mathbf{y}$ are linearly related with a normal error $\boldsymbol{\varepsilon}$ , such that $\mathbf{y} = 1 + 1.2\mathbf{x} + \boldsymbol{\varepsilon}$. The standard deviation of the error $\varepsilon_i$ is $\sigma = 2$. 

We can create a sample input vector ($n = 40$) for $\mathbf{x}$ with the following code:

```{r, eval = T, echo = TRUE}
# Generates a vector of size 40 with equally spaced values between 0 and 1, inclusive
x <- seq(0, 1, length = 40)
```


### Generate data

Create a corresponding output vector for $\mathbf{y}$ according to the equation given above. Use `set.seed(1)`. Then, create a scatterplot with $(x_i, y_i)$ pairs. Base R plotting is acceptable, but if you can, please attempt to use `ggplot2` to create the plot. Make sure to have clear labels and sensible titles on your plots.
```{r generate and plot data}
set.seed(1)
y <- 1 + 1.2*x + rnorm(40, sd = 2)
linear_data <- data.frame(x, y)

linear_plot <- ggplot(data = linear_data, aes(x=x, y=y)) +
  geom_point() +
  labs(title = "Figure 4.1: ", subtitle = "Scatter plot of x and y for the equation y = 1 + 1.2x + epsilon")
linear_plot
```


### Understand the model
i. Find the LS estimates of $\boldsymbol{\beta}_0$ and $\boldsymbol{\beta}_1$, using the `lm()` function. What are the true values of $\boldsymbol{\beta}_0$ and $\boldsymbol{\beta}_1$? Do the estimates look to be good? 
**The true value of B0 is 1 and the true value of B1 is 1.2. The estimates are close to the actual values but are off by ~0.3.**
```{r LS estimates}
linear <- lm(y ~ x)
summary(linear)
```

ii. What is your RSE for this linear model fit? Is it close to $\sigma = 2$? 
**RSE for this model is 1.79, which is somewhat close to 2.**
```{r rse}
sigma <- 2
n <- length(y)
sd_b1 <- sqrt(sigma^2 /((n-1)* (sd(x))^2))  # we will estimate sigma by rse in real life.
sd_b1
summary(linear)
```

ii. What is the 95% confidence interval for $\boldsymbol{\beta}_1$? Does this confidence interval capture the true $\boldsymbol{\beta}_1$?
**The 95% CI for B1 is -1.03 - 2.85. This does include the true value of B1 (which is 1.2).**
```{r}
confint(linear, 'x', level=0.95)
```


iii. Overlay the LS estimates and the true lines of the mean function onto a copy of the scatterplot you made above.
```{r linear plot with lines}
linear_plot_2 <- ggplot(data = linear_data, aes(x=x, y=y)) +
  geom_point() + geom_line(y=1+1.2*x, color="blue") + geom_smooth(method=lm, formula=y~x, color="red", se=FALSE) +
  labs(title = "Figure 4.1: ", subtitle = "Scatter plot of x and y for the equation y = 1 + 1.2x + epsilon", caption = "Blue line represents a perfect linear fit, red line represents the actual fit")
linear_plot_2
```


### diagnoses

i. Provide residual plot where fitted $\mathbf{y}$-values are on the x-axis and residuals are on the y-axis. 
```{r linear residuals}
plot(linear$fitted, linear$residuals, 
     pch  = 16,
     ylim = c(-8, 8),
     main = "Figure 4.2: Residual plot")
abline(h=0, lwd=4, col="red")

```

ii. Provide a normal QQ plot of the residuals.
```{r linear q-q plot}
qqnorm(linear$residuals, ylim=c(-8, 8), main = "Figure 4.3: Q-Q plot")
qqline(linear$residuals, lwd=4, col="blue")
```

iii. Comment on how well the model assumptions are met for the sample you used. 
**The model assumptions are met for this sample. There is some skewness but it is small and we can likely still assume that the data is normal.**


## Understand sampling distribution and confidence intervals

This part aims to help you understand the notion of sampling statistics and confidence intervals. Let's concentrate on estimating the slope only.  

Generate 100 samples of size $n = 40$, and estimate the slope coefficient from each sample. We include some sample code below, which should guide you in setting up the simulation. Note: this code is easier to follow but suboptimal; see the appendix for a more optimal R-like way to run this simulation.
```{r, eval = T, echo = TRUE}
# Inializing variables. Note b_1, upper_ci, lower_ci are vectors
x <- seq(0, 1, length = 40) 
n_sim <- 100              # number of simulations
b1 <- 0                   # n_sim many LS estimates of beta_1 (=1.2). Initialize to 0 for now
upper_ci <- 0             # upper bound for beta_1. Initialize to 0 for now.
lower_ci <- 0             # lower bound for beta_1. Initialize to 0 for now.
t_star <- qt(0.975, 38)   # Food for thought: why 38 instead of 40? What is t_star?

# Perform the simulation
for (i in 1:n_sim){
  y <- 1 + 1.2 * x + rnorm(40, sd = 2)
  lse <- lm(y ~ x)
  lse_output <- summary(lse)$coefficients
  se <- lse_output[2, 2]
  b1[i] <- lse_output[2, 1]
  upper_ci[i] <- b1[i] + t_star * se
  lower_ci[i] <- b1[i] - t_star * se
}
results <- as.data.frame(cbind(se, b1, upper_ci, lower_ci))

# remove unecessary variables from our workspace
#rm(se, b1, upper_ci, lower_ci, x, n_sim, t_star, lse, lse_output) 

```

i. Summarize the LS estimates of $\boldsymbol{\beta}_1$ (stored in `results$b1`). Does the sampling distribution agree with theory? 
```{r sample LS}
#plot b1
results %>%
  ggplot(aes(x = b1)) + 
  geom_histogram(aes(y = after_stat(density)), bins = 30) + 
  stat_function(
    fun = dnorm, 
    args = list(mean = mean(results$b1), sd = sd(results$b1)), 
    lwd = 2, 
    col = 'red'
  )

```
**The sampling distribution of $\hat{\beta_1}$ can be approximated as a normal distribution, which agrees with theory.**

ii.  How many of your 95% confidence intervals capture the true $\boldsymbol{\beta}_1$? Display your confidence intervals graphically. 
**95 of the confidence intervals capture the true value of B1 (or 95% of the simulations).**
```{r sample plot}
#plot1
results <- arrange(results, upper_ci)
results$num <- seq(1,100)
results <- results %>% mutate(contains_beta = ifelse(upper_ci>1.2 & lower_ci<1.2, 1, 0))
table(results$contains_beta)

plot <- ggplot(data=results) +
  geom_line(data=results, aes(x=num, y=upper_ci)) +
  geom_line(data=results, aes(x=num, y=lower_ci)) +
  geom_line(aes(x=num, y=1.2), color="blue") +
  labs(title = "Figure 4.3.1: ", subtitle = "Graph of 95% CIs for each simulation", y = "Value of Beta1", x = "Simulation number") 
plot

#plot2
ggplot(results, aes(num, b1)) + geom_point() + 
  geom_errorbar(aes(ymin = lower_ci, ymax = upper_ci)) +
  geom_hline(yintercept = 1.2, color = "blue") + 
  labs(title = "Figure 4.3.2: ", subtitle = "Graph of 95% CIs for each simulation", y = "Value of Beta1", x = "Simulation number") 
```





