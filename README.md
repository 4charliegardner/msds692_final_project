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

![Scree Plot of State PCA](https://github.com/msds692_final_project/Images/Scree_of_State_PCA.PNG)
      
The first Principle Component comprises 41% of the variance in the Data. The first 5 principle compenents comprise 81% of the data. Overall it seems that these compeonents do a strong job of explaining the data. 

Next, we want to visualize how the features are related to eachother. Plotting the PCA allows you to visulaize multidementional data in just two dimentions and see how the data is grouped and potentially correllated.
```{r}
# Plotting PC1 and PC1 of state data
ggbiplot(ACSstate.pca)
```
![Scree Plot of State PCA](https://github.com/msds692_final_project/Images/PCA_of_State_Data.PNG)
      
Although it is not the clearest, we can see that the poverty statistics are all grouped together. They are also overlapping other variables, including Benefits SNAP, Unemploymnet Rate, Income with Social Security, annual income under $10K, and Public Health Coverage. This suggests there is a positive correlcation between the poverty variables and the other variables. Opposite from these are other varialble that would be negatively correlated, including the Employment Rate, Salaried workers, in armed services, work from home, and earning 75K to 100K. 

## Correlation Analysis
Correlation analysis allows us to determine the association/relationship between different variables. When variables are positively correlated, they both increase or decrease together. WHen variables are negatively correlated, when one increases the other decreases. When analyzing a correlation, when the value approaches 1, there is a very strong positive correlation. When the value approaches -1, there is a very strong negative correlation. When the value is close to 0, there is no correlation and the two variables are independent from each other. 

For this project, we are interested finding in both strong positive and strong negative correlations to create our predictive model.

## Implementation and Analysis

## Conclusion

## References
