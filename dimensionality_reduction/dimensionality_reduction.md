Dimensionality Reduction
========================================================


```r
library(ggplot2)
```


Principal Component Analysis
-----

### What is PCA?
Preparations:
* Data must be all be continuous. (The variance of the vector {A,B,C} can't be calculated.)
** Note that in PCA of GWAS data, the genotypes are coded as number of minor alleles (i.e. 0,1,2)
* Data must be centered (i.e. subtract the mean from each variable so that they have mean 0)
* Data must be scaled so that they have the same variance. Because PCA works by finding directions of maximum variance, allowing variables to have different variances can produce biased results. __Note__: The built-in R functions for PCA, `prcomp` and `princomp` __do not__ scale by default. `prcomp` will scale with the option `scale=TRUE`. You shouldn't be using `princomp` at all. 

### Performing PCA
R has many options for performing PCA, including the builtin functions `prcomp` and `princomp`. However, these are old and crufty, and can require you to know what you're doing to get correct results. Infact, `princomp` calculates PCA by performing eigenvalue decomposition on a correlation matrix and can introduce rounding errors into the results. The R help page for `princomp` suggests not using it at all. (`prcomp` uses the more-modern approach of performing singular-value decomposition on the data matrix itself, which avoids that issue entirely).
I prefer an easy to use package, `FactoMineR` for PCA, and will be using it here. The package also includes factor analysis and other related techniques not discussed here (like correspondence analysis).


```r
library(FactoMineR)
```

```
## Loading required package: car
## Loading required package: ellipse
## 
## Attaching package: 'ellipse'
## 
## The following object is masked from 'package:car':
## 
##     ellipse
## 
## Loading required package: lattice
## Loading required package: cluster
## Loading required package: scatterplot3d
## Loading required package: leaps
```

```r
data(swiss)
```


```r
# Perform PCA
sw.pca <- PCA(swiss)
```

![plot of chunk swissPCA](figure/swissPCA1.png) ![plot of chunk swissPCA](figure/swissPCA2.png) 

```r
print(sw.pca)
```

```
## **Results for the Principal Component Analysis (PCA)**
## The analysis was performed on 47 individuals, described by 6 variables
## *The results are available in the following objects:
## 
##    name               description                          
## 1  "$eig"             "eigenvalues"                        
## 2  "$var"             "results for the variables"          
## 3  "$var$coord"       "coord. for the variables"           
## 4  "$var$cor"         "correlations variables - dimensions"
## 5  "$var$cos2"        "cos2 for the variables"             
## 6  "$var$contrib"     "contributions of the variables"     
## 7  "$ind"             "results for the individuals"        
## 8  "$ind$coord"       "coord. for the individuals"         
## 9  "$ind$cos2"        "cos2 for the individuals"           
## 10 "$ind$contrib"     "contributions of the individuals"   
## 11 "$call"            "summary statistics"                 
## 12 "$call$centre"     "mean of the variables"              
## 13 "$call$ecart.type" "standard error of the variables"    
## 14 "$call$row.w"      "weights for the individuals"        
## 15 "$call$col.w"      "weights for the variables"
```

### Interpreting PCA
#### Eigenvalues
#### PC Scores
#### Loadings
#### How many components should I use?
No rigorous way to choose:
* __Kaiser rule__: Only keep PCs with eigenvalues over 1.
* __Screeplot__: Plot the eigenvalues of PCs in order and look for an elbow (where the line bends into a flat plane, indicating random noise)

```r
# A function to make a prettier screeplot than you'd get with default
# graphics
fmscree <- function(PCAobj) {
    df <- as.data.frame(PCAobj$eig)
    df$component <- seq(1, nrow(df))
    plot <- ggplot(df, aes(x = component, y = eigenvalue)) + geom_point(size = 3) + 
        geom_line()
    plot <- plot + geom_hline(yintercept = 1, color = "red", lty = "dotted")
    print(plot)
}
```

### Examples
#### Comparing cars by their specs.
The `mtcars` dataset has specs and performance for 32 cars from an issue of Motor Trend from 1972. We're going to use 9 of the predictors (skipping two categorical ones). When we look at the correlations, we see theres a good amount of 

```r
data(mtcars)
symnum(cor(mtcars[, -c(8, 9)]))
```

```
##      m cy ds h dr w q g cr
## mpg  1                    
## cyl  + 1                  
## disp + *  1               
## hp   , +  ,  1            
## drat , ,  ,  . 1          
## wt   + ,  +  , ,  1       
## qsec . .  .  ,      1     
## gear . .  .    ,  .   1   
## carb . .  .  ,    . ,   1 
## attr(,"legend")
## [1] 0 ' ' 0.3 '.' 0.6 ',' 0.8 '+' 0.9 '*' 0.95 'B' 1
```

Let's see what comes up in the PCA

```r
mtcars.pca <- PCA(mtcars[, -c(8, 9)], graph = F)
summary(mtcars.pca)
```

```
## 
## Call:
## knit("dimensionality_reduction.Rmd", encoding = "UTF-8") 
## 
## 
## Eigenvalues
##                        Dim.1   Dim.2   Dim.3   Dim.4   Dim.5   Dim.6
## Variance               5.656   2.082   0.504   0.265   0.183   0.124
## % of var.             62.844  23.134   5.602   2.945   2.035   1.375
## Cumulative % of var.  62.844  85.978  91.581  94.525  96.560  97.936
##                        Dim.7   Dim.8   Dim.9
## Variance               0.105   0.059   0.022
## % of var.              1.167   0.650   0.247
## Cumulative % of var.  99.103  99.753 100.000
## 
## Individuals (the 10 first)
##                       Dist    Dim.1    ctr   cos2    Dim.2    ctr   cos2  
## Mazda RX4         |  1.658 | -0.675  0.252  0.166 |  1.192  2.133  0.517 |
## Mazda RX4 Wag     |  1.446 | -0.647  0.232  0.200 |  0.993  1.479  0.471 |
## Datsun 710        |  2.485 | -2.337  3.016  0.884 | -0.332  0.165  0.018 |
## Hornet 4 Drive    |  2.094 | -0.219  0.026  0.011 | -2.008  6.054  0.920 |
## Hornet Sportabout |  2.138 |  1.612  1.436  0.569 | -0.842  1.064  0.155 |
## Valiant           |  2.667 |  0.050  0.001  0.000 | -2.486  9.275  0.869 |
## Duster 360        |  2.949 |  2.758  4.202  0.875 |  0.367  0.202  0.015 |
## Merc 240D         |  2.470 | -2.076  2.382  0.707 | -0.813  0.993  0.108 |
## Merc 230          |  3.459 | -2.332  3.004  0.454 | -1.326  2.641  0.147 |
## Merc 280          |  1.290 | -0.389  0.083  0.091 |  0.590  0.523  0.209 |
##                    Dim.3    ctr   cos2  
## Mazda RX4         -0.208  0.267  0.016 |
## Mazda RX4 Wag      0.113  0.079  0.006 |
## Datsun 710        -0.214  0.283  0.007 |
## Hornet 4 Drive    -0.335  0.694  0.026 |
## Hornet Sportabout -1.050  6.827  0.241 |
## Valiant            0.114  0.080  0.002 |
## Duster 360        -0.662  2.720  0.050 |
## Merc 240D          0.863  4.611  0.122 |
## Merc 230           2.000 24.791  0.334 |
## Merc 280           0.901  5.026  0.487 |
## 
## Variables
##                      Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3
## mpg               | -0.935 15.457  0.874 |  0.040  0.076  0.002 | -0.157
## cyl               |  0.957 16.205  0.917 |  0.023  0.025  0.001 | -0.179
## disp              |  0.945 15.789  0.893 | -0.128  0.790  0.016 | -0.056
## hp                |  0.873 13.475  0.762 |  0.389  7.258  0.151 | -0.012
## drat              | -0.742  9.723  0.550 |  0.493 11.673  0.243 |  0.106
## wt                |  0.888 13.949  0.789 | -0.248  2.956  0.062 |  0.322
## qsec              | -0.534  5.033  0.285 | -0.698 23.430  0.488 |  0.446
## gear              | -0.498  4.388  0.248 |  0.795 30.336  0.632 |  0.147
## carb              |  0.582  5.982  0.338 |  0.699 23.456  0.488 |  0.330
##                      ctr   cos2  
## mpg                4.893  0.025 |
## cyl                6.366  0.032 |
## disp               0.612  0.003 |
## hp                 0.030  0.000 |
## drat               2.249  0.011 |
## wt                20.587  0.104 |
## qsec              39.454  0.199 |
## gear               4.268  0.022 |
## carb              21.541  0.109 |
```

```r
# Make a screeplot to visualize the eigenvalues
fmscree(mtcars.pca)
```

![plot of chunk mtcarspca](figure/mtcarspca.png) 

Looking at the Scree plot, its clear that there are only two major principal components, accounting for 85% of the total variance in the dataset. What do they represent?

```r
# choix tells whether to plot individual PC scores or PC loadings for
# variables
plot(mtcars.pca, choix = "ind")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

Just by eyeballing it, you can see that PC1 (accounting for ~63% of the variance in the dataset) measures the 'fanciness' of the car, as well as it can by specs and performance. In the negative direction you have economy cars, like Hondas and Fiats, as well as a few higher end models which have lower specification. On the other end you have higher end luxury cars: Maseratis, Chryslers, Cadillacs, etc. 


Principal component 2 seems to separate cars based on performance. The Ferrari, Maserati, and the Pantera (a sports car) all have high scores on this PC. On the other hand, you have cars like the Toyota Corona, the Hornet, and especially the Valiant are very poorly performing cars. 


```r
plot(mtcars.pca, choix = "var")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

This plot shows the relationship of the *variables* to the two principal components. Cars that get better mileage are usually economy cars, and mpg shows that. A larger value for mpg results in a lower score on PC1 (the fancy car PC). On the other hand, fancier cars usually have bigger engines, and larger numbers of cylinders, increased displacement, and weight all results in a higher score on PC1. 

Cars that perform poorly on the quarter mile get lower scores on PC2. These are also typically the lower spec'ed cars, so increased values of qsec result in lower scores also get you a lower score on PC1. The opposite is true for number of carbs and horsepower. 
#### Outlier detection: The Florida 2000 election.
It can be hard to identify outliers in high dimensional spaces, where there are so many things going on. A great place to look for weird things happening is the 2000  Florida election, where weird things did happen. The dataset `Florida` in package `car` has vote counts for each florida county, for 10 presidential candidates. What happens when you look at the vote counts with PCA.  

```r
data(Florida, package = "car")
fl.pca <- PCA(Florida[1:10], graph = F)
plot(fl.pca, choix = "ind")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

Lets look at the first two PCs. So theres an expected group around 0,0. And you can see the large population counties (Dade, Hillsborough, etc) off a little farther. They're going to separate out in any analysis not normalized by population size because their populations are so much larger than the other counties. But the real star of this plot is Volusia county, which is far, far away on PC2 from the rest of the counties. 

```r
plot(fl.pca, choix = "var")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 

When we look at the variable plot, we see that high scores on PC2 are driven by voting for some people I've never heard of (Harris, Phillips, Browne). So that's a big hint that somethings up. Lets compare Volusia to two (geographically) nearby counties, Flagler and Seminole. 


```r
Florida[c("SEMINOLE", "FLAGLER", "VOLUSIA"), ]
```

```
##           GORE  BUSH BUCHANAN NADER BROWNE HAGELIN HARRIS MCREYNOLDS
## SEMINOLE 58888 75293      194  1940    551      38     38          5
## FLAGLER  13891 12608       83   435     60       4      1          3
## VOLUSIA  97063 82214      396  2436   3211      33   9888          3
##          MOOREHEAD PHILLIPS  Total
## SEMINOLE        70       27 137044
## FLAGLER         12        3  27100
## VOLUSIA         59     2927 198230
```

So it looks like the variable causing the issue is too many people voting for Harris than you might expect. For comparison, the median number of votes for Harris per county was 4.

In 2000, Volusia county's computers had an error that (among other things) gave 9888 erroneous votes to James Harris, the Socialist Workers Party candidate for president. [Here's a New York Times article on it](http://www.nytimes.com/2000/11/10/us/2000-campaign-florida-vote-democrats-tell-problems-polls-across-florida.html?pagewanted=all&src=pm)
#### Crabs: reducing many size measurements into a one dimensional feature space
The `crabs` dataset is a catalog of measurements of 8 variables on 200 crabs. We're concerned right now with continuous variables, frontal lobe size (FL), rear width (RW), carapace length (CL), carapace width (CW), and body depth (BD). But species (O and M) and sex are available.


```r
data(crabs, package = "MASS")
head(crabs)
```

```
##   sp sex index   FL  RW   CL   CW  BD
## 1  B   M     1  8.1 6.7 16.1 19.0 7.0
## 2  B   M     2  8.8 7.7 18.1 20.8 7.4
## 3  B   M     3  9.2 7.8 19.0 22.4 7.7
## 4  B   M     4  9.6 7.9 20.1 23.1 8.2
## 5  B   M     5  9.8 8.0 20.3 23.0 8.2
## 6  B   M     6 10.8 9.0 23.0 26.5 9.8
```

```r
# These are our variables of interest
sizevars <- 4:8
```


Just by looking at the correlations between these variables it's pretty clear what's going on: all the measurements are strongly correlated with each other. Looking at the screeplot confirms it: There is only one principal component for all the size data. 

```r
round(cor(crabs[sizevars]), 2)
```

```
##      FL   RW   CL   CW   BD
## FL 1.00 0.91 0.98 0.96 0.99
## RW 0.91 1.00 0.89 0.90 0.89
## CL 0.98 0.89 1.00 1.00 0.98
## CW 0.96 0.90 1.00 1.00 0.97
## BD 0.99 0.89 0.98 0.97 1.00
```

```r
plot(crabs[sizevars])
```

![plot of chunk corcrabs](figure/corcrabs.png) 

Lets do the PCA anyway and take a look around:

```r
c.pca <- PCA(crabs[sizevars], graph = F)
summary(c.pca)
```

```
## 
## Call:
## knit("dimensionality_reduction.Rmd", encoding = "UTF-8") 
## 
## 
## Eigenvalues
##                        Dim.1   Dim.2   Dim.3   Dim.4   Dim.5
## Variance               4.789   0.152   0.047   0.011   0.002
## % of var.             95.777   3.034   0.933   0.223   0.034
## Cumulative % of var.  95.777  98.810  99.743  99.966 100.000
## 
## Individuals (the 10 first)
##        Dist    Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3    ctr
## 1  |  4.937 | -4.928  2.535  0.996 | -0.268  0.238  0.003 | -0.122  0.160
## 2  |  4.387 | -4.386  2.009  0.999 | -0.094  0.029  0.000 | -0.039  0.017
## 3  |  4.133 | -4.129  1.780  0.998 | -0.169  0.094  0.002 |  0.034  0.012
## 4  |  3.892 | -3.884  1.575  0.996 | -0.246  0.199  0.004 |  0.015  0.002
## 5  |  3.841 | -3.834  1.535  0.996 | -0.224  0.166  0.003 | -0.015  0.002
## 6  |  2.962 | -2.953  0.910  0.994 | -0.220  0.160  0.006 |  0.038  0.016
## 7  |  2.680 | -2.678  0.749  0.999 |  0.039  0.005  0.000 |  0.082  0.072
## 8  |  2.575 | -2.548  0.678  0.979 | -0.363  0.435  0.020 |  0.063  0.042
## 9  |  2.593 | -2.585  0.698  0.994 | -0.117  0.045  0.002 |  0.062  0.042
## 10 |  2.213 | -2.206  0.508  0.994 |  0.079  0.021  0.001 |  0.157  0.264
##      cos2  
## 1   0.001 |
## 2   0.000 |
## 3   0.000 |
## 4   0.000 |
## 5   0.000 |
## 6   0.000 |
## 7   0.001 |
## 8   0.001 |
## 9   0.001 |
## 10  0.005 |
## 
## Variables
##       Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3    ctr   cos2  
## FL |  0.989 20.434  0.979 | -0.054  1.893  0.003 | -0.115 28.172  0.013 |
## RW |  0.937 18.325  0.878 |  0.350 80.664  0.122 |  0.003  0.014  0.000 |
## CL |  0.992 20.538  0.984 | -0.104  7.195  0.011 |  0.067  9.590  0.004 |
## CW |  0.987 20.350  0.975 | -0.070  3.261  0.005 |  0.141 42.585  0.020 |
## BD |  0.987 20.352  0.975 | -0.103  6.987  0.011 | -0.096 19.639  0.009 |
```


As we expected, one principal component accounts for 95% of the variance in the data. We've effectively reduced the 5 size dimensions down to one, while losing very little of the information in the original dataset. By bringing sex and species back into the picture, we can see that O species crabs are significantly 'bigger': that is, they have a higher score on PC1 than species B.  

```r
c <- crabs
c$PC1 <- c.pca$ind$coord[, 1]
ggplot(c, aes(x = PC1, fill = sp)) + geom_density() + facet_grid(sp ~ sex)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

#### You can get prinicpal components out of nothing.
Principal components can always be found. That doesn't necessarily mean they mean something. Here's some randomly generated data: 200 'observations' of 5 variables, all random normal variates. 

```r
summary(PCA(matrix(rnorm(1000), ncol = 5)))
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-101.png) ![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-102.png) 

```
## 
## Call:
## knit("dimensionality_reduction.Rmd", encoding = "UTF-8") 
## 
## 
## Eigenvalues
##                        Dim.1   Dim.2   Dim.3   Dim.4   Dim.5
## Variance               1.250   1.099   1.030   0.891   0.731
## % of var.             25.003  21.979  20.591  17.812  14.616
## Cumulative % of var.  25.003  46.981  67.572  85.384 100.000
## 
## Individuals (the 10 first)
##        Dist    Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3    ctr
## 1  |  2.050 | -0.220  0.019  0.011 |  1.555  1.100  0.575 | -1.063  0.548
## 2  |  1.491 |  1.100  0.484  0.544 |  0.558  0.142  0.140 |  0.385  0.072
## 3  |  3.029 |  0.049  0.001  0.000 | -1.532  1.068  0.256 |  2.131  2.206
## 4  |  3.819 |  1.685  1.135  0.195 |  1.020  0.473  0.071 | -2.009  1.960
## 5  |  1.420 |  0.247  0.024  0.030 | -0.344  0.054  0.059 | -0.750  0.273
## 6  |  1.950 |  0.458  0.084  0.055 |  0.678  0.209  0.121 |  1.758  1.502
## 7  |  1.757 |  1.190  0.566  0.459 |  0.599  0.163  0.116 |  0.713  0.247
## 8  |  1.849 |  0.050  0.001  0.001 | -0.829  0.312  0.201 |  1.180  0.676
## 9  |  2.970 | -1.708  1.166  0.331 |  1.335  0.811  0.202 |  0.268  0.035
## 10 |  3.288 | -1.781  1.269  0.294 | -0.394  0.071  0.014 |  0.202  0.020
##      cos2  
## 1   0.269 |
## 2   0.067 |
## 3   0.495 |
## 4   0.277 |
## 5   0.279 |
## 6   0.813 |
## 7   0.165 |
## 8   0.407 |
## 9   0.008 |
## 10  0.004 |
## 
## Variables
##       Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3    ctr   cos2  
## V1 |  0.006  0.003  0.000 |  0.809 59.568  0.655 |  0.343 11.401  0.117 |
## V2 |  0.744 44.326  0.554 | -0.111  1.126  0.012 | -0.218  4.610  0.047 |
## V3 |  0.421 14.172  0.177 | -0.491 21.942  0.241 |  0.313  9.506  0.098 |
## V4 |  0.720 41.478  0.519 |  0.399 14.492  0.159 |  0.019  0.036  0.000 |
## V5 |  0.016  0.022  0.000 | -0.178  2.872  0.032 |  0.875 74.447  0.766 |
```

### Special cases and related concepts
* Sparse PCA
* Principal component regression
* Partial least squares

Factor Analysis
-----
### What is factor analysis
### Difference between EFA and PCA


Independent Component Analysis
-----

```r
library(fastICA)
```

Multidimensional Scaling
-----

Nonnegative Matrix Factorization
-----

Kernel Methods
-----
### What is a kernel?
