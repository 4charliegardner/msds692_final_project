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

#prediction error rate for SNAP Model
RMSE(predictions1, test.data$Poverty_All_People)/mean(test.data$Poverty_All_People)
```

![Training and Testing SNAP LM ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Testing%20LM%20for%20SNAP.PNG)

![Prediction Error for SNAP LM ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Prediction%20Error%20for%20LM%20SNAP.PNG)

![Training and Testing Private Health LM](https://github.com/4charliegardner/msds692_final_project/blob/master/Images/Testing%20LM%20for%20SNAP.PNG)

![Prediction Error for Private Health LM ](https://raw.githubusercontent.com/4charliegardner/msds692_final_project/master/Images/Prediction%20Error%20for%20LM%20Private%20Healthcare.PNG)

When evaluating the predictions, we look at the R2 (the higher the better), RMSW (the lower the better), and MAE (the lower the better). We  can also cacluate the prediction error rate by dividing the RMSE by the average value of the outcome variable (and obviously the smaller the better).

While Snap Benefits had a higher correlation, the linear model created from Private Healthcare is comparable since they altnerate better scores when assessing the R-Squated, Root Mean Squared Error, and Mean Absolute Error values. And the prediction error is basically the same and fairly low. In terms of linear models, either Having Private Health Insurance or Snap Benefits could serve as a good indicator and predictor for the poverty rate. 

## Multiple Regression Linear Models 
### StepAIC Analysis
StepAIC reduces the dimensions of a dataset by removing and adding features to make the best fit. This can mean that it retains its multi-dimensionality, and make it more difficult to represent in a plot. But it also can be very useful in creating a linear model to make predictions. 

```{r}
#Create linear model of poverty rate and all other variables
acsNOincome.lm <- lm(Poverty_All_People ~., data = acsNOincome)

#Use AIC stepwise analysis to remove features that do not add to the predictive capacability of the model
AICmodelACS <- stepAIC(acsNOincome.lm, direction = "both", trace = FALSE)

#Show coefficients of new model and their significance to the model
summary(AICmodelACS)

```
{results:

Call:
lm(formula = Poverty_All_People ~ Not_in_labor_force + Unemployment_Rate + 
    Females_In_labor_force + Females_Employment_Rate + Commute_Drive_Alone + 
    Commute_Carpool + Commute_Public_Trans + Commuting_Other_Means + 
    Management_business_science_and_arts_occupations + Sales_and_office_occupations + 
    Natural_resources_construction_and.maintenance_occupations + 
    Industry_Agriculture_forestry_fishing_hunting_and_mining + 
    Industry_Construction + Industry_Manufacturing + Industry_Wholesale_trade + 
    Industry_Transportation_warehousing_utilities + Industry_Information + 
    Industry_Finance_insurance_RealEstate_rental_leasing + Industry_Professional_scientific_and_administrative_waste_management + 
    Industry_Arts_Entertainment_Recreation_Food + Industry_Other_Services + 
    Industry_Public_Administration + WorkerClass_PrivateWage_SalaryWorkers + 
    WorkerClass_SelfEmployed + Income_Earned + Income_Social_Security + 
    Income_Retirement + Income_cash_public_assistance + Benefits_SNAP + 
    With_health_insurance + With_private_health_insurance + With_Public_Health_Coverage + 
    Children_No_health_insurance + Employed_With_Private_Health_Insurance + 
    Employed_With_Public_Health_Insurance + Unemployed_With_Public_Health_Insurance + 
    Unemployed_No_health_insurance + Not_Labor_Market_with_Health_Insurance + 
    Not_Labor_Market_with_Public_Health_Insurance + Not_Labor_Market_without_Health_Insurance, 
    data = acsNOincome)
Residuals:
     Min       1Q   Median       3Q      Max 
-11.6652  -1.4140  -0.1283   1.2956  14.1923 

Coefficients:
                                                                       Estimate Std. Error t value Pr(>|t|)    
(Intercept)                                                          -4.197e+03  1.742e+03  -2.410 0.016016 *  
Not_in_labor_force                                                    9.950e-02  1.624e-02   6.128 1.01e-09 ***
Unemployment_Rate                                                     7.509e-02  3.995e-02   1.880 0.060246 .  
Females_In_labor_force                                                9.725e-02  6.316e-02   1.540 0.123740    
Females_Employment_Rate                                              -1.562e-01  6.379e-02  -2.449 0.014381 *  
Commute_Drive_Alone                                                  -4.943e-02  1.368e-02  -3.614 0.000306 ***
Commute_Carpool                                                      -7.205e-02  2.114e-02  -3.408 0.000662 ***
Commute_Public_Trans                                                 -4.114e-02  2.181e-02  -1.887 0.059301 .  
Commuting_Other_Means                                                -2.353e-01  3.820e-02  -6.159 8.30e-10 ***
Management_business_science_and_arts_occupations                     -9.607e-02  1.577e-02  -6.094 1.24e-09 ***
Sales_and_office_occupations                                         -4.441e-02  2.179e-02  -2.038 0.041643 *  
Natural_resources_construction_and.maintenance_occupations           -1.047e-01  2.734e-02  -3.829 0.000131 ***
Industry_Agriculture_forestry_fishing_hunting_and_mining             -1.426e-01  1.860e-02  -7.670 2.29e-14 ***
Industry_Construction                                                -2.363e-01  3.135e-02  -7.537 6.29e-14 ***
Industry_Manufacturing                                               -1.205e-01  1.446e-02  -8.333  < 2e-16 ***
Industry_Wholesale_trade                                             -1.861e-01  4.418e-02  -4.213 2.59e-05 ***
Industry_Transportation_warehousing_utilities                        -2.844e-01  2.575e-02 -11.048  < 2e-16 ***
Industry_Information                                                 -2.578e-01  6.102e-02  -4.225 2.46e-05 ***
Industry_Finance_insurance_RealEstate_rental_leasing                 -1.905e-01  3.349e-02  -5.689 1.40e-08 ***
Industry_Professional_scientific_and_administrative_waste_management -1.534e-01  2.266e-02  -6.770 1.54e-11 ***
Industry_Arts_Entertainment_Recreation_Food                          -3.651e-02  1.828e-02  -1.997 0.045931 *  
Industry_Other_Services                                              -1.117e-01  4.194e-02  -2.663 0.007785 ** 
Industry_Public_Administration                                       -3.570e-01  2.471e-02 -14.445  < 2e-16 ***
WorkerClass_PrivateWage_SalaryWorkers                                -1.544e-01  1.532e-02 -10.075  < 2e-16 ***
WorkerClass_SelfEmployed                                             -9.998e-02  2.719e-02  -3.677 0.000240 ***
Income_Earned                                                        -2.698e-01  2.016e-02 -13.383  < 2e-16 ***
Income_Social_Security                                               -3.399e-01  1.867e-02 -18.212  < 2e-16 ***
Income_Retirement                                                    -1.716e-01  1.551e-02 -11.062  < 2e-16 ***
Income_cash_public_assistance                                        -1.076e-01  3.178e-02  -3.385 0.000720 ***
Benefits_SNAP                                                         2.907e-01  1.619e-02  17.958  < 2e-16 ***
With_health_insurance                                                -1.514e-01  7.241e-02  -2.092 0.036565 *  
With_private_health_insurance                                        -2.819e-01  2.547e-02 -11.066  < 2e-16 ***
With_Public_Health_Coverage                                           1.464e-01  2.964e-02   4.939 8.26e-07 ***
Children_No_health_insurance                                         -5.160e-02  2.162e-02  -2.386 0.017082 *  
Employed_With_Private_Health_Insurance                                1.637e-01  2.892e-02   5.661 1.65e-08 ***
Employed_With_Public_Health_Insurance                                -6.765e-02  3.100e-02  -2.182 0.029170 *  
Unemployed_With_Public_Health_Insurance                              -1.357e-02  4.976e-03  -2.728 0.006416 ** 
Unemployed_No_health_insurance                                       -1.046e-02  4.465e-03  -2.342 0.019231 *  
Not_Labor_Market_with_Health_Insurance                                4.297e+01  1.742e+01   2.467 0.013679 *  
Not_Labor_Market_with_Public_Health_Insurance                        -8.078e-02  9.663e-03  -8.360  < 2e-16 ***
Not_Labor_Market_without_Health_Insurance                             4.291e+01  1.742e+01   2.464 0.013809 *  
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 2.419 on 3047 degrees of freedom
Multiple R-squared:  0.8634,	Adjusted R-squared:  0.8616 
F-statistic: 481.5 on 40 and 3047 DF,  p-value: < 2.2e-16
}

To use this model to calcuate/predict the poverty rate, you would use the same basic princple as a single regression. 
      Poverty Rate = y-intercept + x(coefficient of x) + z(coefficeint of z)....
Each prediction would require each explanitory variable, otherise you can not caclulate the y variable. 


We can use the glance() function to assess the quality of this AIC model with multiple regressions.

```{r}

#Assessing Linear Model created from the AIC Stepwise function
glance(AICmodelACS)%>%
  dplyr::select(adj.r.squared, sigma, AIC, BIC, p.value)

``` 
adj.r.squared <dbl> sigma <dbl> AIC <dbl> BIC <dbl> p.value <dbl>
1	0.8616028	2.419001	14261.69	14515.17	0

```{r}
#Prediction Error Rate for AIC Model
sigma(AICmodelACS)/mean(acsNOincome$Poverty_All_People)
```
[1] 0.1506062

## Conclusion 
### Which is the best Model?

After comparing the the linear models for Private Health Care (the best single line regression) and the multi-line regression model created with AIC, it is clear that the AIC model is a more accurate predictor.  It has a higher R2 value, a lower RMSE value, and a prediction error rate of only 15% compared to 23% error rate of the Private Health Insurance.

But even though AIC prevents over-fitting of a model, the multi-regression model is very complex and difficult to visualize in 2 dimensions. Overall, the linear model of Private Health Insurance does a strong job of predicting the poverty rate on its own. 

In either case, it is possible to predict the poverty rate of different geographies based upon different demographics, even if you don't have income levels data. 
