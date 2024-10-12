# **Credit Card Fraud Detection project**  

## **Objective** :  

The objective of this project is classification of transactions out of a dataset containing genuine and 
fraud transactions. The emphasis would be on correctly identifying the fraud transactions as this is 
relatively more critical to banking business than misidentifying genuine transactions.  

**************************************************************************  
## **Challenges** :  

1. Fraud transactions make up only 0.17% of the total transactions in the dataset. This can 
severely hamper the learning ability of classification models.

2. Features are all PCA transformed (except Time, Amount and Class).

************************************************************************** 
## **Methodology** :  

I have used resampling to run 3 variations :  

a) Undersampling majority class to match minority class by 100%.  
b) Oversampling minority class to 5% of majority class and then equating both classes.  
c) Oversampling minority class to 20% of majority class and then equating both classes.  

I have then finalized a dataset based on the third variation (20% of majority class) based on a 
combination of F1 score and Recall. The finalized dataset and model is fed into Sagemaker, trained 
and deployed as a serverless inference endpoint.  

**************************************************************************  
## **Project Flow** :  

A) DATA OVERVIEW  
---Initial data checks  
---Charts visualization  

B) FEATURE ENGINEERING  
---Outliers  
---Time adjustment  
---Correlation  
---Duplicated rows  
---Train test Split  
---Scaling  

C) TRAINING  
---Splits for training  
---Models used  
---Evaluation metrics  
------------Training for Variation 1  
------------Results  
------------Training for Variation 2  
------------Results  
------------Training for Variation 3  
------------Results  

D) FEATURE IMPORTANCE  

E) SAGEMAKER  
------------Uploading files  
------------Training with Random Forest and results  
------------Prediction input file setup  
------------Model deployment setup  
------------Serverless inference endpoint and cleanup  

**************************************************************************  
**Detailed steps** :  

## **1) Data Overview:**  

a) All features are PCA transformed (Except Time , Amount and Class)  
b)No Nulls present.  

**************************************************************************  
## **2) Chart visualization**  

a) Fraud datapoints have a more uniform spread relatively vs non frauds.  
b) Time and Amount are out of scale vs other transformed features.  

**************************************************************************  
## **3) Feature Engineering**  

a) Outliers  - Outliers have not been dropped from main dataset because a lot of rows (containing a lot 
of imbalanced Frauds) have been classified as outliers.  

b) Time feature adjustment - Time column has not been converted because there was no improvement in scores with 
the converted Time feature in trial runs and it was also ranked at the bottom in random 
forest feature importance, indicating it is not contributing to the model's pattern learning.  

c) Correlation - Most columns do not seem to be correlated , this could be because this is a PCA 
transformed dataset. There are features that are slightly negatively correlated with target 
“Class”.  

d) Duplicated rows - 1081 rows were dropped (19 frauds included) but as they are duplicates their removal will 
not affect learning.  

e) Train Test split - Dataset is split with 80% train and 20% in set.  

f) Scaling -  Robust Scaler is used to scale Time and Amount columns, it is not sensitive to outliers and 
skewed distributions. It uses the data point's difference from median and IQR division to scale features.

**************************************************************************  
## **4) TRAINING**  

## **a) Splits for training**  

**- Variation 1 :**  
Undersampling majority class 100% (387 Frauds : 387 Genuine ) 

**- Variation 2 :**  
Oversampling minority class to 5% of majority and undersampling majority to match 
minority class. (11329 Frauds : 11329 Genuine)

**- Variation 3:**  
Oversampling minority class to 20% of majority while undersampling majority to match 
minority class (45318 Frauds : 45318 Genuine)

**-Testing Set**
-Testing set remains the same for all (86 Frauds : 56660 Genuine)

************************************************************************** 
## **b) Models used**  

-XGB Classifier  
Builds an ensemble of decision trees , where each new tree corrects the errors made by the 
previous trees. Weak learning trees are combined to make a strong learner.  

-Random Forest Classifier  
Builds multiple decision trees and outputs the most frequent class out of all the predictions 
by individual trees.  

-Logistic Regression  
Uses the logistic sigmoid curve to classify both classes.  

-SVC  
Identifies a hyperplane that maximizes the margin (i.e. the distance from the nearest data 
point of either class to the hyperplane ) to separate both classes.  

***Key hyperparameters used were max depth (to control how deep the tree could grow in terms of split). 
Limiting the max depth reduced overfitting and reducing it too low made the model training poor. These were used for Random Forest and XGB Classifiers. For XGB in trials I also used Gamma, increasing the Gamma reduced overfitting. (Gamma 
decides to make a split on a leaf node or not, based on loss reduction)

************************************************************************** 
## **c) Evaluation metrics**  

- Recall  
Recall for the fraud class is the percentage of the correctly identified fraud classes among all 
the actual fraud classes. (i.e. how many fraud classes were identified)
- Precision 
Precision for the fraud class is the percentage of the correctly identified fraud classes among  
all the classes identified as fraud. (i.e. how many identified as fraud were actually fraud)
- F1 Score 
F1 score is the harmonic mean of Precision and Recall. It includes both metric into its 
formula and rates the ability of the model to identify both classes.
- Confusion matrix  
(Not used for training below, used in Sagemaker training) 
Confusion matrix is the actual number of true positives, true negatives, false positives and 
false negatives.

************************************************************************** 
## **d) Training Results**  

**d.1) Variation 1 : Undersampling majority class 100% (387 frauds ; 387 Genuine )**  
- For XGB : Recall for "fraud" is 88%, while precision is 4%
- For RFC : Recall for "fraud" is 84%, while precision is 9%
- For LogR : Recall for "fraud" is 88%, while precision is 4%
- For SVC : Recall for "fraud" is 83%, while precision is 8%
- F1 score for random forest was higher but recall was lower than Log Regression and XGB.
- Overall precision scores for fraud are very poor across models indicating all models have 
classified lots of genuine transactions as fraud.
- This could be because of massive undersampling with the model unable to learn trends of 
genuine transactions.

**d.2) Variation 2 : Oversampling minority class to 5% of majority and then equating both classes. (11329 frauds : 11329 Genuine).**  
- For XGB : Recall for "fraud" is 83%, while precision is 17%
- For RFC : Recall for "fraud" is 81%, while precision is 33%
- For LogR : Recall for "fraud" is 88%, while precision is 6%
- For SVC : Recall for "fraud" is 86%, while precision is 11%
- F1 score with a larger sample improved for both tree based models.
- Precision is still low, with only Random Forest performing relatively better than other 
models in not classifying genuine classes as fraud.
- Changing the hyperparameters did not increase scores much.

**d.3) Variation 3 : Oversampling minority class to 20% of majority and then equating both classes (45318 frauds : 45318 Genuine)**  
- For XGB : Recall for "fraud" is 84%, while precision is 35%
- For RFC : Recall for "fraud" is 80%, while precision is 53%
- For LogR : Recall for "fraud" is 88%, while precision is 6%
- For SVC : Recall for "fraud" is 87%, while precision is 10%
- All models have maintained  good recall scores across all samples of training.
- As sample size increased, the Precision and F1 score also increased for Tree-based models.
- This variation was selected , because it achieved a 80% + in recall and a better Precision 
and F1 score than the previous 5% run. ( I attempted another run with 120,000 Genuine 
rows and 120,000 Frauds but the dataset overfit at 100% and precision scores improved 
with more number of Genuine rows to learn from but Recall scores did not improve and 
dropped to below 80)
- A reason for Low precision scores could be the models fail to accurately learn patterns of 
fraud rows because of the overlap in the trends of the synthetic rows by SMOTE, so they 
classify some genuine transactions as fraud.

**************************************************************************  
## **5) Feature importance**  
V14 has contributed far ahead than other features with Amount and Time features 
contributing lesser than most others.  

**************************************************************************  
## **6) Sagemaker**  

**a) Uploading files**  
Once Sagemaker session had been created, files were converted to CSV and uploaded to S3 
bucket.

**b) Training with Random Forest and results**  
Model details and train test splits were loaded into Train.py file and run with spot instances set to ‘True’ (for cost benefits).  

![image](https://github.com/user-attachments/assets/ea4943ac-889d-4b66-a357-1ff72571fc89)  

- Testing Accuracy at 99.86% was slightly better than Training accuracy at 98.84%.
- **80% of fraud classes were correctly identified.**
- Out of all the classes identified by the model as fraud , 52% were actually fraud.
- F1 score is 63% which is an evaluation of how well the model identifies genuine vs fraud 
classes.

NOTE : Hyperparameter tuning  was not performed because the result was as per 
expectations and tuning was not affecting scores positively. But the code has been left in 
for the scope of this project.

**c) Prediction input file setup**  
This file was modified to load the inputs in as json files , process the file to match the train.py 
file, make predictions and return the output.  

**d) Model deployment setup**  
Deployment setup was done via SKLearnModel with the training job loaded in.  

**e) Serverless inference endpoint and cleanup**  
The model was deployed as serverless deployment (type of deployment designed for cost effectiveness and efficiency in predictions ) with the endpoint “creditfraud-2024-10-09-0529-39-503”.  
Endpoint was deleted after the run.  

**************************************************************************
