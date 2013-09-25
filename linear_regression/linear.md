Linear Models in R
========================================================

Linear models attempt to explain the relationship between two variables that are related.

Correlation
-----
Correlation measures the strength of the relationship between two variables. Correlations in R are available throught the `cor` function. The most commonly used measure of correlation is Pearson's product-moment correlation coefficient, abbreviated _r_. _r_ measures the straight-line relationship between two variables, with +1 indicating a perfect relationship between the increase in x and increase in y, -1 representing an increase in x related to a decrease in y. _r_=0 means no correlation. _r_ has the useful property where r^2 measures the amount of variance in y that can be explained by x. *r* can be calculated directly as $$r=\frac{1}{n-1} \sum ^n _{i=1} \left( \frac{X_i - \bar{X}}{s_X} \right) \left( \frac{Y_i - \bar{Y}}{s_Y} \right)$$

Significance testing of correlations are performed with `cor.test`.


```r
library(ggplot2)
data(iris)

qplot(iris$Petal.Length, iris$Petal.Width)
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-11.png) 

```r
cor(iris$Petal.Length, iris$Petal.Width)
```

```
## [1] 0.9629
```

```r
cor.test(iris$Petal.Length, iris$Petal.Width)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  iris$Petal.Length and iris$Petal.Width 
## t = 43.39, df = 148, p-value < 2.2e-16
## alternative hypothesis: true correlation is not equal to 0 
## 95 percent confidence interval:
##  0.9491 0.9730 
## sample estimates:
##    cor 
## 0.9629
```

```r

qplot(iris$Sepal.Length, iris$Sepal.Width)
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-12.png) 

```r
cor(iris$Sepal.Length, iris$Sepal.Width)
```

```
## [1] -0.1176
```

```r
cor.test(iris$Sepal.Length, iris$Sepal.Width)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  iris$Sepal.Length and iris$Sepal.Width 
## t = -1.44, df = 148, p-value = 0.1519
## alternative hypothesis: true correlation is not equal to 0 
## 95 percent confidence interval:
##  -0.27269  0.04351 
## sample estimates:
##     cor 
## -0.1176
```


In cases where _r_ is not appropriate, such as a non-linear relationship, _r_ will underestimate the relationship between the two variables. There are are alternatives: Spearman's rho measures the monotonicity of the relationship between two variables (e.g. if an increase in x is an increase in y).


```r
x <- 1:15
y <- x^6
qplot(x, y)
```

![plot of chunk nonlinearcorr](figure/nonlinearcorr.png) 

```r
cor(x, y, method = "pearson")
```

```
## [1] 0.7934
```

```r
cor(x, y, method = "spearman")
```

```
## [1] 1
```


You can look at the correlations of several variables at once by passing a dataframe to `cor`. This can be a little hard to look at, so you can visualize correlation with `symnum`. You can also make pairwise scatterplots with `pairs`.


```r

# Correlation is not defined for factors, So we'll have to skip the
# 'Species' column in iris
pairs(iris[1:4])
```

![plot of chunk cormatrix](figure/cormatrix.png) 

```r
cor(iris[1:4])
```

```
##              Sepal.Length Sepal.Width Petal.Length Petal.Width
## Sepal.Length       1.0000     -0.1176       0.8718      0.8179
## Sepal.Width       -0.1176      1.0000      -0.4284     -0.3661
## Petal.Length       0.8718     -0.4284       1.0000      0.9629
## Petal.Width        0.8179     -0.3661       0.9629      1.0000
```

```r
symnum(cor(iris[1:4]))
```

```
##              S.L S.W P.L P.W
## Sepal.Length 1              
## Sepal.Width      1          
## Petal.Length +   .   1      
## Petal.Width  +   .   B   1  
## attr(,"legend")
## [1] 0 ' ' 0.3 '.' 0.6 ',' 0.8 '+' 0.9 '*' 0.95 'B' 1
```


Simple regression
-----

Correlation is a useful, but blunt instrument for describing the relationship between two variables. _r_ tells you about how well the data fits a straight line, but doesn't really tell you much about the line. You're not given the intercept or the slope. You're just told that there's a line. To understand the relationship of two variables better you need to fit a regression model. 

A linear regression tries to find the line that minimizes the sum of the squared vertical distance from the line to the datapoint (hence: least squares, sums of squares, etc). The ratio of the sum of squares explained by the model to the sum of squares total is known as _R^2_ and is used as a goodness of fit measure for the model. In the case of simple linear regression this is equal to _r^2_. 

A regression line takes the scalar form: $$Y=\beta _{0} + \beta _{1} X$$.  

In the case of linear regression, the coefficient estimates can be calculated directly, but we'll let R handle that for us. 

### Assumptions of linear regression
* __Normality__: The repsonse variable Y is normally distributed
  * Needed for the significance tests to be valid.
  * This is why things like logistic and poisson regression require a special model. A linear model will start predicting Y values outside of the domain of the Y variable youre looking at ({0,1} for logistic,>=0 for poisson)
* __Independence__: The Y variable observations should not be dependent on each other
  * Can by violated by relatedness of individuals (observations are dependent based on degree of relationship) or by time-series data (every observation is correlated to the one before it (autocorrelation))
* __Homoscedasticity__ (homogeneity of variance): The variance of Y at any value of X is uncorrelated with the value of X.

Linear regression models are fit in R with `lm`. The basic format of this is `lm(y~x,data=dataframe)`, where y is the outcome variable and x is the predictor. Running `lm` by itself doesnt really tell you much about the line besides the slope and intercept. But `summary` is useful.


```r
model <- lm(Petal.Length ~ Petal.Width, data = iris)
plot(Petal.Length ~ Petal.Width, data = iris)
abline(model, col = "red")
```

![plot of chunk simpleregression](figure/simpleregression.png) 

```r
summary(model)
```

```
## 
## Call:
## lm(formula = Petal.Length ~ Petal.Width, data = iris)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -1.3354 -0.3035 -0.0295  0.2578  1.3945 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   1.0836     0.0730    14.8   <2e-16 ***
## Petal.Width   2.2299     0.0514    43.4   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.478 on 148 degrees of freedom
## Multiple R-squared: 0.927,	Adjusted R-squared: 0.927 
## F-statistic: 1.88e+03 on 1 and 148 DF,  p-value: <2e-16
```


The summary here gives useful information. First is a summary of the residuals of the model. In a linear regression these should be normally distributed with mean=0. The line intersects the y axis at around 1. For every one unit increase in Petal.Width, you get a 2.2 increase in Petal.Length. The R^2, 0.9271, is pretty high. You also get an F statistic and p-value for the whole model.

You should always look at your regression lines. For example: the following four datasets (called "Anscombe's Quartet") all have the same parameter estimates, while being obviously different datasets.

```r
data(anscombe)
ggplot(anscombe, aes(x = x1, y = y1)) + geom_point(size = rel(5)) + geom_smooth(method = "lm") + 
    xlim(3, 20) + ylim(0, 20)
```

![plot of chunk anscombe](figure/anscombe1.png) 

```r
lm(y1 ~ x1, data = anscombe)
```

```
## 
## Call:
## lm(formula = y1 ~ x1, data = anscombe)
## 
## Coefficients:
## (Intercept)           x1  
##         3.0          0.5
```

```r
ggplot(anscombe, aes(x = x2, y = y2)) + geom_point(size = rel(5)) + geom_smooth(method = "lm") + 
    xlim(3, 20) + ylim(0, 20)
```

![plot of chunk anscombe](figure/anscombe2.png) 

```r
lm(y2 ~ x2, data = anscombe)
```

```
## 
## Call:
## lm(formula = y2 ~ x2, data = anscombe)
## 
## Coefficients:
## (Intercept)           x2  
##         3.0          0.5
```

```r
ggplot(anscombe, aes(x = x3, y = y3)) + geom_point(size = rel(5)) + geom_smooth(method = "lm") + 
    xlim(3, 20) + ylim(0, 20)
```

![plot of chunk anscombe](figure/anscombe3.png) 

```r
lm(y3 ~ x3, data = anscombe)
```

```
## 
## Call:
## lm(formula = y3 ~ x3, data = anscombe)
## 
## Coefficients:
## (Intercept)           x3  
##         3.0          0.5
```

```r
ggplot(anscombe, aes(x = x4, y = y4)) + geom_point(size = rel(5)) + geom_smooth(method = "lm") + 
    xlim(3, 20) + ylim(0, 20)
```

![plot of chunk anscombe](figure/anscombe4.png) 

```r
lm(y4 ~ x4, data = anscombe)
```

```
## 
## Call:
## lm(formula = y4 ~ x4, data = anscombe)
## 
## Coefficients:
## (Intercept)           x4  
##         3.0          0.5
```


Multiple Regression
-----
Multiple regression is an extension of simple regression to multiple predictor variables. In the case of one predictor, you're finding a best-fit line through 2 dimensions (y and x). In a two predictor model you're fitting a best-fit **plane** through a 3 dimensional space (x1,x2, and y). At 4 predictors you're fitting a hyperplane through a four dimensional space and visualization sort-of breaks down. 

Once you can handle that, multiple regression conceptually follows pretty easily from simple regression. Basically, in multiple regression you fit a simple regression to the residuals of the regression of the previous predictor in the model. So you fit the intercept, then what variance is left over is used to for the next predictor, and so forth. Ideally, these predictors are uncorrelated with each other, outside of being correlated with the y variable, making the effects additive. Adding *any* predictor to a regression will increase its R^2 (even if it doesn't predict Y!), because the predictor will fit a certain amount of noise. In multiple regression, you should be considering **Adjusted R^2**, which is an R^2 with a correction for extra terms in the model. For *n* observations of *p* predictors, adjusted R^2 can be calculated as: $$\overline{R}^{2} = R^{2} - (1-R^2)\frac{p}{n-p-1}$$ 


```r
multiple <- lm(Sepal.Length ~ Petal.Length + Sepal.Width, data = iris)
summary(multiple)
```

```
## 
## Call:
## lm(formula = Sepal.Length ~ Petal.Length + Sepal.Width, data = iris)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -0.9616 -0.2349  0.0008  0.2145  0.7856 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    2.2491     0.2480    9.07  7.0e-16 ***
## Petal.Length   0.4719     0.0171   27.57  < 2e-16 ***
## Sepal.Width    0.5955     0.0693    8.59  1.2e-14 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 0.333 on 147 degrees of freedom
## Multiple R-squared: 0.84,	Adjusted R-squared: 0.838 
## F-statistic:  386 on 2 and 147 DF,  p-value: <2e-16
```


In this model, after considering the intercept, for a 1-unit increas in Petal.Length you get a .47-unit increase in Sepal.Length. After considering those two variables, each unit increase in Sepal.Width results in a 0.59 increase in Sepal.Length. 

Formula Language
-----
The symbols you see in R formulas are not exactly what they look like. The `~` separates the response (to the left) from the predictors (to the right). The `+` symbol adds terms to a model, `-` explicity removes them. The symbols `*`,`/`, and `^` don't do what you might think they do either. To do arithmetic within the model incase the terms in the `I` function. For example, adding the term `I(x+y)` would add the value of x and y, while adding `x+y` to your model would add x and y into your model as separate terms. Another useful formula operator is `.` which means 'all terms not mentioned yet'. So `lm(y~.,data=df)` would regress y against all other variables in the data frame df.

Interaction
-----
You can say two variables interact when their relationship to Y are not completely independant of one another. This can be most easily seen with categorical variables:

```r
library(MASS)
data(birthwt)  # Risk factors for low birth-weight.
qplot(lwt, bwt, data = birthwt, color = as.factor(ui)) + geom_smooth(method = lm)
```

![plot of chunk ixnplot](figure/ixnplot.png) 

You can see here that mothers with a history of uterine instability (ui=1) have a different relationship with mothers weight than those without. Those with a history have a slightly decreassing line, but those without have an increasing one. We can say that ui *modifies* lwt's prediction of age, or that these variables *interact*. 

Interaction terms can be specified in R with **:**. ui:lwt specifies a ui-lwt interaction in the model:


```r
summary(lm(bwt ~ lwt + as.factor(ui) + as.factor(ui):lwt, data = birthwt))
```

```
## 
## Call:
## lm(formula = bwt ~ lwt + as.factor(ui) + as.factor(ui):lwt, data = birthwt)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -2119.8  -472.1   -23.3   530.1  1994.8 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)         2496.10     242.37   10.30   <2e-16 ***
## lwt                    4.06       1.79    2.26    0.025 *  
## as.factor(ui)1        31.74     630.52    0.05    0.960    
## lwt:as.factor(ui)1    -4.72       5.10   -0.92    0.356    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 695 on 185 degrees of freedom
## Multiple R-squared: 0.105,	Adjusted R-squared: 0.091 
## F-statistic: 7.27 on 3 and 185 DF,  p-value: 0.000123
```

We can see that there's a different line there but its not strong enough that we can say the difference isn't due to chance. There are shortcuts for creating interactions. Using the __*__ operator adds variables and their two-way interactions to the model. For `a*b`, this expands to `a+b+a:b`. The **^** includes intertactions up to a point. For example `(a+b+c+d)^3` will include all pairs of variables as well as `a:b:c`, `a:b:d`, `a:c:d`, and `b:c:d`. Since we've limited it to 3rd-order interactions `a:b:c:d` will not be considered. 

Dummy variables
-----
Dummy variables are how you incorporate nominal variables into a regression model. When you do the math on it, you're actually changing the intercept of the model (Dummy variables are usually coded 0/1, and like intercepts have no variable terms). For factor terms, R picks one level of the variable as the default and then adds in 0/1 variables for the remaining levels of the factor.


```r
# Predict birthweight(bwt) from mothers weight (lwt), age, race, and
# whether the mother smoked in pregnancy (smoke) Be careful with
# number-coded nominal variables as R likes to try to fit them as
# continuous numerics. In this case, race is coded
# (1=white,2=black,3=other) but theres no real order relationship between
# the levels on that.
birth <- lm(bwt ~ lwt + age + as.factor(race) + smoke, data = birthwt)
summary(birth)
```

```
## 
## Call:
## lm(formula = bwt ~ lwt + age + as.factor(race) + smoke, data = birthwt)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -2281.9  -449.1    24.3   474.1  1746.2 
## 
## Coefficients:
##                  Estimate Std. Error t value Pr(>|t|)    
## (Intercept)       2839.43     321.43    8.83  8.2e-16 ***
## lwt                  4.00       1.74    2.30  0.02249 *  
## age                 -1.95       9.82   -0.20  0.84299    
## as.factor(race)2  -510.50     157.08   -3.25  0.00137 ** 
## as.factor(race)3  -398.64     119.58   -3.33  0.00104 ** 
## smoke             -401.72     109.24   -3.68  0.00031 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## Residual standard error: 682 on 183 degrees of freedom
## Multiple R-squared: 0.148,	Adjusted R-squared: 0.125 
## F-statistic: 6.37 on 5 and 183 DF,  p-value: 1.76e-05
```

From the summary, you can see that smoking reduces the average birthweight by 269 grams,even when accounting for mothers age, race, and weight.

Hypothesis testing in regression
-----
The first major hypothesis test you should consider is whether the model actually predicts the outcome variable at all. This is the *F* statistic at the bottom of `summary.lm`. (The test statistic for overall regression is *F* distributed because it is the ratio of the variance explained by the model to the overall variance *not* explained) You can see in the above example, the combination of predictors used significantly predicts birthweight. 

Aside from testing the overall fit of the model, you can get p-values for each term in the model. This is useful for investigating the relationship between two variables while accounting for other possibly correlated variables first. These are called **partial *F* tests**. These test models nested within each other (e.g. models in which one model has a reduced set of predictors from a larger model): specifically the ratio of variance explained by the larger model to the reduced model. (These are *F* distributed because they are the ratio of chi-square distibuted error measures). If the full model predicts Y much better than the reduced one, you can say that the predictors you're adding in significantly predict Y. When doing single-predictor tests, R reports a *t* statistic instead of an F. The t statistic here is a special case of F when only considering one variable. It is mathematically equivalent. In the above example we see that accounting for the mother's age and the intercept, age is not a significant predictor of a baby's birth weight, and we could drop it from the model without losing any accuracy. 


Model Selection
-----
Often, then, you want to make a model containing only significant predictors. This can be more tricky than it appears. There are three commonly used methods. Forward selection _adds_ variables to a model until they stop adding information to the model. Backward selection starts with the full model and _removes_ variables until it starts removing informative predictors. Stepwise regression _adds_ variables, and at each step, doubles back and removes predictors that are no longer needed. 

### Information criteria
While commonly taught in textbooks, p-value based model selection schemes are not recommended by the majority of statisticians, because it produces biased parameter estimates an, in the case of stepwise regression, uninterpretable p-values. 

The alternative to using p-values is using an information criterion. These prioritize models that are more likely given the data you have and add a penalty for adding an additional term into the model. This has the effect of requiring an additional term to have substantial prediction without assigning it a false-positive probability. The most common of these is Akaike's formation criterion: $AIC=2k-2\ln L$ where L is the likelihood of the model and k is the number of parameters estimated. Another common variant of AIC is the Bayesian Information Criteria, $BIC=-2\ln L+k\ln n$ where n is the number of data points. BIC penalizes more heavily for adding terms to a model and is thus more appropriate in situations where a false positive is worse than a false negative. For these both, smaller is better. BIC is implemented in R as a modified version of AIC by adjusting the penalty term from 2 to $\ln n$ where *n* is the number of observations used to fit the model


```r
# Create a baseline model
birth <- lm(bwt ~ lwt + smoke, data = birthwt)
# Get the AIC
extractAIC(birth)
```

```
## [1]    3 2483
```

```r
# Get the BIC
extractAIC(birth, k = log(nrow(birthwt)))
```

```
## [1]    3 2493
```

```r
# Add the useful predictor ui (presence of uterine irritability)
birth_with_ui <- update(birth, . ~ . + ui)
# AIC goes down, because ui helps the model.
extractAIC(birth_with_ui)
```

```
## [1]    4 2473
```

```r
# Add in the useless predictor, age.
birth_with_age <- update(birth, . ~ . + age)
extractAIC(birth)
```

```
## [1]    3 2483
```

```r
# AIC goes up, because age does not increase the likelihood of the model.
extractAIC(birth_with_age)
```

```
## [1]    4 2485
```


### Model Selection

The best way to do model selection is with the `stepAIC` function implement in the `MASS` package, which should be included in the default installation. `stepAIC` takes an option `direction`, indicating the type of selection  which can be 'forward', 'backward' or 'both'. Choosing 'both' indicates stepwise selection. `stepAIC` defaults to backward selection. 


```r
fullmodel <- lm(bwt ~ . - low, data = birthwt)
reducedmodel <- stepAIC(fullmodel, direction = "both")
```

```
## Start:  AIC=2461
## bwt ~ (low + age + lwt + race + smoke + ptl + ht + ui + ftv) - 
##     low
## 
##         Df Sum of Sq      RSS  AIC
## - age    1       331 77681278 2459
## - ftv    1     47283 77728230 2459
## - ptl    1    106439 77787385 2459
## <none>               77680946 2461
## - lwt    1   1762311 79443257 2463
## - ht     1   3728637 81409584 2468
## - race   1   4599967 82280914 2470
## - smoke  1   4796844 82477791 2470
## - ui     1   5732239 83413186 2473
## 
## Step:  AIC=2459
## bwt ~ lwt + race + smoke + ptl + ht + ui + ftv
## 
##         Df Sum of Sq      RSS  AIC
## - ftv    1     50343 77731620 2457
## - ptl    1    110047 77791325 2457
## <none>               77681278 2459
## + age    1       331 77680946 2461
## - lwt    1   1789656 79470934 2461
## - ht     1   3731126 81412404 2466
## - race   1   4707970 82389248 2468
## - smoke  1   4843734 82525012 2469
## - ui     1   5749594 83430871 2471
## 
## Step:  AIC=2457
## bwt ~ lwt + race + smoke + ptl + ht + ui
## 
##         Df Sum of Sq      RSS  AIC
## - ptl    1    108936 77840556 2455
## <none>               77731620 2457
## + ftv    1     50343 77681278 2459
## + age    1      3390 77728230 2459
## - lwt    1   1741198 79472818 2459
## - ht     1   3681167 81412788 2464
## - race   1   4660187 82391807 2466
## - smoke  1   4810582 82542203 2467
## - ui     1   5716074 83447695 2469
## 
## Step:  AIC=2455
## bwt ~ lwt + race + smoke + ht + ui
## 
##         Df Sum of Sq      RSS  AIC
## <none>               77840556 2455
## + ptl    1    108936 77731620 2457
## + ftv    1     49231 77791325 2457
## + age    1     10218 77830338 2457
## - lwt    1   1846738 79687294 2458
## - ht     1   3718531 81559088 2462
## - race   1   4727071 82567628 2465
## - smoke  1   5237430 83077987 2466
## - ui     1   6302771 84143327 2468
```

```r
aics <- c(extractAIC(fullmodel)[2], extractAIC(reducedmodel)[2])
aics
```

```
## [1] 2461 2455
```


With the known relationship between AIC and relative likelihood, we can see that the reduced model is 16.5441 times more likely to be the true model.

The Generalized Linear Model
-----
GLMs are an extension of the linear model idea to non-normal outcomes. The idea behind GLMs is to find a linear combination of predictors that can be tranformed to an outcome variable through a **link function**. Each distribution has its own link function. In R, GLMs are implemented in `glm`. These have most of the familiar properties of `lm`, with the added options for link function or distribution of outcome variable. GLMs really deserve their own class because they're beyond the scope of this one. But the most common one you might have to use is logistic regression. GLMs have to find their correct coefficients by iteration (instead of an exact calculation like in linear regression), so this may take longer than you might think. 

### Logistic Regression
Logistic regression models binomial output (eg yes/no, case/control, etc.) by transforming the linear predictors through the logit function. Coefficients in logistic regression can be interpreted as **odds ratios** (eg a 1 unit increase in predictor X corresponds to a 1 unit increase in the odds of class 1 membership). Logistic regression can be used by specifying `family='binomial'` in the glm. This will default to the logistic link (instead of the closely related probistic regression.)


```r
data(Pima.te)  # Diabetes in Pima Indian Women
# Perform logistic regression on diabetes status from age, bmi, and number
# of pregnancies.
summary(glm(type ~ age + bmi + npreg, data = Pima.te, family = "binomial"))
```

```
## 
## Call:
## glm(formula = type ~ age + bmi + npreg, family = "binomial", 
##     data = Pima.te)
## 
## Deviance Residuals: 
##    Min      1Q  Median      3Q     Max  
## -1.899  -0.816  -0.544   1.002   2.004  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept)  -5.9733     0.8275   -7.22  5.2e-13 ***
## age           0.0413     0.0156    2.64   0.0082 ** 
## bmi           0.1060     0.0195    5.45  5.1e-08 ***
## npreg         0.0879     0.0509    1.73   0.0841 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1 
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 420.30  on 331  degrees of freedom
## Residual deviance: 357.69  on 328  degrees of freedom
## AIC: 365.7
## 
## Number of Fisher Scoring iterations: 4
```


Other Packages and Functions
-----
* Running `confint(model)` will provide confidence intervals for model parameters
* A variety of hypothesis tests for linear models are found in the __lmtest__ package, most notably the likelihood ratio test, which is sometimes more powerful than the typical tests.
* Multinomial logistic regression can be found in the `multinom` function in __nnet__. 
* For dealing with correlated data, mixed models are avaiable in __nmle__ and __lme4__. Generalized Estimating Equations are avaibale in the packages __gee__ and __geepack__.
* Quantile regression is available in __quantreg__
* Bayesian regression can be found in __MCMCpack__, __bayesm__, and several other packages.
