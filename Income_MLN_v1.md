Multivariate Classification
================

By default, R has only utilized one processor which is poor performance.

Multicore processing can effectively reduce running time. In this project, the running time of:

Single core: 28 minutes

Multi cores: 9 minutes

Almost three times faster!!!

``` r
# for speed up
library(doParallel) 
```

    ## Loading required package: foreach

    ## Loading required package: iterators

    ## Loading required package: parallel

``` r
# return number of cores in your computer, but keep 2 core free
no_cores <- detectCores() - 2
cl <- makeCluster(no_cores, type="PSOCK")  
# start parallel processing
registerDoParallel(cl)  
```

Import multi-csv files in a folder. The 8 csv files name are shown as below.

``` r
# Import Data -----------------------------------------------------------------------------------------------------
# path to folder that holds multiple .csv files (8 csv)
folder <- "C:/Users/user/Desktop/Data_Science/HK_Income/"
# create list of all .csv files in folder
file_list <- list.files(path=folder, pattern="*.csv") 
# read in each .csv file in file_list and create a data frame with the same name as the .csv file
for (i in 1:length(file_list)){
  assign(file_list[i], read.csv(paste(folder, file_list[i], sep='')))
}
file_list
```

    ## [1] "hh2018q1_20pct.csv" "hh2018q2_20pct.csv" "hh2018q3_20pct.csv"
    ## [4] "hh2018q4_20pct.csv" "pp2018q1_20pct.csv" "pp2018q2_20pct.csv"
    ## [7] "pp2018q3_20pct.csv" "pp2018q4_20pct.csv"

Data Processing: Combine quarterly datasets into 1 dataset and drop useless variable.

Noted that the data type of all variables is numeric. In fact, the data type should be factor instead of numeric.

``` r
# combine household income dataframe by rows
hh_2018 <- do.call("rbind", list(hh2018q1_20pct.csv, hh2018q2_20pct.csv, hh2018q3_20pct.csv, hh2018q4_20pct.csv))
# combine personal information dataframe by rows
pp_2018 <- do.call("rbind", list(pp2018q1_20pct.csv, pp2018q2_20pct.csv, pp2018q3_20pct.csv, pp2018q4_20pct.csv))
# merge hh & pp dataframe
hh_pp_2018 <- merge(x=hh_2018,y=pp_2018,by=c("Year_Quarter","Household_reference_no"),all.x = TRUE)
# drop useless Grossing_up_factor
hh_pp_2018$Grossing_up_factor.x <- NULL
hh_pp_2018$Grossing_up_factor.y <- NULL
# Worked_hours have too many level (91 levels), combine it
hh_pp_2018$Worked_hours <- as.integer(hh_pp_2018$Worked_hours)
hh_pp_2018$Worked_hours <- floor(hh_pp_2018$Worked_hours/10)
# display data structure
str(hh_pp_2018)
```

    ## 'data.frame':    46734 obs. of  26 variables:
    ##  $ Year_Quarter                    : int  20181 20181 20181 20181 20181 20181 20181 20181 20181 20181 ...
    ##  $ Household_reference_no          : num  1.1e+09 1.1e+09 1.1e+09 1.1e+09 1.1e+09 ...
    ##  $ Type_of_housing                 : int  3 3 3 3 3 2 3 3 3 1 ...
    ##  $ Tenure_of_accommodation         : int  3 3 3 3 3 2 1 1 1 3 ...
    ##  $ Number_of_persons_usually_living: int  3 3 3 2 2 1 1 2 2 2 ...
    ##  $ Monthly_rent                    : int  9 9 9 8 8 99 99 99 99 2 ...
    ##  $ Monthly_household_income        : int  10 10 10 9 9 3 10 11 11 3 ...
    ##  $ Relationship_to_household_head  : int  5 1 7 1 2 1 1 1 2 1 ...
    ##  $ Age                             : int  7 8 7 9 8 14 12 9 10 12 ...
    ##  $ Sex                             : int  1 2 2 1 2 1 1 2 1 1 ...
    ##  $ Educational_attainment          : int  6 7 6 7 8 3 8 7 4 2 ...
    ##  $ Marital_status                  : int  2 1 2 2 2 2 3 2 2 2 ...
    ##  $ Economic_activity_status        : int  1 1 8 1 1 6 1 1 1 1 ...
    ##  $ Whether_being_underemployed     : int  2 2 9 2 2 9 2 2 2 2 ...
    ##  $ Usual_place_of_work             : int  1 1 9 1 1 9 1 1 1 1 ...
    ##  $ Industry_for_employed           : int  5 18 99 22 22 99 18 13 18 8 ...
    ##  $ Industry_for_underemployed      : int  9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Pre_industry_for_unemployed     : int  9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Occupation_for_employed         : int  3 3 99 3 3 99 1 3 3 8 ...
    ##  $ Pre_occupation_for_unemployed   : int  99 99 99 99 99 99 99 99 99 99 ...
    ##  $ Worked_hours                    : num  7 3 99 3 4 99 5 4 4 4 ...
    ##  $ M_e_earnings_for_employed       : int  14 14 99 12 13 99 19 14 17 6 ...
    ##  $ M_e_earnings_for_underemployed  : int  9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Duration_of_unemployment        : int  9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Reasons_leaving_for_unemployed  : int  9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Whether_foreign_domestic_helper : int  2 2 9 2 2 9 2 2 2 2 ...

Convert variables type to factor.

``` r
# Convert variables type from int to factor
# return number of columns
nocol <- ncol(hh_pp_2018) 
for (i in 3:nocol){
  hh_pp_2018[,i] <- as.factor(hh_pp_2018[,i])
}
str(hh_pp_2018)
```

    ## 'data.frame':    46734 obs. of  26 variables:
    ##  $ Year_Quarter                    : int  20181 20181 20181 20181 20181 20181 20181 20181 20181 20181 ...
    ##  $ Household_reference_no          : num  1.1e+09 1.1e+09 1.1e+09 1.1e+09 1.1e+09 ...
    ##  $ Type_of_housing                 : Factor w/ 4 levels "1","2","3","4": 3 3 3 3 3 2 3 3 3 1 ...
    ##  $ Tenure_of_accommodation         : Factor w/ 6 levels "1","2","3","4",..: 3 3 3 3 3 2 1 1 1 3 ...
    ##  $ Number_of_persons_usually_living: Factor w/ 14 levels "1","2","3","4",..: 3 3 3 2 2 1 1 2 2 2 ...
    ##  $ Monthly_rent                    : Factor w/ 13 levels "1","2","3","4",..: 9 9 9 8 8 13 13 13 13 2 ...
    ##  $ Monthly_household_income        : Factor w/ 11 levels "1","2","3","4",..: 10 10 10 9 9 3 10 11 11 3 ...
    ##  $ Relationship_to_household_head  : Factor w/ 15 levels "1","2","3","4",..: 5 1 7 1 2 1 1 1 2 1 ...
    ##  $ Age                             : Factor w/ 14 levels "1","2","3","4",..: 7 8 7 9 8 14 12 9 10 12 ...
    ##  $ Sex                             : Factor w/ 2 levels "1","2": 1 2 2 1 2 1 1 2 1 1 ...
    ##  $ Educational_attainment          : Factor w/ 8 levels "1","2","3","4",..: 6 7 6 7 8 3 8 7 4 2 ...
    ##  $ Marital_status                  : Factor w/ 3 levels "1","2","3": 2 1 2 2 2 2 3 2 2 2 ...
    ##  $ Economic_activity_status        : Factor w/ 8 levels "1","2","3","4",..: 1 1 8 1 1 6 1 1 1 1 ...
    ##  $ Whether_being_underemployed     : Factor w/ 3 levels "1","2","9": 2 2 3 2 2 3 2 2 2 2 ...
    ##  $ Usual_place_of_work             : Factor w/ 3 levels "1","2","9": 1 1 3 1 1 3 1 1 1 1 ...
    ##  $ Industry_for_employed           : Factor w/ 24 levels "1","2","3","4",..: 5 18 24 22 22 24 18 13 18 8 ...
    ##  $ Industry_for_underemployed      : Factor w/ 8 levels "1","2","3","4",..: 8 8 8 8 8 8 8 8 8 8 ...
    ##  $ Pre_industry_for_unemployed     : Factor w/ 9 levels "1","2","3","4",..: 9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Occupation_for_employed         : Factor w/ 10 levels "1","2","3","4",..: 3 3 10 3 3 10 1 3 3 8 ...
    ##  $ Pre_occupation_for_unemployed   : Factor w/ 11 levels "1","2","3","4",..: 11 11 11 11 11 11 11 11 11 11 ...
    ##  $ Worked_hours                    : Factor w/ 11 levels "0","1","2","3",..: 8 4 11 4 5 11 6 5 5 5 ...
    ##  $ M_e_earnings_for_employed       : Factor w/ 21 levels "1","2","3","4",..: 14 14 21 12 13 21 19 14 17 6 ...
    ##  $ M_e_earnings_for_underemployed  : Factor w/ 9 levels "1","2","3","4",..: 9 9 9 9 9 9 9 9 9 9 ...
    ##  $ Duration_of_unemployment        : Factor w/ 6 levels "1","2","3","4",..: 6 6 6 6 6 6 6 6 6 6 ...
    ##  $ Reasons_leaving_for_unemployed  : Factor w/ 4 levels "1","2","3","9": 4 4 4 4 4 4 4 4 4 4 ...
    ##  $ Whether_foreign_domestic_helper : Factor w/ 3 levels "1","2","9": 2 2 3 2 2 3 2 2 2 2 ...

Classification of Multi-Class Response Variable:

There are 11 levels in response variable "Monthly\_household\_income"

Code Description

01 &lt; 4,000

02 4,000 - 5,999

03 6,000 - 7,999

04 8,000 - 9,999

05 10,000 - 14,999

06 15,000 - 19,999

07 20,000 - 24,999

08 25,000 - 29,999

09 30,000 - 39,999

10 40,000 - 49,999

11 &gt; 50,000

``` r
summary(hh_pp_2018$Monthly_household_income)
```

    ##     1     2     3     4     5     6     7     8     9    10    11 
    ##  1253  1114  1023  1228  3396  3829  4225  3556  6944  5270 14896

14896 observations belong to income group 11.

Construct Classification Model --- Multinomial Logistic Regression (MNL)

Use 23 factors (age,gender,working hours,etc) to predict the household belong to which income group.

To avoid overfitting problem, 2-fold cross-validation had been used.

``` r
# Cross-vaildation setting (2 fold CV)--------------------------------------------------------------------
require(caret)
```

    ## Loading required package: caret

    ## Loading required package: lattice

    ## Loading required package: ggplot2

``` r
require(e1071)
```

    ## Loading required package: e1071

``` r
set.seed(12345)
control <- trainControl(method="repeatedcv", number=2, repeats=2)

# Classification model(1) Multinomial Logistic Regression (MNL)--------------------------------------
multinom_model <- train(Monthly_household_income ~.-Year_Quarter-Household_reference_no,
                data=hh_pp_2018, 
                method="multinom",
                MaxNWts = 3000,
                metric = "Accuracy",
                trControl = control,
                trace = FALSE)
# Model summary
multinom_model
```

    ## Penalized Multinomial Regression 
    ## 
    ## 46734 samples
    ##    25 predictor
    ##    11 classes: '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (2 fold, repeated 2 times) 
    ## Summary of sample sizes: 23369, 23365, 23367, 23367 
    ## Resampling results across tuning parameters:
    ## 
    ##   decay  Accuracy   Kappa    
    ##   0e+00  0.4161958  0.2701590
    ##   1e-04  0.4162814  0.2702755
    ##   1e-01  0.4171909  0.2695507
    ## 
    ## Accuracy was used to select the optimal model using the largest value.
    ## The final value used for the model was decay = 0.1.

Measure Model Performance: Confusion Matrix

Confusion matrix shows that the classification model accuracy = 0.4291, it means that predicted income group of almost 43% observations consistent with their true income group.

However, how about the mis-classification result of the rest (100% - 43%) = 57% observations?

``` r
# Fit MNL in data (raw refer to return the predicted income group instead of prob)
hh_qq_MNL <- hh_pp_2018
hh_qq_MNL$Pre_Income_MNL <- predict(multinom_model, newdata = hh_qq_MNL, "raw")
# Measure model performance
confusionMatrix(hh_qq_MNL$Monthly_household_income, hh_qq_MNL$Pre_Income_MNL)
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction     1     2     3     4     5     6     7     8     9    10
    ##         1    785   117     8     6    97    42    11     0    40     1
    ##         2    243   457    23    29   168    25     7     0    37     1
    ##         3    253   133   101    28   282    48     8     2    56     0
    ##         4    180    57    60   212   387   129    38     2    67     0
    ##         5    288    65    54   109  1188   472   152    32   540    11
    ##         6    220    23    33    84   650   965   171    31   865    20
    ##         7    139    18    26    53   521   465   587    39  1061    71
    ##         8     73     4    11    22   298   342   240   177   954    61
    ##         9     88     6    16    20   320   481   386   130  2133   136
    ##         10    74     3     3     6   112   158   182    69  1048   226
    ##         11    70     9     9     5    60   110   135    70   992   213
    ##           Reference
    ## Prediction    11
    ##         1    146
    ##         2    124
    ##         3    112
    ##         4     96
    ##         5    485
    ##         6    767
    ##         7   1245
    ##         8   1374
    ##         9   3228
    ##         10  3389
    ##         11 13223
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.4291          
    ##                  95% CI : (0.4246, 0.4336)
    ##     No Information Rate : 0.5176          
    ##     P-Value [Acc > NIR] : 1               
    ##                                           
    ##                   Kappa : 0.2769          
    ##                                           
    ##  Mcnemar's Test P-Value : <2e-16          
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity           0.32532 0.512332 0.293605 0.369338  0.29096  0.29812
    ## Specificity           0.98944 0.985668 0.980125 0.977990  0.94823  0.93416
    ## Pos Pred Value        0.62650 0.410233 0.098729 0.172638  0.34982  0.25202
    ## Neg Pred Value        0.96420 0.990465 0.994684 0.992045  0.93320  0.94705
    ## Prevalence            0.05163 0.019087 0.007361 0.012282  0.08737  0.06926
    ## Detection Rate        0.01680 0.009779 0.002161 0.004536  0.02542  0.02065
    ## Detection Prevalence  0.02681 0.023837 0.021890 0.026276  0.07267  0.08193
    ## Balanced Accuracy     0.65738 0.749000 0.636865 0.673664  0.61960  0.61614
    ##                      Class: 7 Class: 8 Class: 9 Class: 10 Class: 11
    ## Sensitivity           0.30621 0.320652  0.27371  0.305405    0.5467
    ## Specificity           0.91883 0.926833  0.87645  0.890334    0.9258
    ## Pos Pred Value        0.13893 0.049775  0.30717  0.042884    0.8877
    ## Neg Pred Value        0.96871 0.991315  0.85775  0.987604    0.6556
    ## Prevalence            0.04102 0.011812  0.16675  0.015834    0.5176
    ## Detection Rate        0.01256 0.003787  0.04564  0.004836    0.2829
    ## Detection Prevalence  0.09041 0.076090  0.14859  0.112766    0.3187
    ## Balanced Accuracy     0.61252 0.623743  0.57508  0.597869    0.7362

For those 57% mis-classification result, we can measure the over/under estimation of their income group.

A new variable "Deviation" has been created to calculate the different between predicted and real income group.

if "Deviation" is positive number, the predicted income had been over-estimate.

In the other hand, if "Deviation" is negative number, the predicted income had been under-estimate.

For example, if "Deviation" is +2, it means that the predicted income is more over-estimate 2 classes than real income.

``` r
# Convert datatype from factor to numeric
Pred <- as.numeric(hh_qq_MNL$Pre_Income_MNL)
Real <- as.numeric(hh_qq_MNL$Monthly_household_income)

# Different between predicted income group and real income group
Deviation <- Pred - Real
```

Let see the frequency of deviation (+ - ) between predicted income group and real income group.

``` r
# for freq table
library(descr) 
freq(Deviation)
```

![](Income_MLN_v1_files/figure-markdown_github/unnamed-chunk-9-1.png)

    ## Deviation 
    ##       Frequency  Percent
    ## -10          70   0.1498
    ## -9           83   0.1776
    ## -8          100   0.2140
    ## -7           87   0.1862
    ## -6          225   0.4814
    ## -5          491   1.0506
    ## -4          972   2.0799
    ## -3         1362   2.9144
    ## -2         2758   5.9015
    ## -1         3291   7.0420
    ## 0         20054  42.9109
    ## 1          5716  12.2309
    ## 2          4981  10.6582
    ## 3          2602   5.5677
    ## 4          1937   4.1447
    ## 5           896   1.9172
    ## 6           552   1.1812
    ## 7           133   0.2846
    ## 8           153   0.3274
    ## 9           125   0.2675
    ## 10          146   0.3124
    ## Total     46734 100.0000

Plot the frequency of deviation (+ - ) between predicted income group and real income group.

``` r
plot(freq(Deviation))
```

![](Income_MLN_v1_files/figure-markdown_github/unnamed-chunk-10-1.png)![](Income_MLN_v1_files/figure-markdown_github/unnamed-chunk-10-2.png)

![](https://raw.githubusercontent.com/sin121212/HK-monthly-household-income-2018/master/plot.png)

Feature Selection:

Apart from the accuracy, researcher may want to know which independence variables are more useful or relevant for the income group.

The top 5 important features are Number\_of\_persons\_usually\_living, Monthly\_rent, M\_e\_earnings\_for\_employed, Tenure\_of\_accommodation and Age.

``` r
var_imp <- varImp(multinom_model)
feature <- var_imp$importance
feature$Var_Name <- rownames(feature)
rownames(feature) <- NULL
feature$Variable <- gsub("[[:digit:]]","",feature$Var_Name)
# mean by group
feature_mean <- aggregate(feature$Overall, list(feature$Variable), mean)
# sorting
feature_mean <- feature_mean[order(-feature_mean$x),]
# top 5 important features
head(feature_mean,5)
```

    ##                             Group.1        x
    ## 11 Number_of_persons_usually_living 76.97121
    ## 10                     Monthly_rent 45.94688
    ## 7         M_e_earnings_for_employed 33.65011
    ## 18          Tenure_of_accommodation 24.40793
    ## 1                               Age 23.91474

Build Small Size Model:

Let try to fit a new MNL model with those top 5 importance variables instead of all 23 variables.

``` r
multinom_model_5var <- train(Monthly_household_income ~Number_of_persons_usually_living+Monthly_rent+M_e_earnings_for_employed+Tenure_of_accommodation+Age,
                data=hh_pp_2018, 
                method="multinom",
                MaxNWts = 3000,
                metric = "Accuracy",
                trControl = control,
                trace = FALSE)
# Model summary
multinom_model_5var
```

    ## Penalized Multinomial Regression 
    ## 
    ## 46734 samples
    ##     5 predictor
    ##    11 classes: '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (2 fold, repeated 2 times) 
    ## Summary of sample sizes: 23368, 23366, 23367, 23367 
    ## Resampling results across tuning parameters:
    ## 
    ##   decay  Accuracy   Kappa    
    ##   0e+00  0.4204113  0.2720160
    ##   1e-04  0.4204006  0.2719912
    ##   1e-01  0.4184534  0.2688313
    ## 
    ## Accuracy was used to select the optimal model using the largest value.
    ## The final value used for the model was decay = 0.

Fit new model into data and measure new model perfermance.

``` r
# Fit MNL in data (raw refer to return the predicted income group instead of prob)
hh_qq_MNL_5var <- hh_pp_2018
hh_qq_MNL_5var$Pre_Income_MNL <- predict(multinom_model_5var, newdata = hh_qq_MNL_5var, "raw")
# Measure model performance
confusionMatrix(hh_qq_MNL_5var$Monthly_household_income, hh_qq_MNL_5var$Pre_Income_MNL)
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction     1     2     3     4     5     6     7     8     9    10
    ##         1    789   129     7     4    86    41    21     0    56     5
    ##         2    251   465     5     3   209    20     9     0    39     0
    ##         3    266   152    82     4   319    33    18     0    72     1
    ##         4    180    74    21   159   446   158    62     0    52     0
    ##         5    306    85    44    77  1143   538   273    17   472    24
    ##         6    235    35    34    55   675   990   335    28   723    18
    ##         7    148    16    30    46   574   485   689    33   838    69
    ##         8     79     3    17    21   336   335   315   146   906    40
    ##         9    110     3    18    21   331   527   561    92  1919   113
    ##         10    81     3     4     5    95   209   272    45   940   175
    ##         11    95     8    14     6    56   167   250    53  1011   102
    ##           Reference
    ## Prediction    11
    ##         1    115
    ##         2    113
    ##         3     76
    ##         4     76
    ##         5    417
    ##         6    701
    ##         7   1297
    ##         8   1358
    ##         9   3249
    ##         10  3441
    ##         11 13134
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.4213          
    ##                  95% CI : (0.4169, 0.4258)
    ##     No Information Rate : 0.5131          
    ##     P-Value [Acc > NIR] : 1               
    ##                                           
    ##                   Kappa : 0.269           
    ##                                           
    ##  Mcnemar's Test P-Value : <2e-16          
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: 1 Class: 2 Class: 3 Class: 4 Class: 5 Class: 6
    ## Sensitivity           0.31063  0.47790 0.297101 0.396509  0.26768  0.28261
    ## Specificity           0.98950  0.98582 0.979745 0.976928  0.94694  0.93433
    ## Pos Pred Value        0.62969  0.41741 0.080156 0.129479  0.33657  0.25855
    ## Neg Pred Value        0.96150  0.98886 0.995756 0.994682  0.92785  0.94143
    ## Prevalence            0.05435  0.02082 0.005906 0.008580  0.09137  0.07496
    ## Detection Rate        0.01688  0.00995 0.001755 0.003402  0.02446  0.02118
    ## Detection Prevalence  0.02681  0.02384 0.021890 0.026276  0.07267  0.08193
    ## Balanced Accuracy     0.65007  0.73186 0.638423 0.686718  0.60731  0.60847
    ##                      Class: 7 Class: 8 Class: 9 Class: 10 Class: 11
    ## Sensitivity           0.24563 0.352657  0.27305  0.319927    0.5478
    ## Specificity           0.91951 0.926382  0.87344  0.889688    0.9226
    ## Pos Pred Value        0.16308 0.041057  0.27635  0.033207    0.8817
    ## Neg Pred Value        0.95022 0.993793  0.87160  0.991028    0.6594
    ## Prevalence            0.06002 0.008859  0.15038  0.011705    0.5131
    ## Detection Rate        0.01474 0.003124  0.04106  0.003745    0.2810
    ## Detection Prevalence  0.09041 0.076090  0.14859  0.112766    0.3187
    ## Balanced Accuracy     0.58257 0.639519  0.57325  0.604807    0.7352

Compare Accuracy of 2 Models:

23 variables model: 42.91%

5 variables model: 42.13%

The accuracy of model which contains all 23 variables and top 5 important variables almost the same!!!

It shows that small size model can replace with complex model.

Advantage (1) decreasing the training time (2) decreasing risk of overfitting

``` r
# end parallel processing ----------------------------------------------------------------------------------
stopCluster(cl)
registerDoSEQ()
```
