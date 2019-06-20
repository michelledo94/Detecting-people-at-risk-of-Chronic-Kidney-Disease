# Detecting-people-at-risk-of-Chronic-Kidney-Disease

View full code and visualization on RPubs: http://rpubs.com/michelledo202/ckd


The data set (`casedata.csv`) consists of selected information from 8,819 adults 20 years of age or older taken from the 1999 to 2000 and 2001 to 2002 surveys. The sample subjects were randomly divided into two pools: a 6,000-case training set and a 2,819-case hold-out sample for testing. A test for CKD was administered to everyone in the study population. The dependent variable of interest is CKD, a binary variable indicating whether or not the subject had CKD. The 33 independent variables include age, weight, income, cholesterol level, systolic/diastolic blood pressure, family history of diabetes, cardiovascular diseases and so on.


The goal of the project is to predict the risk score for CKD as well as the disease status of tested patients (hold-out sample), find the best predictors of CKD and build a predictive model that could be turned into a quick screening tool for early detection of the disease. Given that the total annual bill for treating kidney failure (the last stage of CKD) is over $35 billion, early detection and treatment of CKD can save insurance companies $250,000 per case in which the patient does not progress to kidney failure and the Federal government approximately $1 billion per year (US Dept. of Health and Human Services).


Features: 
* __ID__: Identification number
* __Age__: Age (years)
* __Female__: 1 if female
* __Racegrp__: Self-reported race/ethnic group (white, black, Hispanic, other)
* __Educ__: 1 if more than high school
* __Unmarried__: 1 if unmarried
* __Income__: 1 if household income is above the median
* __CareSource__: Self-reported source of medical care (Dr./HMO, clinic, noplace, other)
* __Insured__: 1 if covered by health insurance
* __Weight__: Weight (kg)
* __Height__: Height (cm)
* __BMI__: Body mass index (kg/m2)
* __Obese__: 1 if BMI is greater than 30 kg/m2
* __Waist__: Waist circumference (cm)
* __SBP__: Systolic blood pressure (max)
* __DBP__: Diastolic blood pressure (min)
* __HDL(mg/dL)__: the good cholesterol
* __LDL(mg/dL)__: the bad cholesterol
* __Total Chol(mg/dL)__: the sum of good and bad cholesterol
* __Dyslipidemia__: Too high LDL or too low HDL
* __PVD (Peripheral vascular disease)__: reflected by reduced SBP at the leg relative to the
arm
* __Activity__: Mostly sit (1); stand or walk a lot (2); lift light loads or climb stairs often (3); heavy work and heavy loads (4)
* __Poor Vision__: Self-reported poor vision
* __Smoker__: Smoked at least 100 cigarettes
* __Hypertension__: The presence of at least one of four indicators of high blood pressure
* __Fam.Hypertension__: Family history of hypertension (high blood pressure)
* __Diabetes__: Self-reported physician diagnosed or lab test result
* __Fam Diabetes__: Family history of diabetes
* __Stroke__: Self-reported response to Has a doctor ever told you that you had a stroke?
* __CVD__: Response to Has a doctor ever told you that you had angina pectoris, myocardial infarction, or stroke?
* __Fam CVD__: Family history of cardiovascular disease
* __CHF__: Self-reported response to Has a doctor ever told you that you had congestive heart failure?
* __Anemia__: Treatment for anemia received in past three months or hemoglobin at exam lower than 11g/dL
* __CKD__: Chronic kidney disease as indicated by measured serum creatinine

```{r message=FALSE, warning=FALSE}
library(dplyr)
library(ggcorrplot)
library(ggplot2)
library(caret)
library(mice)
library(tidyr)
library(ROCR)
library(varhandle)
```

### Data preparation

```{r warning=FALSE}
casedata = read.csv("casedata.csv")
```


Check the data:

```{r}
casedata1=casedata[,-c(1, 34)] # remove ID and disease status
str(casedata1)
```

Check missing pattern: 

```{r results='hide'}
# Unique values per column
lapply(casedata, function(x) length(unique(x))) 
```


```{r fig.width=10}
#Check for Missing values
missing_values = casedata %>% summarize_all(funs(sum(is.na(.))/n())) # from package tidyr
missing_values = gather(missing_values, key="feature", value="missing_pct")
missing_values %>% 
  ggplot(aes(x=reorder(feature,-missing_pct),y=missing_pct))+
  geom_bar(stat="identity",fill="tomato3")+
  coord_flip()+theme_bw()
```

Most of the missing values come from CKD, which is expected because we will hold out observations with missing CKD status. Variables with a significant amount of missing values are Income, PoorVision, Unmarried and Fam.CVD. In total, there are 24 variables having missing values. 

I'll combine the multiple imputation methods to predict missing values based on known values. 

 
```{r}
init = mice(casedata1, maxit=0) 
meth = init$method
predM = init$predictorMatrix
```


Skip variables without missing values, ones that will be used as predictors: 

```{r}
meth[c("Smoker", "Racegrp", "PVD", "Female", "Fam.Hypertension", "Fam.Diabetes", "Dyslipidemia", "CareSource", "Age")]=""
```


Specify imputation methods for each variable with linear regression for continuous variables, logistic regression for binary variables and polytomous logistic regression for ordinal variables. 

```{r}
meth[c("Educ", "Unmarried", "Insured", "Obese", "PoorVision", "Hypertension", 
       "Diabetes", "Stroke", "CVD", "Fam.CVD", "CHF", "Anemia", "Income")]="logreg" 
meth[c("Weight","Height", "BMI", "Waist", "SBP", "DBP", "HDL", "LDL", "Total.Chol")]="norm" 
meth[c("Activity")]="polyreg"

categorical = casedata1[,c("Income", "Educ", "Unmarried", "Insured", "Obese", "PoorVision", "Hypertension", "Diabetes", "Stroke", "CVD", "Fam.CVD", "CHF", "Anemia", "Smoker", "PVD", "Female", "Fam.Hypertension", "Fam.Diabetes", "Dyslipidemia", "Activity")]
categorical_var = lapply(categorical, as.factor)
categorical_var=as.data.frame(categorical_var)
casedata1 = cbind(casedata1[,c("Weight", "Height", "BMI", "SBP", "DBP", "Waist", "HDL", "LDL", "Total.Chol", "Racegrp", "CareSource", "Age")], categorical_var)
```

Impute missing values: 

```{r results='hide', cache=TRUE}
set.seed(1)
imputed = mice(casedata1, method=meth, predictorMatrix=predM, m = 5)
```

Integrate imputed data into the main dataset:

```{r}
complete = mice::complete(imputed)
fulldata=mutate(complete, CKD=casedata$CKD)
sapply(fulldata, function(x) sum(is.na(x))) # Check the number of missing values after imputation
```

One-hot encode categorical variables: 

```{r}
dv = dummyVars(~ Racegrp+CareSource, data = fulldata, sep = ".", fullRank = TRUE) 
nv = as.data.frame(predict(dv, newdata = fulldata))
```


```{r}
fulldata=fulldata[,-c(10, 11)] # combine dummified variables
fulldata=cbind(fulldata, nv)
```


Separate 2819 cases without CKD status for later use as a test set:

```{r}
fd=subset(fulldata, is.na(CKD)==FALSE)
test=subset(fulldata, is.na(CKD)==TRUE)
dim(fd)
```

### EDA

Correlation plot:

```{r}
# Handle factor variables
cv = fd[,c("Educ", "Unmarried", "Insured", "Obese", "PoorVision", "Hypertension", 
       "Diabetes", "Stroke", "CVD", "Fam.CVD", "CHF", "Anemia", "Income", "Dyslipidemia", 
       "PVD", "Smoker","Fam.Hypertension", "Female", "Fam.Diabetes", "Activity")]
cv = lapply(cv, as.numeric)
cv = as.data.frame(cv)

for (i in 1:19){
  cv[,i] = cv[,i] - 1
}

fd = fd[,-which(names(fd) %in% c("Educ", "Unmarried", "Insured", "Obese", "PoorVision", "Hypertension", 
       "Diabetes", "Stroke", "CVD", "Fam.CVD", "CHF", "Anemia", "Income", "Dyslipidemia", 
       "PVD", "Smoker","Fam.Hypertension", "Female", "Fam.Diabetes", "Activity"))]
fd = cbind(cv, fd)
```


```{r fig.width=12, fig.height=12}
cor <- cor(fd)
ggcorrplot(cor)
```

Comparing attributes of people with and without CKD (the red line indicates those tested positive with the disease):

```{r message=FALSE, warning=FALSE, fig.align='center'}
a=subset(fd, fd$CKD==1)
b=subset(fd, fd$CKD==0)
plot(density(a$Age), col="red", main='Comparing age distribution', xlab='Age')
lines(density(b$Age))
```

There's a staggering difference in age distribution of people with and without CKD. Most of those diagnosed with the disease are between 70 and 90 years old. 

```{r fig.align='center'}
plot(density(a$Activity, bw = 0.05), col = 'red', main = "Comparing Activity patterns", xlab = 'Activity level')
lines(density(b$Activity))
```


Activity Levels are given as 1 - Mostly sit, 2 - Stand or walk a lot, 3 - Lift light loads or climb stairs often and 4 - Heavy work and heavy loads. People having CKD tend to have a more sedentary lifestyle with little to no physically activities. Meanwhile, those without CKD are more likely to be physically active. 


```{r fig.align='center'}
plot(density(a$SBP, bw = 0.6), col = 'red', main = 'Comparing systolic blood pressure patterns', xlab = 'Systolic blood pressure level')
lines(density(b$SBP, bw = 0.6))
```

Those tested positive with CKD tend to have a higher level of systolic blood pressure. 

```{r fig.align='center'}
plot(density(a$Income, bw=0.2), col = 'red', main = "Comparing income levels", xlab = 'Income level')
lines(density(b$Income, bw=0.2))
```

Note: Income level is a binary variable indicating whether or not a person has an above-median income. 

```{r}
prop.table(table(fd$Income, fd$CKD), margin = 2) # specific ratios  
```

42% of the people tested negative with CKD have an income level that's above the median, while the number for those tested positive is only 28%. 

### Logistic regression

Based on the corelation matrix and some background research, we can safely remove variables HDL, LDL (bad/good cholesterol, providing overalapping information with Total Cholesterol), Obese, Waist (highly correlated with Weight) and BMI (computed from Weight and Height). The dataset will be split by 70/30 for training and validating regression models.

```{r}
fd=fd[,-which(names(fd) %in% c("BMI","HDL", "LDL", "Obese", "Waist"))] # remove unwanted variables 
split = round(nrow(fd)*.7) # split by 70/30
train = fd[1:split,]
validate=fd[(split+1):nrow(fd),]
dim(train)
```


Generalized linear model with all variables: 

```{r}
model=glm(CKD~.,family="binomial", data=train)
summary(model)
```

Model 1 - Logistic regresison with variables selected using stepwise regression. The procedure is used for selecting the best predictors for the dependent variable using many methods to evaluate model fit like t-test, F-test, adjusted R-squared, etc. Here we use AIC (Akaike Information Criterion - https://en.m.wikipedia.org/wiki/Akaike_information_criterion) and p-value. The smaller the AIC value is, the better the model is. 

Forward selection starts with no variable in the model, then tests the addition of each variable using a chosen model fit criterion, adds the variable (if any) whose inclusion gives the most statistically significant improvement of the fit, and repeats this process until none improves the model to a statistically significant extent.

In the summary, we will see two deviances NULL and Residual. Deviance is a measure of goodness of fit of a model. Higher numbers always indicate bad fit. The null deviance shows how well the dependent variable is predicted by a model that includes only the intercept (grand mean), while residual includes independent variables.


```{r results='hide'}
model1=step(model,direction="forward")
```


```{r}
summary(model1)
```

Next, I try backward elimination, which starts with all candidate variables, tests the deletion of each variable using a chosen model fit criterion, deletes the feature (if any) whose loss gives the most statistically insignificant deterioration of the model fit, and repeats this process until no further variables can be deleted without a statistically significant loss of fit.

```{r results='hide'}
model2=step(model,direction="backward") 
```


```{r}
summary(model2)
```

Interpret: The final result above gives us the most important variables. The AIC value indicates that model 2 is better than model 1. 

Predict on the validation set:

```{r cache = TRUE}
prob = predict(model2, type = "response", newdata=validate)
```

### Model evaluation

Find the Area Under the Curve (AUC) of Model 2:

```{r}
ROCRpred = prediction(prob, validate$CKD)
auc = round(as.numeric(performance(ROCRpred, "auc")@y.values),2)
cat("AUC of Model2 is",auc)
```

The model has an AUC of 0.88, which is quite decent. Let's see the ROC curve with respect to True Posive Rate (TPR) and False Positive Rate (FPR):

```{r fig.width=7, fig.height=5, fig.align='center'}
ROCRperf = performance(ROCRpred, "tpr", "fpr")

# Adding threshold labels
plot(ROCRperf, colorize=TRUE, print.cutoffs.at = seq(0,1,0.1), text.adj = c(-0.2, 1.7))
abline(a=0, b=1)

auc_train <- round(as.numeric(performance(ROCRpred, "auc")@y.values),2)
legend(.8, .2, auc_train, title = "AUC", cex=1)
```

### Classification threshold

```{r cache=TRUE}
opt.cut = function(ROCRperf, ROCRpred){
    cut.ind = mapply(FUN=function(x, y, p){
        d = x^2 + (y-1)^2
        ind = which(d == min(d))
        c(sensitivity = y[[ind]], specificity = 1-x[[ind]], 
            cutoff = p[[ind]])
    }, ROCRperf@x.values, ROCRperf@y.values, ROCRpred@cutoffs)
}
print(opt.cut(ROCRperf, ROCRpred))
```

Opt.cut is the threshold that would give us the highest possible TPR. If the project's goal is to find out as many positive cases as possible and avoid FN at all cost, which is more preferred in healthcare, then we can settle at 0.063. But if the goal is to maximize accuracy rate, which takes both TP and TN into account, we should increase the threshold a bit. 

```{r}
# evaluation function
c_accuracy=function(actuals,classifications){
  df=data.frame(actuals,classifications);
  
  tp=nrow(df[df$classifications==1 & df$actuals==1,]);        
  fp=nrow(df[df$classifications==1 & df$actuals==0,]);
  fn=nrow(df[df$classifications==0 & df$actuals==1,]);
  tn=nrow(df[df$classifications==0 & df$actuals==0,]); 
  
  recall=tp/(tp+fn)
  precision=tp/(tp+fp)
  accuracy=(tp+tn)/(tp+fn+fp+tn)
  tpr=recall
  fpr=fp/(fp+tn)
  fmeasure=2*precision*recall/(precision+recall)
  scores=c(recall,precision,accuracy,tpr,fpr,fmeasure,tp,tn,fp,fn)
  names(scores)=c("recall","precision","accuracy","tpr","fpr","fmeasure","tp","tn","fp","fn")
  
  #print(scores)
  return(scores);
}
```

Let's check a few cutoff values: 

```{r cache = TRUE}
class=ifelse(prob>0.063,1,0)
c_accuracy(validate$CKD, class)
```


```{r}
class=ifelse(prob>0.08,1,0)
c_accuracy(validate$CKD, class)
```


```{r}
class=ifelse(prob>0.15,1,0)
c_accuracy(validate$CKD, class)
```

Note: TPR decreases as accuracy rate increases. 

```{r fig.width=10, fig.align='center'}
barplot(sort(abs(model2$coefficients[-1]), decreasing=TRUE), col="steelblue", cex.names = .6, las=2, main="Variable comparison")
```


Based on the final model, we can build a simple questionnaire or screening software to check whether a person has a high risk for CKD and should undergo further medical tests. 

### Sample screening test: 

1. Gender? (Female/Male/Other)
2. Race group? (White/Hispanic/Black/Other)
3. Age?
4. Care source? (Clinic/Dr.-HMO/No place/Other)
5. Weight?
6. Height?

7. What is your most common type of activity? (Choose one)
+ Mostly sit 
+ Stand or walk a lot 
+ Lift light loads or climb stairs often 
+ Heavy work and heavy loads 

8. Check on whatever applies to you: 
+	Have a treatment for anemia received in past three months or hemoglobin at exam lower than 11g/d (Anemia)
+	Have been diagnosed with peripheral vascular disease reflected by reduced SBP at the leg relative to the arm (PVD)
+ Have been diagnosed with angina pectoris, myocardial infarction, or stroke (CVD)
+ Have a family history of hypertension (high blood pressure)
+ Have been diagnosed with diabetes
+ Have a family history of cardiovascular disease
+ Have a history of hypertension


After collecting data from the questionnaire above, suppose the input data of new patients is arranged in a similar fashion with the one we've worked on before, an example screening tool could be given as follows: 

```{r}
screeningTool = function(db) {
    p = predict(model2, type="response", newdata=db)
    db['CKD'] = ifelse(p>cutoff,1,0)
    return(db)
}
```
