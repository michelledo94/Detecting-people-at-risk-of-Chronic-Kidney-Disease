# Detecting-people-at-risk-of-Chronic-Kidney-Disease
The data set consists of selected information from 8,819 adults 20 years of age or older taken from the 1999 to 2000 and 2001 to 2002 surveys. The sample subjects were randomly divided into two pools: a 6,000-case training set and a 2,819-case hold-out sample for testing. A test for CKD was administered to everyone in the study population. The dependent variable of interest is CKD, a binary variable indicating whether or not the subject had CKD. The 33 independent variables include age, weight, income, cholesterol level, systolic/diastolic blood pressure, family history of diabetes, cardiovascular diseases and so on.

The goal of the project is to predict the risk score for CKD as well as the disease status of tested patients (hold-out sample), find the best predictors of CKD and build a predictive model that could be turned into a quick screening tool for early detection of the disease. Given that the total annual bill for treating kidney failure (the last stage of CKD) is over $35 billion, early detection and treatment of CKD can save insurance companies $250,000 per case in which the patient does not progress to kidney failure and the Federal government approximately $1 billion per year (US Dept. of Health and Human Services).

Features: 
+ID: Identification number
+Age: Age (years)
Female: 1 if female
Racegrp: Self-reported race/ethnic group (white, black, Hispanic, other)
Educ: 1 if more than high school
Unmarried: 1 if unmarried
Income: 1 if household income is above the median
CareSource: Self-reported source of medical care (Dr./HMO, clinic, noplace, other)
Insured: 1 if covered by health insurance
Weight: Weight (kg)
Height: Height (cm)
BMI: Body mass index (kg/m2)
Obese: 1 if BMI is greater than 30 kg/m2
Waist: Waist circumference (cm)
SBP: Systolic blood pressure (max)
DBP: Diastolic blood pressure (min)
HDL(mg/dL): the good cholesterol
LDL(mg/dL): the bad cholesterol
Total Chol(mg/dL): the sum of good and bad cholesterol
Dyslipidemia: Too high LDL or too low HDL
PVD (Peripheral vascular disease): reflected by reduced SBP at the leg relative to the
arm
Activity: Mostly sit (1); stand or walk a lot (2); lift light loads or climb stairs often (3); heavy work and heavy loads (4)
Poor Vision: Self-reported poor vision
Smoker: Smoked at least 100 cigarettes
Hypertension: The presence of at least one of four indicators of high blood pressure
Fam.Hypertension: Family history of hypertension (high blood pressure)
Diabetes: Self-reported physician diagnosed or lab test result
Fam Diabetes: Family history of diabetes
Stroke: Self-reported response to Has a doctor ever told you that you had a stroke?
CVD: Response to Has a doctor ever told you that you had angina pectoris, myocardial infarction, or stroke?
Fam CVD: Family history of cardiovascular disease
CHF: Self-reported response to Has a doctor ever told you that you had congestive heart failure?
Anemia: Treatment for anemia received in past three months or hemoglobin at exam lower than 11g/dL
CKD: Chronic kidney disease as indicated by measured serum creatinine
