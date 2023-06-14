---
title: "Plotting in R"
output: 
  html_document: 
    keep_md: yes
date: "2023-06-06"
---

## Plotting

The most basic way to visualize data in R is the `plot()` function.
At a minimum it takes two arguments, `plot(x, y)`.
`x` is a numeric vector that will be the x-axis coordinates of the plot
`y` is a numeric vector (of the same length as x) that will be the y-axis 
coordinates of the plot.



```r
benzene <- c(1.3, 4.5, 2.6, 3.4, 6.4)
day <- c(1, 2, 3, 4, 5)
plot(x = day, y = benzene)
```

![](readme_files/figure-html/unnamed-chunk-1-1.png)<!-- -->


Typically you will be plotting some existing data that you've imported into a 
`data.frame`. We will be using the `chicago_air` `data.frame` to create our air
quality graphs. Use the `data()` function to load the data from the `region5air`
package.


```r
#library(devtools)
#install_github("natebyers/region5air")
library(region5air)
data(chicago_air)
head(chicago_air)
```


```
##         date ozone temp solar month weekday
## 1 2013-01-01 0.032   17  0.65     1       3
## 2 2013-01-02 0.020   15  0.61     1       4
## 3 2013-01-03 0.021   28  0.17     1       5
## 4 2013-01-04 0.028   18  0.62     1       6
## 5 2013-01-05 0.025   26  0.48     1       7
## 6 2013-01-06 0.026   36  0.47     1       1
```


We will start with some basic graphs of the Chicago ozone data and then create
some overlay plots. Then we will use the `lattice` and `ggplot2` packages to 
display the same data in alternative ways.

## Base graphics

First, we create a scatterplot of temperature on the x-axis and ozone on the y-axis`.


```r
plot(chicago_air$temp, chicago_air$ozone)
```

![](readme_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

To see data plotted over time, we need to convert the `date` column to a `Date`
data type. 


```r
chicago_air$date <- as.Date(chicago_air$date)
```


Here is ozone plotted by day as a line graph.


```r
plot(chicago_air$date, chicago_air$ozone, type = 'l')
```

![](readme_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


You can also place multiple graphs on the same plotting screen. Use the `par()`
function to define the grid for plotting. The `mfrow` parameter takes a vector
of two values. The first value is the number of rows and the second is the number
of columns. Below we create a plot of ozone over time and temperature over time
in one row with two columns. See `?plot` for a description of the parameters
used.


```r
par(mfrow = c(1,2))

plot(chicago_air$date, chicago_air$ozone, 
     type='l', 
     pch = 16,
     col = "purple", 
     lwd = 2.5,
     xlab="Date", 
     ylab = 'Ozone (ppm)', 
     main = 'Chicago Ozone Data')

plot(chicago_air$date, chicago_air$temp, 
     type='l', 
     xlab = "Date",
     ylab= 'Ozone (ppm)',
     main = 'Chicago Temp Data')
```

![](readme_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

```r
dev.off()  # Must use dev.off to clear the plot area setting introduced by par()
```

```
## null device 
##           1
```

Be sure to use `dev.off()` to clear the plot area set by `par()`.

Below is a bar plot with a reference line added to it, representing the NAAQS
standard.


```r
bar_plot <- barplot(height = chicago_air$ozone, 
                    names.arg = chicago_air$date,
                    xlab="Date", 
                    ylab="Ozone (ppm)", 
                    main = 'Chicago Ozone Data')

abline(h=.070, col="red")
```

![](readme_files/figure-html/unnamed-chunk-8-1.png)<!-- -->


The `abline()` function can be used to make vertical lines, or any line using
the intercept and slope.

## lattice

`lattice` is a popular R graphics package.


```r
install.packages("lattice")
library(lattice)
```



To make a line p, use the `xyplot()` function, which takes a formula
as the first argument. In R, a formula such as `y = x` is written with the tilda
character, `y ~ x`. We will provide the `chicago_air` `data.frame` and use the
formula `ozone ~ date` to make a line plot of ozone values over time (i.e. ozone
values on the y-axis and dates on the x axis).


```r
xyplot(ozone ~ date, 
       data = chicago_air, 
       type = "l", 
       ylab = "Ozone (ppm)")
```

![](readme_files/figure-html/unnamed-chunk-11-1.png)<!-- -->


We can create a bar graph using the parameter `type = "h"`.


```r
ozone_lattice <- xyplot(ozone ~ date,
                        data = chicago_air, 
                        type = "h", 
                        ylab = "Ozone (ppm)")
plot(ozone_lattice) 
```

![](readme_files/figure-html/unnamed-chunk-12-1.png)<!-- -->



The advantage of the lattice package is that it allows you to easily split up the
data by conditioning. We’ll create some variables that will allow us to split up
our data by time periods. This is done using the factor function in R.

### Conditioning using factor()

Factors are variables in R that are treated as categorical variables or 'levels'
of data. This can be helpful for graphing and statistical summaries. First we'll
make a generic factor variable to see how it works.


```r
data <- c(1,2,2,3,1,2,3,3,1,2,3,3,1)
fdata <- factor(data)
fdata
```

```
##  [1] 1 2 2 3 1 2 3 3 1 2 3 3 1
## Levels: 1 2 3
```

The levels are displayed when the factor variable is printed. You can also give
the levels labels when you create the variable.


```r
display.data <- factor(data, labels = c("I","II","III"))
display.data
```

```
##  [1] I   II  II  III I   II  III III I   II  III III I  
## Levels: I II III
```


In the `chicago_air` `data.frame` we will create a factor columns for months 
and weekdays to split our data into groups. The functions `month()` and
`weekdays()` will extract the month and weekday from the `date` column. We'll
provide labels for the factor levels to ensure that the months and days are in
chronological order (the default is alphabetical).


```r
chicago_air$month <- factor(months(chicago_air$date), 
                            labels = month.name) # month.name is a built-in vector

chicago_air$weekday <- factor(weekdays(chicago_air$date, abbreviate = TRUE),
                              labels = c("Sun", "Mon", "Tue", "Wed", "Thu", 
                                         "Fri", "Sat"))
```

Now we'll create a `lattice` box plot using the `bwplot()` function. The formula
will be `ozone ~ weekday | month` to put weekdays on the x-axis, ozone on the 
y-axis, and condition the plot by month.


```r
bwplot(ozone ~ weekday | month, 
       data = chicago_air,
       ylab = "Ozone (ppm)",
       xlab = "2013", 
       pch = '|',
       scales = list(x=list(rot=45)))
```

![](readme_files/figure-html/unnamed-chunk-16-1.png)<!-- -->


## ggplot2

The `ggplot2` package is also useful for displaying relationships between variables
by conditioning. 


```r
install.packages("ggplot2")
```

```
## Installing package into '/cloud/lib/x86_64-pc-linux-gnu-library/4.3'
## (as 'lib' is unspecified)
```

```r
library(ggplot2)
```



The `ggplot()` function takes a `data.frame` as the first argument and an "aesthetic
mapping" using the `aes()` function. Additional information for the plot is added to the
function with a `+`, usually in the form of a `geom_*()` function. Below is a 
line graph of ozone over time.


```r
ggplot(chicago_air, aes(x = date, y = ozone)) + geom_line()
```

![](readme_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

Before we can plot our `chicago_air` data using conditional columns in `ggplot2`
we must reshape this "wide" `data.frame` into a "long" format. Below we create a 
long `data.frame` using the `pivot_long()` function from the `tidyr` package.
We also create a new column named `high_temp` to create different colors on our
graphs.


```r
# install.packages("tidyr")
library(tidyr)

chicago_air$high_temp <- chicago_air$temp > 80

chicago_long <- pivot_longer(chicago_air, 
                             cols = c("ozone", "temp", "solar"),
                             names_to = "variable",
                             values_to = "value",
                             values_drop_na = TRUE) # drop NAs

head(chicago_long)
```

```
## # A tibble: 6 × 6
##   date       month weekday high_temp variable  value
##   <date>     <fct> <fct>   <lgl>     <chr>     <dbl>
## 1 2013-01-01 May   Fri     FALSE     ozone     0.032
## 2 2013-01-01 May   Fri     FALSE     temp     17    
## 3 2013-01-01 May   Fri     FALSE     solar     0.65 
## 4 2013-01-02 May   Sat     FALSE     ozone     0.02 
## 5 2013-01-02 May   Sat     FALSE     temp     15    
## 6 2013-01-02 May   Sat     FALSE     solar     0.61
```

To color high temperature values, we set the `color` argument in the `aes()` 
function to the `high_temp` column. To plot each variable on its own graph in
stacked rows, we use the `facet_grid()` function with the argument `rows` set 
to the variable column.


```r
ggplot(chicago_long, aes(x = date, y = value, color = high_temp)) + 
  geom_point() +
  facet_grid(rows = "variable", scales="free_y")
```

![](readme_files/figure-html/unnamed-chunk-21-1.png)<!-- -->
## Plotting confidence intervals

`ggplot2` also makes it easy to plot shaded confidence intervals on graphs.
The function `geom_smooth()` will create a fitted line to the points on the
graph below, along with a 95% confidence shaded area.


```r
ggplot(chicago_air, aes(temp, ozone) ) +
  geom_point() + 
  geom_smooth(method=lm)
```

![](readme_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

You can also fit a loess curve for modeling local relationships in a nonlinear
curve.


```r
ggplot(chicago_air, aes(temp, ozone) ) +
  geom_point() + 
  geom_smooth(method=loess)
```

![](readme_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

## Saving Plots

Plots can be saved in RStudio using the "Export" button at the top of the "Plots"
pane.

![](img/save_plot.png)

You can also save a plot made by `ggplot2` using the `ggsave()` function..


```r
my_plot <- ggplot(chicago_air, aes(temp, ozone) ) +
  geom_point() + 
  geom_smooth(method=loess)

ggsave(filename = "my_plot.png", plot = my_plot)
```
