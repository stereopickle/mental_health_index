# mental_health_index

This repository contains a code exploring the National Health and Nutrition Examination Survey Data from 2017 - 2018.  
I am primarily interested in learning what are the key factors that contribute to the progression of depressive symptoms in order to provide a guideline in future suicide prevention research. 

# Structure
- 001.Linear_Regression_NHANES_data.ipynb : jupyter notebook containing entire code in the order it was processed.


# Data Collection
Selection of data was downloaded from CDC website. (https://wwwn.cdc.gov/nchs/nhanes/continuousnhanes/default.aspx?BeginYear=2017)  
NHANES provides separate XPT files for each sub-sections. I only used the demographics, medical condition, substance use related data. But the collection code will iterate through all files in the folder, if one wants to expand the dataset.
Codebooks are distributed across different section of this website.  

Initially this dataset contained approximately 8000 participants. 

![percentage of answers in population](/PNG/p_answers_pop.png)

(quesitons are shown the reversed order in the graphic)  
Q1 - Have little interest in doing things  
Q2 - Feeling down, depressed, or hopeless  
Q3 - Trouble sleeping or sleeping too much  
Q4 - Feeling tired or having little energy  
Q5 - Poor appetite or overeating  
Q6 - Feeling bad about yourself  
Q7 - Trouble concentrating on things  
Q8 - Moving or speaking slow or too fast or fidgety and restless  
Q9 - Thought you would be better off dead or of hurting yourself  
Q10 - Difficulty these problems have caused (not at all, somewhat, very, extremely)  

# Dependent Variable
MHI_Index: In order to quantify one's mental health or depressive symptoms, I created a mental health index (MHI), which is a cumulative score of ten depression screener questionnaires. I subsetted the dataset to only people who have answered to have been bothered by feeling depressed or hopeless at any degree, because I'm interested in learning how mild depressive symptoms progress to be fatal.

![MHI_distribution](/PNG/MHI_distributionn.png)  
Distribution of MHI within a subset

# Independent Variables
## Demographics
### Age & Gender
age, gender: Survey age was capped at 80, meaning age of anyone older was inputted as 80. To adapt to this, I added some noise to this data by dispersing the data with age 80 to be random integer between 80 to 90. 59% was female. 

![Age and Gender](/PNG/age_gender_sm.png)  


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

I also rann adjusted R-squared on the entire dataset as the purpose is more analytic than prediction.  
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
### Residuals 
Liberally speaking, our residual is approximately normal, but not quite. Further investigation is necessary.  
![residual distribution](/PNG/resid_dist.png)  

High variance, but generally its homoscedasticity is a-okay. Slight over-estimation on both end.
![residual plot](/PNG/resid_plot.png)  


## Conclusion

Overall, 

1. Chronic illness such as cancer is a key predictor of frequent experience of depressive symptoms. Having cancer contributed to approximately 3.8 points increase in MHI. One way to get this much score increase is going from experiencing no symptom to experiencing one of the symptoms all the time.

![cancer](/PNG/cancer.png)  

2. On a similar line, having a disability was also a key predictor of frequent experience of depressive symptoms, contributing to approximately 3.2 points increase in MHI.

![disability](/PNG/disability.png)  

3. Being an only adult in the household also contributes to 2.7 point increase in MHI, after taking account for household size.

![n_adults](/PNG/n_adults.png)  

4. Next important key factor is the use of substances. Being a smoker attributes to one point increase in MHI.

![smoker](/PNG/smoker.png)  

5. Higher average number of daily cocaine use increase contributes to higher MHI.

6. Additionally education level are also important key factors of MHI. (Higher education, the lower MHI score.)

![education](/PNG/education.png)  
