# Predicting the Poverty Rate with ACS 5Yr Variables
MSDS 692 Final Project

Final presentation video: https://youtu.be/9nSMkF7D_mU

## Project Overview
Could we predict the poverty rate of different geographies in the United States by excluding income levels and instead utilizing other demographic variables?

This project explores the data from the U.S. Census’ American Community Survey (ACS-5yr) to find correlations between the poverty rate and other variables. We will explore data both at the state and county level, and find the best predictors and models. These could then be used to find the poverty rate in counties where the U.S. Census does not collect poverty data each year. 

## EDA
### Principle Component Analysis

Principle Component Analysis (PCA) is a technique reducing a dataset's dimensions by extracting features that best represent the variance in that dataset. PCA is helpful when working with data that have many variables and you are not certain which are the most important.
```{r}
#perform Principle Component Analysis on State data
ACSstate.pca <- prcomp(ACSstate, center = TRUE,scale. = TRUE)

#Visualize the culminative variance of the different principle components
fviz_eig(ACSstate.pca, main = "Scree plot of PCA for State data")
```

![Scree Plot of State PCA](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Scree_of_State_PCA.PNG)
      
The first Principle Component comprises 41% of the variance in the Data. The first 5 principle compenents comprise 81% of the data. Overall it seems that these compeonents do a strong job of explaining the data. 

Next, we want to visualize how the features are related to eachother. Plotting the PCA allows you to visulaize multidementional data in just two dimentions and see how the data is grouped and potentially correllated.
```{r}
# Plotting PC1 and PC1 of state data
ggbiplot(ACSstate.pca)
```
![Scatter Plot of State PCA](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/PCA%20of%20State%20Data.PNG)

      
Although it is not the clearest, we can see that the poverty statistics are all grouped together. They are also overlapping other variables, including Benefits SNAP, Unemploymnet Rate, Income with Social Security, annual income under $10K, and Public Health Coverage. This suggests there is a positive correlcation between the poverty variables and the other variables. Opposite from these are other varialble that would be negatively correlated, including the Employment Rate, Salaried workers, in armed services, work from home, and earning 75K to 100K. 

## Correlation Analysis
Correlation analysis allows us to determine the association/relationship between different variables. When variables are positively correlated, they both increase or decrease together. WHen variables are negatively correlated, when one increases the other decreases. When analyzing a correlation, when the value approaches 1, there is a very strong positive correlation. When the value approaches -1, there is a very strong negative correlation. When the value is close to 0, there is no correlation and the two variables are independent from each other. 

For this project, we are interested finding in both strong positive and strong negative correlations to create our predictive model.

```{r}
#Find correlation values between the Poverty Rate and each of the other variables
PovertyCorrelations <- cor(acsNOincome$Poverty_All_People, acsNOincome)

#create dataframe of results
PovCorr_Trans <- as.data.frame(t(as.matrix(PovertyCorrelations)))
colnames(PovCorr_Trans) <- c("correlation")

#Order the complete list of variables by correlation value, so to easily identify strongest correlations
PovCorr_Trans[order(-PovCorr_Trans$correlation), , drop =FALSE]
```
![Correlation Table of Poverty ~ ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Correlation%20Table%20Complete.PNG)

The Variables that have the strongest positive correlation with the Poverty Rate are Benefits_SNAP,	Income_Supplemental_Security, Unemployment_Rate,	With_Public_Health_Coverage, and Not_in_labor_force. 

The variables that havew the strongest negative correlation with the Poverty Rate are With_private_health_insurance,  Not_Labor_Market_with_Private_Health_Insurance, Employment_Rate, Females_Employment_Rate, and Employed_With_Private_Health_Insurance. 	

We can visualize these variables in a correlation matrix. 
```{r}
#Create Data Table with Strongest Correlations with Poverty Rate
ACScorrelations <-ACS_5_Pop_Counties_Clean[1:3088 , c(83, 56, 54, 8, 59, 58, 71, 5, 10)]

#Correlation Matrix
CorrMatrix <- cor(ACScorrelations)
corrplot(CorrMatrix, method = "circle", type = "lower")
```

![Correlation Matrix of Poverty ~ ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Correlation%20Matrix%20Short.PNG)

## Regression Analysis
### Linear Modelling

Linear regression is a basic type of predictive analysis, which can model the relationship between two variales (one dependent and the other explanitory). From looking at the correlations scatterplots, it is clear that a simple linear regression can be created for these variables.


![Linear Model for SNAP Benefits ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/LM%20SNAP.PNG)

![Linear Model for SNAP Benefits ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/LM%20SNAP%20plot.PNG)


```{r}
#scatterplot of linear model
ggplot(ACScomplete, aes(x= Benefits_SNAP, y=Poverty_All_People)) + 
  geom_point() + stat_smooth(method = "lm", col = "red")

```
### Testing Linear Models
While it is good to have linear models for data that we alraedy have, we will eventually need to use the model to predict new data. We can use cross-validation to measure how well a model perform in predicting data it has not yet seen. To do this, we can divide the data we have into two segments: one for training the model and the other for testing it. 

```{r}
# Split the data into training and test set

set.seed(123)
training.samples <- acsNOincome$Poverty_All_People %>%
  createDataPartition(p = 0.8, list = FALSE)
train.data  <- acsNOincome[training.samples, ]
test.data <- acsNOincome[-training.samples, ]

```
Next, We create a linear model for the variables using only the training data set, and use it to make predictions. We then can test how well these predictions do against the actual data. 

```{r}

# Build the model for SNAP Benefits
SNAPmodel <- lm(Poverty_All_People ~ Benefits_SNAP, data = train.data)

# Make predictions with SNAP Model and compute the R2, RMSE and MAE
predictions1 <- SNAPmodel %>% predict(test.data)
data.frame( R2 = R2(predictions1, test.data$Poverty_All_People),
            RMSE = RMSE(predictions1, test.data$Poverty_All_People),
            MAE = MAE(predictions1, test.data$Poverty_All_People))


```

![Training and Testing SNAP LM ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Testing%20LM%20for%20SNAP.PNG)

![Training and Testing Private Health LM](https://github.com/4charliegardner/msds692_final_project/blob/master/Images/Testing%20LM%20for%20SNAP.PNG)



## Conclusion

## References
