Dimensionality Reduction with PCA
========================================================


```r
library(ggplot2)
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
library(car)
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
data(UScereal, package = "MASS")
```



```r
# PCA can only be done on numeric variables
numericcols <- sapply(UScereal, is.numeric)
# Perform PCA, skip the plots for now
cereal.pca <- PCA(UScereal[numericcols])
```

![plot of chunk cerealPCA](figure/cerealPCA1.png) ![plot of chunk cerealPCA](figure/cerealPCA2.png) 

### Interpreting PCA
#### Eigenvalues
The __eigenvalue__ of a principal component is the amount of variance explained by that component. These are in order, so the first eigenvalue will be the largest and so on. We can retrieve them from `cereal.pca$eig`.


```r
cereal.pca$eig
```

```
##        eigenvalue percentage of variance cumulative percentage of variance
## comp 1   4.507994               50.08882                             50.09
## comp 2   1.359057               15.10063                             65.19
## comp 3   1.194672               13.27413                             78.46
## comp 4   0.738865                8.20961                             86.67
## comp 5   0.593874                6.59860                             93.27
## comp 6   0.468633                5.20704                             98.48
## comp 7   0.101423                1.12692                             99.61
## comp 8   0.028421                0.31579                             99.92
## comp 9   0.007062                0.07847                            100.00
```

As we can see, we've gotten 9 PCs from the data, ordered by the amount of variance explained by the component. The first accounts for 50.0888% of the variance in the data. The first three PCs account for 78.4636% of the data. All of the PCs explain all of the variance in the data.
#### How many components should I use?
The eigenvalues of the PCs tell us how many it is useful to retain or examine. But there's not a really good way to objectively get it. 
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


```r
fmscree(cereal.pca)
```

![plot of chunk screereal](figure/screereal.png) 

#### Loadings
The __loadings__ describe the relationships of the _variable_ to each of the components. A positive value for a loading indicates that an increased value for that variable results in an increased value for that PC. Variable level information is available in `cereal.pca$var`. The loadings themselves are in `cereal.pca$var$coord`.

```r
cereal.pca$var$coord
```

```
##            Dim.1   Dim.2    Dim.3     Dim.4     Dim.5
## calories  0.8435  0.4190 -0.27970 -0.056451  0.008425
## protein   0.9106 -0.2351 -0.08269 -0.013654 -0.152761
## fat       0.5610  0.5195  0.21762 -0.090330 -0.541080
## sodium    0.6960 -0.1273 -0.16472 -0.360982  0.337697
## fibre     0.7809 -0.5359  0.24877 -0.001072 -0.066369
## carbo     0.5860  0.1571 -0.76765  0.098056  0.038199
## sugars    0.4144  0.5722  0.52017 -0.227678  0.342911
## shelf     0.5648  0.1765  0.19356  0.731655  0.198482
## potassium 0.8510 -0.4144  0.27550 -0.015867 -0.028513
```

FactoMineR has a convenient way of plotting loadings, by calling `plot` on PCA objects.

```r
# choix chooses between individuals and variables.  Its 'choix' because the
# FactoMineR people are French.
plot(cereal.pca, choix = "var")
```

![plot of chunk plotvariablecereal](figure/plotvariablecereal.png) 

When you look at the plot you can already see a divide between sugary cereals (higher scores on PC2) vs fibery cereals (lower scores on PC2), but its hard to say anything about PC1, since all of the loadings are positive.
#### Contributions
Related to the loadings, the __contributions__ describe how much of a PC is due to a variable. These are available in `cereal.pca$var$contrib`. 

```r
round(cereal.pca$var$contrib, 2)
```

```
##           Dim.1 Dim.2 Dim.3 Dim.4 Dim.5
## calories  15.78 12.92  6.55  0.43  0.01
## protein   18.39  4.07  0.57  0.03  3.93
## fat        6.98 19.86  3.96  1.10 49.30
## sodium    10.75  1.19  2.27 17.64 19.20
## fibre     13.53 21.13  5.18  0.00  0.74
## carbo      7.62  1.82 49.33  1.30  0.25
## sugars     3.81 24.09 22.65  7.02 19.80
## shelf      7.08  2.29  3.14 72.45  6.63
## potassium 16.06 12.63  6.35  0.03  0.14
```

These are percents, and every column sums to 100%. If we look at PC1, we see that it's driven by protein, potassium, and calories mostly. Sugars and fat don't really enter into it. PC2, on the other hand is almost all sugars, calories, and fats. Be sure to check the loadings if you want a good understanding of what's going on with your PCs.
#### PC Scores
The __PC score__ describes where on the PC an _observation_ is. This is typically the most useful part of PCA, because it describes where your sample is in the reduced-dimension principal component space. Plotting them is the usual thing to do. If you have useful labels, use them in the plot.

```r
# You can plot, just like before with plot(cereal.pca,choix='ind') but the
# labels get a little crowded so I'm going to use ggplot
scoredf <- data.frame(PC1 = cereal.pca$ind$coord[, 1], PC2 = cereal.pca$ind$coord[, 
    2], brand = rownames(UScereal))
ggplot(scoredf, aes(x = PC1, y = PC2, label = brand)) + geom_point() + xlim(-5, 
    10)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-31.png) 

```r
ggplot(scoredf, aes(x = PC1, y = PC2, label = brand)) + geom_text(size = 3, 
    alpha = 0.75) + xlim(-5, 10)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-32.png) 


With how long the labels are, the graph of the PCs is hard to read. Maybe we'll get some insight on PC1 by looking at the extremes of the distribution.

```r
summary(scoredf$PC1)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  -2.770  -1.460  -0.666   0.000   0.801   7.140
```

```r
Q1 <- summary(scoredf$PC1)[["1st Qu."]]
Q3 <- summary(scoredf$PC1)[["3rd Qu."]]
# The low scorers
subset(scoredf, PC1 < Q1, select = c(PC1, PC2))
```

```
##                         PC1     PC2
## Apple Jacks          -1.722  0.2416
## Cocoa Puffs          -1.590  0.5019
## Corn Chex            -1.730 -0.7413
## Corn Flakes          -1.753 -1.0040
## Corn Pops            -1.998  0.2303
## Count Chocula        -1.567  0.4820
## Froot Loops          -1.594  0.4294
## Golden Crisp         -2.034  0.4493
## Honey-comb           -2.671 -0.4217
## Kix                  -2.210 -0.6659
## Multi-Grain Cheerios -1.647 -0.6143
## Puffed Rice          -2.767 -0.7922
## Rice Chex            -2.257 -0.8058
## Rice Krispies        -1.682 -0.7696
## Trix                 -1.759  0.5265
## Wheaties             -1.464 -1.0119
```

```r

# The high scorers
subset(scoredf, PC1 > Q3, select = c(PC1, PC2))
```

```
##                                          PC1      PC2
## 100% Bran                             5.9600 -2.58111
## All-Bran                              7.1416 -3.08459
## All-Bran with Extra Fiber             2.6422 -4.62435
## Clusters                              2.0732  1.37038
## Cracklin' Oat Bran                    2.6876  1.29875
## Fruit & Fibre: Dates Walnuts and Oats 1.4667  0.51943
## Fruitful Bran                         1.4513 -0.06441
## Grape-Nuts                            6.9624  0.36009
## Great Grains Pecan                    5.0824  2.88487
## Mueslix Crispy Blend                  1.8938  1.72182
## Nutri-Grain Almond-Raisin             1.7604  0.88614
## Oatmeal Raisin Crisp                  2.7234  2.09258
## Post Nat. Raisin Bran                 1.8195  0.25373
## Quaker Oat Squares                    1.9441  0.40633
## Raisin Bran                           0.8451 -0.10079
## Raisin Nut Bran                       2.0654  1.12906
```

Interpreting PCs is always subjective, but looking at this I think it's clear that PC1 is separating kid's cereal (Fruit Loops, Count Chocula) from grown up cereal (All the bran cereals). An equally valid interpretation is that PC1 is separating candy cereals from healthy cereals.

What about PC2?

```r
summary(scoredf$PC2)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  -4.620  -0.666   0.224   0.000   0.574   2.880
```

```r
Q1 <- summary(scoredf$PC2)[["1st Qu."]]
Q3 <- summary(scoredf$PC2)[["3rd Qu."]]
# The low scorers
subset(scoredf, PC2 < Q1, select = c(PC1, PC2))
```

```
##                                PC1     PC2
## 100% Bran                  5.96001 -2.5811
## All-Bran                   7.14164 -3.0846
## All-Bran with Extra Fiber  2.64216 -4.6244
## Bran Flakes                0.64965 -1.1962
## Cheerios                  -1.39716 -1.1670
## Corn Chex                 -1.73009 -0.7413
## Corn Flakes               -1.75266 -1.0040
## Product 19                -0.85156 -0.6939
## Puffed Rice               -2.76741 -0.7922
## Rice Chex                 -2.25670 -0.8058
## Rice Krispies             -1.68239 -0.7696
## Shredded Wheat 'n'Bran    -0.99629 -1.5235
## Shredded Wheat spoon size -1.10435 -1.3276
## Special K                 -1.27171 -1.2382
## Wheat Chex                 0.08802 -0.8006
## Wheaties                  -1.46383 -1.0119
```

```r
# The high scorers
subset(scoredf, PC2 > Q3, select = c(PC1, PC2))
```

```
##                               PC1    PC2
## Apple Cinnamon Cheerios   -0.6964 0.6418
## Basic 4                    0.8008 0.8282
## Cap'n'Crunch              -0.4447 1.4604
## Cinnamon Toast Crunch     -0.3240 1.4909
## Clusters                   2.0732 1.3704
## Cracklin' Oat Bran         2.6876 1.2988
## Fruity Pebbles            -1.0184 1.1628
## Golden Grahams            -0.5170 0.6507
## Great Grains Pecan         5.0824 2.8849
## Just Right Fruit & Nut     0.6535 0.7561
## Mueslix Crispy Blend       1.8938 1.7218
## Nut&Honey Crunch          -0.1140 0.9913
## Nutri-Grain Almond-Raisin  1.7604 0.8861
## Oatmeal Raisin Crisp       2.7234 2.0926
## Raisin Nut Bran            2.0654 1.1291
## Smacks                    -0.9345 1.2464
```

This one is not as clear as the others, but by looking at this in combination with the loadings for PC2 from the previous section, we can say that that PC2 is separating sugary cereals and fibery cereals.

You can keep digging through for more information from additional PCs, but I couldn't pick out a good pattern from the loadings for PC3 (lower for increased calories but higher for increased sugar?), and since it already has a low eigenvalue (1.1947), I'll call the rest of the variation noise.

### More Examples
#### Comparing cars by their specs.
The `mtcars` dataset has specs and performance for 32 cars from an issue of Motor Trend from 1972. We're going to use 9 of the predictors (skipping two categorical ones). When we look at the correlations, we see there's a good amount of 

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

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

Just by eyeballing it, you can see that PC1 (accounting for ~63% of the variance in the dataset) measures the 'fanciness' of the car, as well as it can by specs and performance. In the negative direction you have economy cars, like Hondas and Fiats, as well as a few higher end models which have lower specification. On the other end you have higher end luxury cars: Maseratis, Chryslers, Cadillacs, etc. 


Principal component 2 seems to separate cars based on performance. The Ferrari, Maserati, and the Pantera (a sports car) all have high scores on this PC. On the other hand, you have cars like the Toyota Corona, the Hornet, and especially the Valiant are very poorly performing cars. 


```r
plot(mtcars.pca, choix = "var")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

This plot shows the relationship of the *variables* to the two principal components. Cars that get better mileage are usually economy cars, and mpg shows that. A larger value for mpg results in a lower score on PC1 (the fancy car PC). On the other hand, fancier cars usually have bigger engines, and larger numbers of cylinders, increased displacement, and weight all results in a higher score on PC1. 

Cars that perform poorly on the quarter mile get lower scores on PC2. These are also typically the lower spec'ed cars, so increased values of qsec result in lower scores also get you a lower score on PC1. The opposite is true for number of carbs and horsepower. 
#### Outlier detection: The Florida 2000 election.
It can be hard to identify outliers in high dimensional spaces, where there are so many things going on. A great place to look for weird things happening is the 2000  Florida election, where weird things did happen. The dataset `Florida` in package `car` has vote counts for each florida county, for 10 presidential candidates. What happens when you look at the vote counts with PCA.  

```r
data(Florida, package = "car")
fl.pca <- PCA(Florida[1:10], graph = F)
plot(fl.pca, choix = "ind")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 

Lets look at the first two PCs. So theres an expected group around 0,0. And you can see the large population counties (Dade, Hillsborough, etc) off a little farther. They're going to separate out in any analysis not normalized by population size because their populations are so much larger than the other counties. But the real star of this plot is Volusia county, which is far, far away on PC2 from the rest of the counties. 

```r
plot(fl.pca, choix = "var")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

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

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 

#### You can get prinicpal components out of nothing.
Principal components can always be found. That doesn't necessarily mean they mean something. Here's some randomly generated data: 200 'observations' of 5 variables, all random normal variates. 

```r
summary(PCA(matrix(rnorm(1000), ncol = 5)))
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-111.png) ![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-112.png) 

```
## 
## Call:
## knit("dimensionality_reduction.Rmd", encoding = "UTF-8") 
## 
## 
## Eigenvalues
##                        Dim.1   Dim.2   Dim.3   Dim.4   Dim.5
## Variance               1.142   1.126   1.010   0.908   0.814
## % of var.             22.840  22.514  20.198  18.165  16.283
## Cumulative % of var.  22.840  45.354  65.552  83.717 100.000
## 
## Individuals (the 10 first)
##        Dist    Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3    ctr
## 1  |  2.523 | -2.072  1.880  0.675 |  0.159  0.011  0.004 | -0.224  0.025
## 2  |  1.795 |  0.397  0.069  0.049 |  1.237  0.680  0.475 |  0.585  0.169
## 3  |  2.086 | -0.427  0.080  0.042 | -0.004  0.000  0.000 | -1.014  0.509
## 4  |  1.947 |  0.390  0.067  0.040 | -1.104  0.541  0.321 |  1.485  1.092
## 5  |  2.813 |  1.584  1.099  0.317 |  0.485  0.105  0.030 | -1.533  1.163
## 6  |  1.978 |  0.146  0.009  0.005 |  0.498  0.110  0.063 | -1.080  0.578
## 7  |  2.000 | -1.490  0.972  0.555 |  0.389  0.067  0.038 | -1.022  0.517
## 8  |  3.049 | -1.258  0.693  0.170 |  2.236  2.221  0.538 | -0.674  0.225
## 9  |  2.047 |  0.557  0.136  0.074 |  0.854  0.324  0.174 |  0.795  0.313
## 10 |  1.394 | -0.052  0.001  0.001 | -1.229  0.671  0.777 |  0.355  0.062
##      cos2  
## 1   0.008 |
## 2   0.106 |
## 3   0.236 |
## 4   0.582 |
## 5   0.297 |
## 6   0.298 |
## 7   0.261 |
## 8   0.049 |
## 9   0.151 |
## 10  0.065 |
## 
## Variables
##       Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3    ctr   cos2  
## V1 |  0.666 38.863  0.444 |  0.186  3.076  0.035 |  0.296  8.661  0.087 |
## V2 | -0.485 20.625  0.236 |  0.292  7.552  0.085 |  0.678 45.547  0.460 |
## V3 |  0.650 37.014  0.423 |  0.109  1.047  0.012 |  0.333 10.969  0.111 |
## V4 | -0.161  2.283  0.026 |  0.781 54.174  0.610 |  0.095  0.890  0.009 |
## V5 | -0.118  1.215  0.014 | -0.620 34.151  0.384 |  0.585 33.933  0.343 |
```

### Special cases and related concepts
* Sparse PCA
* Principal component regression

Kernel Methods
-----
### What is a kernel?
