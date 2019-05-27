# Detecting-people-at-risk-of-Chronic-Kidney-Disease

View code and visualization on RPubs: http://rpubs.com/michelledo202/ckd


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
