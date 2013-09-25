ggplot2
========================================================

Hadley Wickham. "A Layered Grammar of Graphics". Journal of Computational and Graphical Statistics Vol. 19, Iss. 1, 2010 http://vita.had.co.nz/papers/layered-grammar.pdf



```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 2.15.2
```

```r
# Some sample data
set.seed(1234)
df <- data.frame(cond = factor(rep(c("A", "B"), each = 200)), rating = c(rnorm(200), 
    rnorm(200, mean = 0.8)))

# Load some datasets

# US economic time series
data(economics)
head(economics)
```

```
##         date   pce    pop psavert uempmed unemploy
## 1 1967-06-30 507.8 198712     9.8     4.5     2944
## 2 1967-07-31 510.9 198911     9.8     4.7     2945
## 3 1967-08-31 516.7 199113     9.0     4.6     2958
## 4 1967-09-30 513.3 199311     9.8     4.9     3143
## 5 1967-10-31 518.5 199498     9.7     4.7     3066
## 6 1967-11-30 526.2 199657     9.4     4.8     3018
```

```r
# Fisher's iris dataset
data(iris)
head(iris)
```

```
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
## 3          4.7         3.2          1.3         0.2  setosa
## 4          4.6         3.1          1.5         0.2  setosa
## 5          5.0         3.6          1.4         0.2  setosa
## 6          5.4         3.9          1.7         0.4  setosa
```

```r
# Data on 50k+ diamonds
data(diamonds)
head(diamonds)
```

```
##   carat       cut color clarity depth table price    x    y    z
## 1  0.23     Ideal     E     SI2  61.5    55   326 3.95 3.98 2.43
## 2  0.21   Premium     E     SI1  59.8    61   326 3.89 3.84 2.31
## 3  0.23      Good     E     VS1  56.9    65   327 4.05 4.07 2.31
## 4  0.29   Premium     I     VS2  62.4    58   334 4.20 4.23 2.63
## 5  0.31      Good     J     SI2  63.3    58   335 4.34 4.35 2.75
## 6  0.24 Very Good     J    VVS2  62.8    57   336 3.94 3.96 2.48
```


Lets face it: the base R graphics are really ugly. Lets look at this scatterplot of diamond price versus carat.

```r
plot(diamonds$price ~ diamonds$carat)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

Kind of gross, huh? Lets compare it to the equivalent graphic created by ggplot2:

```r
ggplot(diamonds, aes(x = carat, y = price)) + geom_point()
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

Thats a bit better, but more to write. We'll talk about a shorter syntax called **qplot** at the end. 

Plots in ggplot2 are made up of a couple different things. 
Aesthetics
-------------------------

Aesthetics map variables in the dataframe you want to plot to parts of the graph. For this you use the __aes()__ function. Aesthetics are usually set in the layer, but can be set in the geom to override layer settings. There are several aesthetics you can map to. In the scatterplot above you can see we've mapped x to the carat of the observation and y to the price. But if you wanted to add more information, you can change the color based on a variable, or change the shape based on a variable. Here's a higher-information graph:


```r
ggplot(diamonds, aes(x = carat, y = price, color = color, shape = cut)) + geom_point()
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 


Some aesthetics can be difficult to differentiate, especially for categorical variables. For example, making a scatter plot with shape mapped to a factor with more than 6 categories wont work. Another problem for high categrory discrete variables is the color aesthetic. For, say, a variable with 22 categories, it becomes difficult to tell value 11 from 12, and ends up looking like a big rainbow. 

Just because you can do something, doesnt mean you necessarily should. Graphs with too much information can become overwhelming and difficult to interpret. I like to stay around three variables for clarity's sake. 

Geoms
-------------------------

Geoms are the actual graphs that you make (e.g. scatterplot, line graph, etc). These are added to the graph with the plus symbol. Geoms only understand certain aesthetics: it wouldnt make much sense to map a variable to y in a histogram, for example. Geoms can be combined in a graph, which is useful for plotting the actual observation on a line graph, or drawing a regression line on a graph.  

### Histograms
**geom_hist()** _will_ complain about not setting the width of the bins. It picks the range of the x variable divided by 30 as a default but you should set it yourself anyway.


```r
ggplot(diamonds, aes(x = carat)) + geom_histogram()
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust
## this.
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-51.png) 

```r
ggplot(diamonds, aes(x = carat)) + geom_histogram(binwidth = 0.1)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-52.png) 

### Density
Density is a useful alternative to the histogram, and doesnt rely on binsizes which can alter the appearance of the distribution. 

```r
ggplot(diamonds, aes(x = carat)) + geom_density()
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

### Boxplots
Boxplots are the histograms less informative cousin, available through __geom_boxplot__.

```r
ggplot(df, aes(x = cond, y = rating, fill = cond)) + geom_boxplot()
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 


### Bar graphs 
Bar graphs are available via **geom_bar**. They default to counting the values in the x axis (**stat_bin**).

```r
ggplot(diamonds, aes(x = cut)) + geom_bar()
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 


If you have the y axis in the dataset you can tell **geom_bar** to use that by mapping a variable to the y aesthetic and then using **stat_identity**.

```r
tips <- data.frame(time = factor(c("Lunch", "Dinner"), levels = c("Lunch", "Dinner")), 
    total_bill = c(6.89, 17.23))
ggplot(tips, aes(x = time, y = total_bill)) + geom_bar(stat = "identity")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 


### Line graphs

Line graphs connect values over the x-axis, and are made with __geom_line()__. This geom accepts several useful aesthetics, including size (which controls the line width), color, and group. 


```r
ggplot(economics, aes(x = date, y = unemploy/pop)) + geom_line()
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 


The group aesthetic can be used to draw different lines per variable. But grouping will occur automatically on lines if you assign the grouping variable an aesthetic.


```r
data(Orange)
head(Orange)
```

```
##   Tree  age circumference
## 1    1  118            30
## 2    1  484            58
## 3    1  664            87
## 4    1 1004           115
## 5    1 1231           120
## 6    1 1372           142
```

```r
ggplot(Orange, aes(x = age, y = circumference, color = Tree)) + geom_line() + 
    geom_point(alpha = 0.4)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 


### Smoothing/Fit lines

ggplot2 can add useful things like regression lines or smoothing lines using **geom_smooth**. It's basic action is to add a local model fit using **loess**, which is something akin to a rolling average. The gray bands of represent the standard error of the fit, which can be turned off and on with the boolean flag, **se**.  


```r
ggplot(economics, aes(x = date, y = unemploy/pop)) + geom_line() + geom_smooth()
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using
## loess. Use 'method = x' to change the smoothing method.
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 



```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length)) + geom_point() + geom_smooth(method = lm)
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13.png) 


If you are grouping on an aesthetic that you mapped to a factor or ordered variable, **geom_smooth** will split the smoother apart: 


```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) + geom_point() + 
    geom_smooth(method = lm)
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14.png) 


### Arbitrary lines

ggplot will let you place arbitrary lines onto a graph. **geom_hline**, **geom_vline**, and **geom_abline** let you add horizontal, vertical, and arbitrary lines to a plot.

```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) + geom_point() + 
    geom_hline(yintercept = 5.1) + geom_vline(xintercept = 4) + geom_abline(intercept = 4.5, 
    slope = 0.5)
```

![plot of chunk unnamed-chunk-15](figure/unnamed-chunk-15.png) 

Coordinates
-------------------------

### Axes
Axis labels can be set with **xlab** and **ylab**. The axis ranges of graphs can be controlled by the shortcuts **xlim** and **ylim**. For example xlim(0,5) bounds the x-axis between 0 and 5. Compare: 


```r
ggplot(diamonds, aes(x = carat, y = price)) + geom_point()
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-161.png) 

```r
ggplot(diamonds, aes(x = carat, y = price)) + geom_point() + xlim(1, 2) + ylim(5000, 
    10000) + xlab("weight of diamond") + ylab("price of diamond")
```

```
## Warning: Removed 44778 rows containing missing values (geom_point).
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-162.png) 


coord_flip inverts axes:


```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) + geom_point() + 
    geom_smooth(method = lm, se = F)
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-171.png) 

```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) + geom_point() + 
    geom_smooth(method = lm, se = F) + coord_flip()
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-172.png) 


### Polar coordinates
You can add **coord_polar()** to your plot to make it plot using polar coordinates instead of Cartesian. This is usually more difficult to interpret than Cartesian and frankly kind of goofy looking so you should probably have a good reason for doing this.

```r
ggplot(economics, aes(x = date, y = unemploy/pop)) + geom_line()
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-181.png) 

```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) + geom_point() + 
    geom_smooth(method = lm) + coord_polar()
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-182.png) 


Positioning and Overplotting 
-------------------------
Instead of making two separate plots you can combine them using position adjustments to the geom. These get ugly for more than two or three groups. 

```r
# Keep the columns where they would be and just make the histograms a
# little more transparent.
ggplot(df, aes(x = rating, fill = cond)) + geom_histogram(binwidth = 0.5, alpha = 0.5, 
    position = "identity")
```

![plot of chunk unnamed-chunk-19](figure/unnamed-chunk-191.png) 

```r

# Dodge the columns
ggplot(df, aes(x = rating, fill = cond)) + geom_histogram(binwidth = 0.5, alpha = 0.5, 
    position = "dodge")
```

![plot of chunk unnamed-chunk-19](figure/unnamed-chunk-192.png) 


Faceting
-------------------------
Faceting is probably my favorite feature of ggplot2. Facetting allows you to make subplots based on one (or two variables). facet functions will take an arugment that looks like an equation: ~, y, x. Putting a variable in the x position of the equation will facet the graph across that axis.  **facet_grid** will lay them out in a grid for you:

```r
ggplot(diamonds, aes(x = carat)) + geom_histogram(binwidth = 0.1) + facet_grid(cut ~ 
    .)
```

![plot of chunk unnamed-chunk-20](figure/unnamed-chunk-201.png) 

```r
ggplot(diamonds, aes(x = carat)) + geom_histogram(binwidth = 0.1) + facet_grid(. ~ 
    cut)
```

![plot of chunk unnamed-chunk-20](figure/unnamed-chunk-202.png) 

```r
ggplot(diamonds, aes(x = carat)) + geom_histogram(binwidth = 0.1) + facet_grid(cut ~ 
    color)
```

![plot of chunk unnamed-chunk-20](figure/unnamed-chunk-203.png) 


Facetting works for any type of plot. 

```r
ggplot(diamonds, aes(x = carat, y = price, color = cut)) + geom_point() + facet_grid(color ~ 
    .)
```

![plot of chunk unnamed-chunk-21](figure/unnamed-chunk-21.png) 

Themeing
-------------------------
Many parts of the layout of the plot such as fonts, positioning, etc can be adjusted with the **theme** function. A useful thing to add to your chart is **theme_bw**, which gets rid of the background, and makes your chart look a bit more presentable. Labels can alternately be set with **labs**.

```r
ggplot(iris, aes(x = Sepal.Width, y = Sepal.Length, color = Species)) + geom_point() + 
    geom_smooth(method = lm, se = F) + theme_bw() + theme(legend.position = "bottom") + 
    labs(x = "Sepal Width", y = "Sepal Length", title = "Morphologic Variation in 3 Species of Iris")
```

![plot of chunk unnamed-chunk-22](figure/unnamed-chunk-22.png) 

There are a lot of options you can give to **theme** but they're straightforward and I dont feel like going over them. 

Programatically creating plots
-------------------------
One of the nice things about ggplot graphics is that the plots themselves are objects. So you can create them on the fly in programs. 

```r
plot <- ggplot(diamonds, aes(x = carat, y = price, color = cut))
if (nrow(diamonds) > 5000) {
    # If there are a ton of datapoints, make them transparent so that you can
    # see more of them
    plot <- plot + geom_point(alpha = 0.5)
} else {
    plot <- plot + geom_point()
}
plot
```

![plot of chunk unnamed-chunk-23](figure/unnamed-chunk-23.png) 


qplot
-------------------------
Plotting things in ggplot can be verbose. That's why **qplot** is included in ggplot2. It's intended to be used similarly to R's plot function, for common plots. If you supply one axis, **qplot** defaults to a histogram. If you supply two, it goes for a scatter plot. You can add more aesthetics with a keyword arguement. You can also add more geoms like a ggplot object


```r
qplot(Sepal.Length, data = iris)
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust
## this.
```

![plot of chunk unnamed-chunk-24](figure/unnamed-chunk-241.png) 

```r
qplot(Sepal.Width, Sepal.Length, data = iris, color = Species)
```

![plot of chunk unnamed-chunk-24](figure/unnamed-chunk-242.png) 

```r
qplot(Sepal.Width, Sepal.Length, data = iris, color = Species) + geom_smooth(method = lm, 
    se = F)
```

![plot of chunk unnamed-chunk-24](figure/unnamed-chunk-243.png) 
