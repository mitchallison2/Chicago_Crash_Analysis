# Chicago Crash Type Predictor

**Authors**: Mitch Allison

## Overview

This project uses crash data from Chicago in order to create a model that can predict what type of crash has occured. The model uses data from the crash, person, and vehicle databases.

Chicago can use this model to determine what kind of crash has occured by only inputting 23 simple answers.

This project used an iterative approach to answering the business problem, so this project will be explained chronologically to best illustrate how the problem was approached.

## Business Problem

The cause of a crash, known as the crash type, can be difficult to surmise. There are lots of variables at play and people can lie or be un/nonresponsive when questioned in more serious cases.

Our goal is to create a model that, given some simple inputs, can predict the cause of a car accident in Chicago.

***
Notes:
* Although a bit of a weak data problem, it's hard to percieve of a clear need to identify crash types given an input.
***

## Data

The intake datasets include records from 2015 to August 2022.

Traffic_Crashes_-_Vehicles.csv - Information about vehicle status at time of crash. Includes variables such as vehicle defect, number of crashes, and vehicle length.

Traffic_Crashes_-_People.csv - Information about person status at time of crash. Includes variables such as drivers licence type, origin, sex, and age.

Traffic_Crashes_-_Crashes.csv - Information about the crash's environmental factors. Includes variables such as weather, street defect, and beat of occurance.

***
Notes:
* This data is publicly available, from the City of Chicago Data Portal.
* This data and it's derivatives(cleaned data, train/test splits) aren't included in the repo as they're large in file size.
***

## Methods

I applied iteration principles in order to clean the data and find a model that was able to predict the data. The iteration ended up taking the following steps:
* Data Preparation(Round 1)
* First Simple Model
* Data Preparation(Round 2)
* Gridsearch (Round 1)
* Data Preparation(Round 3)
* Gridsearch (Round 2)
* Final Model Analysis

***
Notes:
* More details about these steps can be found in the technical_notebook.ipynb
* This approach is appropriate considering the size/complexity of the datasets. I had over 1 million data points with 60+ categorical variables to boil down.
***

## Challenges

The following challenges led to the iterations in data prep, modeling, and gridsearching:
* SMOTE USAGE: Our targets were very imbalanced, as you can see on the graph below. The top class was more than the 2nd and 3rd class added together. I wanted to use SMOTE to help fix this issue to create a better model. This is due to the size of the dataset(1.3 million). Trying to use SMOTE on such a large dataset, with so many variables after applying OHE, made this too computationally expensive. FIX: Use undersampling or a model that compensates for class imbalance.
![target imbalance](./graphs/all_causes.jpg)


* TOO MANY TARGETS: As mentioned above, there was some fairly serious class imbalance. The top cause was 'unable to determine' which also isn't helpful for predicting. How is it helpful to feed the model information and the model tells you that it's 'unable to determine' the cause? FIX: Reducing the targets. This would also decrease the size of the dataset which would increase the speed of gridsearches.


* TOO MANY VARIABLES/TOO COMPLEX: We were able to reduce the original data to only 29 variables. However, all of these variables are categorical, which makes the OHE applied dataframe absolutely gigantic. FIX: additional data cleaning/engineering would need to be applied.


* SIMPLEIMPUTER NOT OPERATING: I'm still unsure of the cause of this issue, but using the imputer as a part of the pipelines on my gridsearches caused them to become unfunctional.

* TRANSFORMATION REQUIREMENTS: For categorical data: apply onehotencoder so that models can be made from categorical data. Scaled data so that it could better be modeled. Encode target data so that it could be modeled.


## Data Preparation

I examined the data and found these steps had to be taken in round 1:
* (Crashes data only) - 34 steps to clean data
* Ohe for categoricals
* Some light engineering to reduce complexity, such as creating an ordinal injury_level to describe injury
* Reduced from 49 to 18 columns

After creating a FSM, take these steps in round 2:
* Combine person and vehicle datasets with crash data
* person - reduce complexity. Reduce columns from 30 to 7.
* vehicle - reduce complexity. Reduce columns from 71 to 11.

After having difficulty with gridsearches, take these steps in round 3:
* Further reduce complexity of data: engineer 'simple' variables for categorical columns with more than 4 categories. For example, this boils down the list of possible car defects to defective/not defective.
* Screen data to top 4 causes: data is very imbalanced, and the top cause is 'unable to determine'. This isn't a helpful target to predict, so screen data to top 4 causes after ignoring the #1 cause.
* Screen data to data that makes sense(has drivers license, is old enough to drive, etc)
* Here are the top 4 classes I chose to predict on:
![target selection](./graphs/top_4_causes.jpg)
* Further reduce complexity: remove unneccessarily complex variables
* Able to reduce data down to 230k data points from 1M+
* Able to get down to 11 'simple' variables, 10 non-simple variables, and 4 target classes.

***
Notes:
* This data preparation happened in the order detailed in the methods section.
***


## Models and Analysis
The first simple model only had a score of 0.38, using multioutputclassifier with logisticregression.

By using train test splits, pipelines, and gridsearches, I was able to find an optimal model with randomforestclassifier.

This model was able to achieve a score/accuracy/recall/precision/f1 of 0.64
![confusion matrix](./graphs/confusion_matrix.jpg)

I also found the most important variables in determining the correct crash type to be sex and age.

## Conclusions

I was able to create a model that could predict a crash type given 23 inputs, as long as we're only predicting upon the top 4 crash types.

The most important variables in determine the crash type was sex and age.

***
Next steps:
Continue to iterate upon model:
- This model was only trained on 21% of total crashes, and only predicts the top 4 crash types. Including other types of crash types will allow the model to predict additional crash types.
- Improve performance by running additional gridsearches and checking other model types to see if they have better results.

Look more into ‘Unable to determine’ cause:
- It's likely that many crashes get lumped into this cause, when there is a true cause at hand. Can we discover the true cause based on model?
- The number one crash type, containing more than half of all crashes, has to be thrown out due to being uninterpretable. Is there a better way of interpreting this cause?

Deploy model:
- Answer 23 simple questions to find cause of crash. May be useful to law enforcement.
***

## For More Information

Please review our full analysis in [our Jupyter Notebook](./Technical_notebook.ipynb) or our [presentation](./Shareholder_presentation.pdf).

For any additional questions, please contact:
Mitch Allison
Email: mitch.allison2@gmail.com
GitHub: @mitchallison2
LinkedIn: linkedin.com/in/mitch-allison2


## Repository Structure

```
├── README.md                           <- The top-level README for reviewers of this project
├── Technical_notebook.ipynb            <- Narrative documentation of process in Jupyter notebook
├── Shareholder_presentation.pdf        <- PDF version of project presentation
├── .gitignore                          <- Ignored files(all data, saved files)
├── Datasets                            <- External/saved datasets. Empty due to size of datasets
├── EDA_Cleaning                        <- Folder with data preperation
  └── EDA_CRASHES.ipynb                 <- Contains data prep 1, 2. Works with crash data. Contains FSM.
  └── EDA_People.ipynb                  <- Contains data prep 2. Works with people data.
  └── EDA_Vehicles.ipynb                <- Contains data prep 2. Works with vehicle data.
  └── joined_dataset(person-vehicle-crash).ipynb                  <- Contains data prep 2. Combines crash, people, vehicle data.
  └── final_data_prep.ipynb             <- Contains data prep 3. Works with combined data.
├── Modeling_gridsearch                 <- Contains notebooks for gridsearching and modeling
  └── gridsearch1.ipynb                 <- First try of gridsearch. Didn't work due to mentioned difficulties.
  └── gridsearch2.ipynb                 <- 2nd try of gridsearch. Contains final model and final model scores.
  └── Model2.ipynb                      <- Attempted slight upgrade on FSM, not worth talking about.
├── Graphs                              <- Contains graphs
  └── all_causes.jpg                    <- Shows inbalance of crash causes
  └── confusion_matrix.jpg              <- Shows confusion matrix for final model
  └── Graphs.ipynb                      <- Where graphs were built
  └── top_4_causes                      <- Shows top 4 causes of crashes
```
