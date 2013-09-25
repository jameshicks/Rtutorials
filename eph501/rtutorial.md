R Tutorial for EPH501 Spring 2014
========================================================

What is R?
-----
#### R is a programming language
#### R is Open Source
#### R is free

What does R do?
-----

R Basics
-----
### Operations
R is best thought of as a fancy calculator. Anything you could do in a calculator, you can do in R. Starting with basic operations

```r
# Addition uses the '+' symbol
2 + 2
```

```
## [1] 4
```

```r
# Subtraction uses the '-' symbol
3 - 2
```

```
## [1] 1
```

```r
# Multiplication uses the '*' symbol
6 * 2
```

```
## [1] 12
```

```r
# Division uses the '/' symbol
24/4
```

```
## [1] 6
```

```r
# R obeys the order of operations. So this should equal 37
4^3 - 2 * 6 - 5 * 3
```

```
## [1] 37
```

### Comparisons

### Variables
#### Types
Each variable has a 'type' that tells R what you can and cant do with it. Here are some common types
* _Factor_: For nominal data: i.e. data where the label means something but you can't put it in order. Examples of nominal variables are $\{Cat, dog, bird\}$, or $\{Honda,Mazda,Ford,Chevy\}$.
* _Ordered_: Data where the ordering means something (ordinal), but you can't do much else with it. Scales are ordinal data. For example, the answers to the question, 'How do you feel today?' $\{Depressed,Sad,Find,Happy,Joyous\}$ are ordinal. A 'Happy' response feels better than 'sad', for sure, but you can't really say someone is '2 happier' than someone who resposnded 'sad', in the way that you can say someone is '2 inches taller' than someone else.
* _Numeric_: You guessed it! Numbers. $\{0,1,2.35,-1\}$ are all numeric type.
* _Characters_: Collections of letters and numbers. `'I am writing an R tutorial'` and `'This is a character type'` are both character typed. Character types have to be entered with quotes around them.
You can see what type a variable has with `typeof` or `str` (short for 'structure')

#### Scalars
#### Vectors
Vectors can be thought of as a collection of scalar variables, and can be created with the function `c`
### Functions
### Data frames
The function `str` is a useful way to get the structure of a dataframe: 

```r
data(iris)
str(iris)
```

```
## 'data.frame':	150 obs. of  5 variables:
##  $ Sepal.Length: num  5.1 4.9 4.7 4.6 5 5.4 4.6 5 4.4 4.9 ...
##  $ Sepal.Width : num  3.5 3 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 ...
##  $ Petal.Length: num  1.4 1.4 1.3 1.5 1.4 1.7 1.4 1.5 1.4 1.5 ...
##  $ Petal.Width : num  0.2 0.2 0.2 0.2 0.2 0.4 0.3 0.2 0.2 0.1 ...
##  $ Species     : Factor w/ 3 levels "setosa","versicolor",..: 1 1 1 1 1 1 1 1 1 1 ...
```

#### Subsetting Dataframes
### Reading data
### Saving data
### Installing new packages

Basic plotting 
-----

Statistics
-----

### t-tests
### Nonparametric tests
### Contingency Tables
### Analysis of Variance (ANOVA)
### Correlation
#### Pearson's r
#### Spearman's rho
### Regression 
