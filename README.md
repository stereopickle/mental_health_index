# Depressed to Suicide Ideation 
## Preliminary Guideline for Suicide Prevention Research

This repository contains a process exploring the National Health and Nutrition Examination Survey Data from 2017 - 2018 to find factors that attribute to progression of depressive symptoms. The goal of this project to provide some guideline in future suicide prevention effort. 

# Structure
- 001.Linear_Regression_NHANES_data.ipynb : jupyter notebook containing entire code in the order it was processed.

# Problem
According to National Institute of Mental Health (NIMH), in 2017, suicide was the tenth leading cause of death in US and the second among age 10 to 34. Yet, suicide is also one of the most stigmatized topic. People avoid talking about suicidal ideation, making it even harder to learn what makes people go from feeling unmotivated or depressed to having suicidal ideation.
 
source: NIMH website (https://www.nimh.nih.gov/health/statistics/suicide)

# Data 
The National Health and Nutrition Examination Survey (NHANES), which is a part of Centers for Disease Control and Prevention, collects and publishes extensive amount of interview and medical exam from a population of noninstitutionalized civilians living in US. 

In order to accurately represent the population, the NHANES oversample from underrepresented population each cycle. This dataset specifically oversampled hispanics, non-hispanic black and Asians, people at or below 185% of poverty guideline, and people aged 80 and older.

The NHANES website provides separate SAS XPT files for each sub-sections. In this analysis, I concatenated 21 of these sub-sections from topics such as demographics, medical condition, and substance use.  

source: NHANES website (https://wwwn.cdc.gov/nchs/nhanes/continuousnhanes/default.aspx?BeginYear=2017)

A total of 8704 participants completed interviews and medical exams between 2017 and 2018. 
As I'm interested in learning how mild depressive symptoms progress to be fatal, I narrowed down the dataset to only include people who have said that they have experienced feeling bothered by feeling depressed or hopeless recently. This resulted in a total of 1222 people, which is 14% of the total data. 


# Target Variable: Mental Health Index (MH_Index)
In order to quantify one's mental health or depressive symptoms, I created a variable called **mental health index (MHI)**, which is a cumulative number of answers to ten depression screener questionnaires. Each question was rated on 4-point scale, indicating how often they feel bothered by some of these depressive symptoms in past 2 weeks. Final MHI (Mental Health Index) ranges from 0 to 30. Below graph shows percent of each answer out of all people who answered at least half of these questionnaire (*n* = 5087). (A few missing values were inferred from answers to other highest correlated question.) 

![percentage of answers in population](/PNG/p_answers_pop.png)

This is overall distribution of MHI within our subset.

![MHI_distribution](/PNG/MHI_distributionn.png)  
Distribution of MHI within a subset, which shows that most of the participants were at around 5 points. To recap, high MHI score indicates higher reported score of depression screener. Most people who answered these 10 questions reported MHI of 0 (not included in the analysis).

![Age and Gender](/PNG/age_gender_sm.png)  

Our group had 59% of female, and relatively evenly distributed age ranges from 18 to 70s, and some elders above 80.


# Independent Variables
Based on exploratory analysis and my experience at suicide prevention center, I focused on three broad topics: Demographics, existing medical condition (some of which I call the ‘hopelessness’ measure), and substance use. 

## Demographics
### Age & Gender
age, gender: Survey age was capped at 80, meaning age of anyone older was inputted as 80. To adapt to this, I added some noise to this data by dispersing the data with age 80 to be random integer between 80 to 90. 59% was female. 


### Number of Household Members & Number of adults
total_household, n_adults, n_dependents, only_adult: From my experience from suicide prevention center, I've learned that hopelessness is often the determining factor of suicide ideation. I want to test this measure on various factors. One of causes of hoplessness (not always) might be being the sole caretaker of the household when facing an overwhelming problem. So I engineered new features "number of adults" and "only adult" to from information on number of children and total household.  

![number of adults](/PNG/n_adults.png)  

### Income 
income_avg, income_p_household: Tiered incomes was inputted as mean value of the tier interval. There was correlation between income and MH_Index. For the fit, log transformed data was used. Additionally I included a new feature income / adults and income / household.

### Marital Status
marital_st: Married, widowed, divorced, separated, single, living with partner. 
WDS_marital: binary. widowed, divorced, separated
They didn't have much direct relationship to MH_Index, but interaction with WDS showed some trend in grouping these variables. (See Interaction)  

### Veterans
veteran: I combined veterans who have served in US and ones who have served in foreign countries. There was no significant difference in MHI between veterans and non-veterans.

### Education
education: I combined information for 20+ and for 18, 19 year olds. Kruskal Wallis showed that there are significant difference in MHI between education group (p < 0.001). 

## Medical Condition
### Obesity
BMI: In the previous dataset (2013-2014), I used sagittal abdominal diameter as more accurate measure of obesity. But this data was not available in 2017 - 2018 set. So instead I used BMI. 

### Physical disability
disability: is a binary value indicating having ANY physical difficulty out of hearing, walking, seeing, dressing, bathing, or doing errands alone.  
Two-sample t-test showed that there is a significant difference in MHI between having disability and not having disability.  

### Chronic Illness
cancer: binary (have diagnosed cancer or not)  
Also to test what I called 'hopelessness' measure earlier, I added cancer_d_yr (year diagnosed cancer) and yr_since_cancer (age - cancer_d_yr).  

## Substance Abuse
Some research suggested that tobacco smoking is a risk factor for depression. I wanted to explore more. 
### Smoker
smoker: smoker, nonsmoker, quitter, occasional smoker  
cig_init_yr: the year started smoking  
cig_quit_yr: the year quit smoking  
avg_n_cig: average number of cigaretts per day  

### Drug Use
marijuana: frequent use of marijuana (y/n)  
n_cocaine: number of time used cocaine.  
heroine: ever used heroine  
n_meth: number of time used Methamphetamine  

## Interaction
### Only Adult X Number of Household
NeverMarriedXAge: Kruskal Wallis test looking at if the MH_Index is different between different number of adults in the household showed that there are significant difference between number of adults (one or more of the pairs at least) at the significance level of 0.05. (p< 0.05) Based on this result, I also included interaction between being only adult and number of total household. Having the interaction in the model, being only adult yielded high coefficient at p-value < 0.01. 

### WDS X Age

![number of adults](/PNG/ageXmarital.png)  

WDSXAge: Looking at the plot, even though messy, marital status (2, 3, 4) seemed to have a trend against others. 2, 3, 4 refers to widowed, divorced and separated. As WDS status likely means quite differently across age, I decided to group WDS and look at interaction between WDS and age. Additionally I also included "never married" as a counter variable to "WDS". Never married and age showed some interaction from visualization.

![never married X age](/PNG/never_mar_age.png)  

### Age X Year quitting or starting cigarette
AgeXYearQuitting
AgeXYearInitSmoke

## Data Transformation
Log transformation on right skewed continuous data (e.g. income) and dummy conversion on non-binary categorical data (e.g. marital st) was applied.
Since I was interested in finding factors to prioritize in order of importance, I also applied standardization on all scalar data. 

## Model Selection
### Baseline model
Instead of using a dummy, I made a baseline model by fitting preprocessed demographics data, after removing select features manually. I yielded cross-validation score using sklearn's train_test_split. (train R2 = 0.047 and test R2 = 0.026) From here I added medical condition and substance use sets of features and interactions and evaluated model fit.

### Feature Selection
#### 1. As is / Dropping multicollinearity measures manually
Train-Test-Split results improved some from baseline.  
Train R2: 0.164599799643526  
Test R2: 0.14995473970717976  

I also ran adjusted R-squared on the entire dataset as the purpose is more analytic than prediction.   
Adjusted R2: 0.146  

#### 2. Recursive Feature Elimination (RFE)
Train data improved but test data didn't from as is model. But adjusted R2 was slightly higher. For this reason I chose this as a final model.  
Train R2: 0.16772477909317296   
Test R2: 0.14011221042122435    

Adjusted R2: 0.150

#### 3. Variance Inflation Factor (VIF)
Test performance was much worse.   
Train R2: 0.16481350361134806  
Test R2: -1.2452051215043353e+19  

Adjusted R2: 0.147

#### 4. Stepwise Selection & 5. Lasso  
Alternative functions are in the jupyter notebook. However I did not utilize these methods in the evaluation.  

## Final Model
After taking RFE to select features, I iteratively removed the feature with the highest p-value as long as it contributed to the goodness of fit (Using adjusted R2). Led to a final model with Adjusted R2 = 1.56 / R2 = 0.167. Final features with p > 0.05and coefficients are reported here. (For a full summary, please refer to Jupyter notebook.)  

| Variable  | Coef | p |  
| ------------- | ------------- | ----- |
| Intercept | 6.9096 | 0.000 |  
| disability | 3.2099 | 0.000 |  
| cancer | 3.7632 | 0.003 |  
| smoker_3 | 0.8871 | 0.018 |   
| age_sc | -0.6583 | 0.002 |  
| cancer_d_yr_sc | -1.0153 | 0.006 |   
| n_cocaine_sc | 0.5218 | 0.002 |   
| education_sc | -0.4724  | 0.003 |   
| NeverMarriedXAge | 0.8294 | 0.025 |  


### Residuals 
Liberally speaking, our residual is approximately normal, but not quite. Further investigation is necessary.  
![residual distribution](/PNG/resid_dist.png)  

High variance, but generally its homoscedasticity is a-okay. Slight over-estimation on higher end.
![residual plot](/PNG/resid_plot.png)  


## Conclusion

Mental health is a complex issue it is very difficult to narrow down all the factors that may contribute to one’s mental health progression, and the resulting model has a very low goodness of fit. But below I report some of the meaningful findings (p < 0.5) to guide suicide prevention research.  
1. Having a disability was also a key predictor of frequent experience of depressive symptoms, contributing to approximately 3.2 points increase in MHI.

![disability](/PNG/disability.png)  

2. Chronic illness such as cancer is a key predictor of frequent experience of depressive symptoms. Having cancer contributed to approximately 3.8 points increase in MHI. One way to get this much score increase is going from experiencing no symptom to experiencing one of the symptoms all the time. 

![cancer](/PNG/cancer.png)  

3. Moreover, each standard deviation increase in year of diagnosis (shorter time to the current time) attributed to 1 point decrease in MHI. 

4. Next important key factor is the use of substances. Being a smoker attributes to 0.9 point increase in MHI.

![smoker](/PNG/smoker.png)  

5. Higher average number of daily cocaine use increase also attributed significantly to higher MHI (coef = 0.52).


6. Next to year of diagnosis of cancer, increase of age without ever married was the next highest predictor of increase in MHI.

![never_married](/PNG/never_married.png)  

7. Additionally education level are also important key factors of MHI. (Higher education, the lower MHI score.)

![education](/PNG/education.png)  
