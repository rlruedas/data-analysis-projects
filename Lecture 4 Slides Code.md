<style>
  .reveal pre {
    font-size: 13pt;
  }
  .reveal section p {
    font-size: 32pt;
  }
  .reveal div {
    font-size: 30pt;
  }
  .reveal h3 {
    color: #484848;
    font-weight: 150%;
  }
</style>

Lecture 4: Graphics part 2
====
author: CPECGM2
font-family: Helvetica
autosize: false
width:1440
height:900


Agenda
=====

* customizing labels, font size, etc.
* `stat_summary` for error bars and parallel coordinates with error bars
* making choropleth maps with `geom_polygon`

Here are the libraries we will need

```r
library(plyr)
library(ggplot2)
library(reshape2)
```


Customizing appearances
======

Make a table of avg `cty` mileage:


```r
mpg.means = ddply(mpg, 'class', summarize, avg.cty = mean(cty))
mpg.means
```

```
       class  avg.cty
1    2seater 15.40000
2    compact 20.12766
3    midsize 18.75610
4    minivan 15.81818
5     pickup 13.00000
6 subcompact 20.37143
7        suv 13.50000
```

=====

Our basic plot:


```r
ggplot(data = mpg.means, mapping=aes(x=class, y = avg.cty, group=1)) + geom_bar(stat='identity')
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

Reording the factor levels
=========

Bad: `class` levels are ordered alphabetically by default

`reorder()`: reorders the levels of a factor


```r
# check the order of the levels
levels(mpg.means$class)
```

```
NULL
```

```r
# the reorder command changes the ordering of the vehicle classes
mpg.means$class = with(mpg.means, reorder(class, avg.cty, mean))
# what is the new order of the levels?
levels(mpg.means$class)
```

```
[1] "pickup"     "suv"        "2seater"    "minivan"    "midsize"   
[6] "compact"    "subcompact"
```

=======

Here are the arguments for `reorder()`:

argument of `reorder` | description
---------|------------
`x` (first argument) | The vector that you want reordered
`X` (second argument) | A numeric vector that determines the ordering of `x`
`FUN=mean` (third argument) | If multiple values in `X` are associated with a single factor level, then take their average (or `max` or `min` if you prefer)

=========

Using `reorder()` makes the plot easier to read:


```r
mpg.plot = ggplot(data = mpg.means, mapping=aes(x=class, y = avg.cty, group=1)) + geom_bar(stat='identity')
mpg.plot
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-5-1.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" style="display: block; margin: auto;" />

Manually ordering and removing categorical levels in a plot
=========

`scale_x_discrete()`: manually choose which levels to display and their ordering (for discrete variables). 


```r
# Select 4 classes and their ordering
mpg.plot + scale_x_discrete(limits = c('compact', 'midsize', 'suv', 'pickup'), labels = c('Compact', 'Midsize', 'SUV', 'Truck'))
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-6-1.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" style="display: block; margin: auto;" />

`scale_y_discrete()`: same thing but for a discrete y-axis.

Labeling the axes and title
========

Use `labs()`:


```r
mpg.plot = mpg.plot + labs(x = '', y = 'Miles Per Gallon', title='City Mileage by Class') 
mpg.plot
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-7-1.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" style="display: block; margin: auto;" />

=========

For long labels, we can use the special character `\n` to denote a line break:

```r
mpg.plot + labs(title='Average City Mileage \n (Grouped by Class)') 
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-8-1.png" title="plot of chunk unnamed-chunk-8" alt="plot of chunk unnamed-chunk-8" style="display: block; margin: auto;" />

Changing the overall font
==========

`theme()`: controls a lot of things, including text. You should never have tiny unreadable text in a slideshow!


```r
# change the font size to 22
mpg.plot + theme(text = element_text(size=22))
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-9-1.png" title="plot of chunk unnamed-chunk-9" alt="plot of chunk unnamed-chunk-9" style="display: block; margin: auto;" />

=======


```r
# change the font size to 22, and rotate the x axis categories
mpg.plot + theme(text = element_text(size=22), axis.text.x = element_text(angle=45, vjust=1, hjust=1))
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-10-1.png" title="plot of chunk unnamed-chunk-10" alt="plot of chunk unnamed-chunk-10" style="display: block; margin: auto;" />

`vjust` and `hjust`: vertical and horizontal justification

Using `cut` to create new discrete variables
=======

Bar charts and facets require discrete variables. 

If you want to facet a continuous variable, use `cut` or `cut_number` to create discrete levels.


```r
# make discrete levels from hwy 
mpg = mutate(mpg, hwy.cat = cut_number(hwy, n = 4))
# reorder the classes for nicer plot
mpg$class = with(mpg, reorder(class, hwy, mean))
# how many records in each class, for different hwy levels?
ggplot(data = mpg, mapping=aes(x = class)) + geom_bar() + facet_wrap('hwy.cat', ncol=1) 
```

=======

Here is the plot:

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-12-1.png" title="plot of chunk unnamed-chunk-12" alt="plot of chunk unnamed-chunk-12" style="display: block; margin: auto;" />


Drawing errorbars
======

`geom_errorbar()` takes mappings `x`, `ymin`, and `ymax`:




```r
# This plot says: treatment makes something better!
ggplot(data = small.df, mapping = aes(x = treatment, ymin = ymin, ymax = ymax, color = treatment)) + geom_errorbar(width=.3) + geom_point(mapping=aes(y=outcome), size=4)
```

<img src="Lecture 4 Slides Code-figure/unnamed-chunk-14-1.png" title="plot of chunk unnamed-chunk-14" alt="plot of chunk unnamed-chunk-14" style="display: block; margin: auto;" />

Computing errorbars
========

Here are three different functions for computing error bars:

* `mean_cl_normal()`: computes 95% CIs using a normal approximation (equal to t-test)
* `mean_cl_boot()`: 95% CIs using a bootstrap
* `mean_se()`: computes standard errors using normal approximation


































```
processing file: Lecture 4 Slides Code.rpres
Quitting from lines 197-200 (Lecture 4 Slides Code.rpres) 
Error: Hmisc package required for this function
Backtrace:
     x
  1. +-knitr::knit(...)
  2. | \-knitr:::process_file(text, output)
  3. |   +-base::withCallingHandlers(...)
  4. |   +-knitr:::process_group(group)
  5. |   \-knitr:::process_group.block(group)
  6. |     \-knitr:::call_block(x)
  7. |       \-knitr:::block_exec(params)
  8. |         +-knitr:::in_dir(...)
  9. |         \-knitr:::evaluate(...)
 10. |           \-evaluate::evaluate(...)
 11. |             \-evaluate:::evaluate_call(...)
 12. |               +-evaluate:::timing_fn(...)
 13. |               +-base:::handle(...)
 14. |               +-base::withCallingHandlers(...)
 15. |               +-base::withVisible(eval(expr, envir, enclos))
 16. |               \-base::eval(expr, envir, enclos)
 17. |                 \-base::eval(expr, envir, enclos)
 18. \-ggplot2::mean_cl_normal(mpg$cty)
Warning messages:
1: package 'ggplot2' was built under R version 4.0.5 
2: package 'reshape2' was built under R version 4.0.5 
Execution halted
```
