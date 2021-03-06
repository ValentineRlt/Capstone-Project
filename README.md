 # Investing the Future of Energy 

The 2020 decade was originally heralded as the last one during which our Society could effectively address the environmental damage resulting from excessive GHG emissions caused by our system’s present dependence on fossil fuels. A transition to Renewable Energy [RE] sources is a compelling solution which could enable the achievement of a Net Zero Carbon Economy by 2050 (as proclaimed by the UK government in 2019<sup>1</sup>).

I was inspired by Client Earth’s founder, James Thornton’s statement that transforming such a deeply rooted institution as the Energy Sector necessitates a drastic change in both the legal and financial system<sup>2</sup>. Therefore, I chose to focus on the most data-driven actor, curious as to what informs their investment decisions. Indeed, I had a hunch that a better understanding of investors’ motivations could help incentivize them to support the development of RE. Alternatively, such insight could also benefit budding RE companies seeking. In any case, information resulting from this study seemed to have the potential to accelerate the transition away from fossil fuels.  
	
Whilst I initially sought a dataset informing of  RE investments evolution, their return on investments [ROI] or, optimally, Green Bonds’ trade, I eventually had to settle on the most thorough and freely accessible dataset I could find. The International Renewable Energy Agency [IRENA] generously shares a dataset consisting of International Development Agencies’ [IDA] investments of RE projects between 2000 and 2017. 
	
This set notably contained a promising variable which gave this **Capstone Project its target: I would aim to predict the amount invested in a project (expressed in millions of dollars)**. Considering the set’s relatively small size for a data analysis, I decided to make this a classification task, defining 4 classes in accordance to each quartile of total investment.
Curious to uncover what IDA’s choice of investment placement might be a factor of, I merged the IRENA dataset with various features selected from the World Bank’s [WB] Open Data Library to investigate possible relations. 

**Explore results on Tableau here:** https://bit.ly/VRpjtab 

<sup>1</sup> *UK Government commitment Net Zero by 2050, June 2019 announcement https://bit.ly/uknetzero*

<sup>2</sup> *The Outdoors Podcast, The Whole Truth with James Thornton, (22/04/2020) https://bit.ly/Outdoorpod*

## Problem statement:

*What features of a RE project most determine the category of investment it will receive from IDAs?*

## Methodology: 

1. Collect data from IRENA & the WB’s Open Data Library.
2. Reformat dataframes for them to coincide for merging.
3. Clean and engineer features to prepare data for analysis.
4. Preprocess all numerical, categorical and linguistic features using a single, efficient loop which gridsearches optimal text vectorization hyperparameters.
5. Loop through chosen classifier models’ gridsearch, storing best hyperparameters in a master dataframe.
6. Extract top 3 models with best accuracy, from master dataframe, to reveal their alternative performance (precision, recall) with confusion matrices and ROC curves.
7. Display Logistic Regression’s coefficients’ feature importances.
8. Attempt to improve model: play with adding more features and elaborate feature engineering to rerun above workflow from step 2-7.

## Data collection:
**IRENA’s website**

Transformed downloaded excel file into a CSV and uploaded it to a pandas dataframe in a Jupyter Notebook. 

**Obtained features:** 

Country, Project, Date, Investor, Technology, Asset Class, Invested USDM (millions of dollars).

**World Bank Open Data Library**

Picked various economic indicators as additional features to explore whether they weighed into studied RE projects’ received investments from IDAs. 

**Obtained features:** 

GDP growth as annual percent, GNI growth as annual percent, Adjusted Net National Savings in USD, Adjusted Savings including Energy depletion in USD, Adjusted Savings including CO2 damage in USD, Adjusted Savings including Particulate Damage as percent of GNI, Unemployment rate as percent of Labor Force.
(cf. [Glossary](https://github.com/ValentineRlt/Capstone-Project/blob/master/Glossary.txt) for measures’ definitions)

Upon [the second round of analysis](https://github.com/ValentineRlt/Capstone-Project/blob/master/4.%20Modelling%2C%20trial%202.ipynb) (step 8), I added the Forest Area as percent of Land and Natural Resource Rents as percent of GDP. In addition, I engineered the GDP, GNI and Net National Savings, creating new features revealing how each measure had evolved from the year prior to the projects’ funding. Operations are detailed in the [Modelling, trial 1 notebook part 2.4.](https://github.com/ValentineRlt/Capstone-Project/blob/master/3.%20Modelling%2C%20trial%201.ipynb). 

## Data Prep:

#### Dataset Joining:
_(cf. “[WB Data Cleaning & Merging](https://github.com/ValentineRlt/Capstone-Project/blob/master/2a.%20WB%20Data%20Cleaning%20and%20Merging.ipynb)" notebook)_

In preparation for subsequent merging, I created a function to transform datasets collected from the WB platform in order to have them correspond with IRENA’s format. The function includes the renaming of columns and features, data type conversion and concatenation to the overarching WB feature dataset. Upon completion, I merged the result onto IRENA’s countries and date feature, complementing each project observation with data about its country’s socio-economic conjuncture.
	
#### Data Cleaning:
Renamed columns; converted data into correct type (N.B. Date and Invested Sum); imputed missing values with the respective columns’ mean.

#### Feature Engineering:
In the [model’s second run](https://github.com/ValentineRlt/Capstone-Project/blob/master/4.%20Modelling%2C%20trial%202.ipynb), I added three new categorical columns inferred from the GDP, GNI & Adjusted Net National Savings. By comparing the measure to its previous year’s value, this addition informs us of whether it had grown, slowed or stayed the same hence my naming it the measure’s “fluc” (short for fluctuation). 

#### Target Preparation:
Created four classes defined by each of the invested sum’s quartiles:
  * Class 0 : first quartile of investment, 25% of projects having received the smallest investments.
  * Class 1 : second quartile,  25% of projects having received investments greater than class 0’s maximum and flooring at the median amount.
  * Class 2 : third quartile, 25% of projects having received an investments greater or equal to the median amount and flooring at the fourth quartile’s.
  * Class 3 : fourth quartile, 25% of projects having received the greatest investments.

Whilst the target lends itself to a regression model, experimenting with it returned poor results, possibly due to the dataset’s relatively small size (7490 rows upon cleaning). This alternative seems a good compromise as it nonetheless provides an interesting perspective on the distribution of IDA’s RE investments.

#### Train and Test Set set-up: 
Early data visualizations having revealed a notable evolution of investments’ scales between 2000 and 2017, I decided to split the data chronologically. I used all data prior to 2015 as the training set and that funded between 2015 - 2017 as the test set. Value counts demonstrate that these two years account for about 20% of the total observations, thus emulating the dimensions of a conventional train_test_split(). 

## Modelling:

#### Baseline
  * Class 0  =  0.250104
  * Class 1  =  0.249896
  * Class 2  = 0.249896
  * Class 3 = 0.250104

#### Model 

  * After stemming and lemmatizing the “Project Description” column, I was able to simultaneously gridsearch the term frequency inverse function’s [tfidf] optimal hyperparameters (nb max_depth, max_features, min_features, n_grams), Hot-Encode categorical features and scale numerical features to return a “ready-to-model” sparse matrix (courtesy of tfidf).
  * Created loops to run through the following models and their parameter grids. Best results were stored in an overarching dataframe for future use.
  * Ran this loop in four batches: the first included all classifiers which admit of parameter grids: KNearestNeighbors, Logistic Regression [LR], Random Forest Classifier [RFC], AdaBoost, GBoost. The second covered both Naive Bayes’ models which do not need parameter-grids for gridsearch. The last two ran Support Vector Machine’s Classifier [SVC] and a Neural Network respectfully. Given the considerable time their processing requires (over 10 hours each), doing them separately made this workflow more manageable. 
  * I extracted the three models with highest accuracy to investigate their alternative measures by drawing their classification report, confusion matrix, ROC and Precision Recall Curve. 
  * The three top models were the RFC (accuracy 0.5252), LR (accuracy 0.5235) and SVC (accuracy 0.5126). 
  * Plotted LR’s coefficients’ feature importances to derive insight into what features most weigh on projects’ received investment’s sum. 

## Results:

#### Initial Gridsearch 
Interestingly, while the RFC scored a marginally better accuracy than both LR and SVC, the latter two both proved more useful. Firstly, the possibility to extract LR’s coefficients make it a more insightful tool than RFC by virtue of providing information about the model’s prediction process. Secondly, SVC’s confusion matrix demonstrated a significantly better ability to differentiate all four classes with the following true positive rates:
			class 0 : 0.8 	class 1 : 0.75		class 2 : 0.7		class 3 : 0.93
Contrast with **RF**’s and **LR**’s respectively:
			class 0 : 0.71 		class 1 : 0.71		class 2 : 0.41		class 3 : 0.92
			class 0 : 0.66		class 1 : 0.61		class 2 : 0.44		class 3 : 0.9
			
It is worth noting all three models do a better job at correctly identifying classes 0 and 3 and are more confused when it comes to the middle classes, 1 and 2. This might simply be the result of both those classes being less distinguishable overall. Indeed, while class 0 is composed of all investments lesser than $47,000 and class 3 all investments greater than $15 million, class 1 and 2 have to sift through a swathe of projects receiving investments spread between $47,000 and $544,000 and between $544,000 and $15million, respectively. It is likely easier to identify between a project belonging to either extreme of the funding spectrum rather than that in the middle.  Greater amounts of data to train the data on, could surely improve these middle scores. 

Ultimately, were I to help a RE project enhance their funding application (this model’s best purpose), I would do well to hone in on the recall score. Recall informs of how good the model is at predicting the positive class when the actual outcome is positive whereas precision, speaks of the model’s ability to identify only relevant features. For the sake of thoroughness, I therefore ought to ensure my model correctly identifies which features weigh in or out of favour of the project’s acquiring a certain fork of investment. Maximizing the number of true positives would enhance my credibility. Although false positives wouldn’t be too hazardous.
This said, neither the ROC nor the Precision-Recall [PR] curves return particularly good trade-offs. Still, LR boasts the best PR micro-average and all three curves further affirm LR’s greater ability at identifying “book-ending” classes.

#### Top terms that increase belonging to class 0: 
  * 'Diversos proyectos' - Possibly indicative that many smaller scale projects are based in South America 
#### Top terms and features that decrease belonging to class 0: 
  * 'Aggregated activities’
  * Net National Savings being the same as the year before
  * Non Export Credits & Credit lines as asset classes
  * A project being ‘Chile’ based
  * Being financed by the ‘IADB’ (Inter-American Development Bank, Latin America-based).

#### Top terms and features that decrease belonging to class 1: 
_(no positively influencing feature found)_
  * Important overlap with class 0, with the following additions: 
  * Net National Savings being the same as the year before and having slowed since
  * 'Access', ‘africa’, ‘apl’ figuring in the project description

#### Top terms and features that increase belonging to class 2:
  * ‘Flow’ and ‘installation solar’ figuring in the project description
  * Asset class being a ‘Credit line’
  * Top terms and features that decrease belonging to class 2:
  * Overlap with two previous classes with the additions:
  * GNI staying the same as the previous year 
  * Project technologies being ‘wind’ or ‘solar’ energy

#### Top terms and features that increase belonging to class 3:
  * Net National Savings staying the same or slowing from the previous year
  * Investors being the IADB,  OPIC or KEXIM (cf. [Glossary](https://github.com/ValentineRlt/Capstone-Project/blob/master/Glossary.txt) for full names)
  * Being ‘Chile’ based
  * Adjusted Savings including CO2 damage
  * GDP having slowed since the previous year
#### Top terms and features that decrease belonging to class 3:
  * GNI having stayed the same as previous year
  * Technology being solar energy or geothermal energy
  * Asset class being an insurance bond
  * GDP having grown from the previous year
  * Words ‘aggregated activities’, ‘access’, ‘Africa’ and ‘Bagasse’ (sugarcane or sorghum stalked juice, basis for biofuel) figuring in the project description



## Analysis

*These findings were relevant with certain researchable facts about the evolution of RE over the past two decades. For instance, Chile’s RE notably increased threefold between 2013-2017<sup>3</sup> therefore its weighing positively on the belonging to class 3 seems to follow.*
However, there seems to be an inconsistency with features predicting class 2, as the instance of ‘solar energy’ pertains to both an increasing and decreasing factor. This discrepancy echoes the below mentioned analysis’ limitations. 

 **Drawbacks to this analysis:** 

  * The model’s performance would likely benefit from a greater amount of observations to train on. 
  * Important to bear in mind this data only relays information on IDAs’ investments. The image it depicts is therefore biased and incomplete. Indeed, whilst China and the US are two renowned actors and investors of the field, they are not given such a position in these observations. Furthermore, given how politicized IDA’s agendas tend to be, observed results may simply be a result of policies and political goals rather than strategic operations aimed at earnestly advancing a Sustainable future.
  * One could interpret the reception of certain sums of investment as flags of the project’s success, the country’s successful implementation of sustainable energy sources, or even the country’s overall transition to a greener economy. Yet this assumption is unsound as the amount of investment does not guarantee either the project’s coming to commercial fruition, nor its positive impact on the environment. For instance, constant technological improvement may result in the decrease in RE technologies’ cost. It may therefore even be that the total invested sum increasingly decorrelates with the technologies’ effective implementation. Next, a very costly technology may initially attract huge amounts and still prove ineffective. Therefore, the initial invested sum does not seem the best feature by which to measure countries’ RE transition. 
  
<sup>3</sup> _‘Is a Renewable Energy Underway in Chile?’ - https://bit.ly/ChileRE_ 
	
## Future Directions
  * Refine NLP performed on the “Project Description” feature by appealing to a more elaborate initial normalization preprocessing.
  * Change the classification’s threshold based on the model’s ROC curves’ indication of optimal trade-off. 
  * Collect more data on the topic, possibly focussing on a single country or region to perform a time-series analysis.
  * Extend the analysis to encompass a wider range of funding actors (i.e. Venture Capital, Private Equity…).
  * Retrieve given projects’ ROI and use as target.
  * Explore projects’ impact on the country’s (or region’s) overall carbon emission and fossil fuel consumption and add as feature or target. 
  * Building on these improvement steps, work towards the development of a model to support the decision-making of investors’ keen to bridge financial and environmental profit. 
	
## Conclusion:

This project proved an interesting and informative enquiry into the dynamics and motivations of IDA’s finance flows with regards RE Projects. 

However, the collected observations were yet too few to elaborate an accurate prediction system. While it may be that constant technological innovations and changing environmental dispositions result in changing patterns of investment, my model and its features surely need improvement.

My next step is to find data on these projects’ ROI to replace my present target variable(total invested sum), which would in turn become another predictor. I will also complement the sources of investment with non-IDA actors. Besides improving my model’s performance, this will quite surely make it a more useful object of reference.
