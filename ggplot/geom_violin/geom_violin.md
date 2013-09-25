geom_violin
======


```r
library(ggplot2)
```



```r
nclass <- 1e+05
a <- data.frame(condition = rep("A", times = nclass), value = rnorm(nclass, 
    mean = 10, sd = 5))
b <- data.frame(condition = rep("B", times = nclass), value = rnorm(nclass, 
    mean = 25, sd = 5))
data <- rbind(a, b)
```



```r
ggplot(data, aes(x = condition, y = value, fill = condition)) + geom_violin()
```

![plot of chunk figure](figure/figure1.png) 

```r
ggplot(data, aes(x = condition, y = value, fill = condition)) + geom_boxplot(alpha = 1/2) + 
    geom_violin(alpha = 1/5) + coord_flip()
```

![plot of chunk figure](figure/figure2.png) 

