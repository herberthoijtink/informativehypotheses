# Monte Carlo Simulations


```r
library(tidyverse)
# {-} no chapter number
# see Appendix.Rmd how to make parts in a book
# don't forget library(bookdown)
# R - help -cheatsheets
```

## The Confidence Interval

In this exercise I will try to repeat the example given by [Gerko Vink](https://www.gerkovink.com/markup/Wk1/Solution_to_Ex1.html)

The main idea of this exercise is to illustrate the nature of the *Confidence Interval* as described by [Neyman (1934)](http://www.stat.cmu.edu/~brian/905-2008/papers/neyman-1934-jrss.pdf)

We set a seed to make our results reproducible:

```r
set.seed(6465)
```

 - The first step is to take 100 samples (in this case of size 800) from a *normal distributuon* with $\mu = 0$ and $\sigma = 1$:

```r
samples <-plyr::rlply(100, rnorm(800, 0, 1))
```

 - Secondly, we need to calculate for the mean of each sample: the absolute bias; standard error lower bound of the 95% confidence interval and upper bound of the 95% confidence interval.

We can construct a function that does this:

```r
samp_function <- function(x) {
 
  m <- mean(x)
  n <- length(x)
  se <- 1/sqrt(n)
  bias <- abs(-0 - m)
  df <- n - 1
  interval <- qt(.975, df) * se
  return(c(m, bias, se, m - interval, m + interval))

}

format <- c("Mean" = 0, "Bias" = 0, "Std.Err" = 0, "Lower" = 0, "Upper" = 0)
```

Now we use the constructed function `samp_function` on all 100 samples contained in the object `samples`. And we also add a new column to the results that indicates which CI of the respective samples does contain $\mu$.


```r
results <- samples %>%
  vapply(., samp_function, format) %>%
  t %>%
  as_tibble %>% 
  mutate(Covered = ifelse(Lower < 0 & Upper > 0, 1, 0))
```

We can also add a table with the sample statistics of the samples whose CI's do not contain $\mu$.


```r
# the results contains the table content that is piped into the table
results %>%
  filter(Covered ==0) %>%
  kableExtra::kable(caption = "Here is a table of the samples" )
```

<table>
<caption>(\#tab:unnamed-chunk-6)Here is a table of the samples</caption>
 <thead>
  <tr>
   <th style="text-align:right;"> Mean </th>
   <th style="text-align:right;"> Bias </th>
   <th style="text-align:right;"> Std.Err </th>
   <th style="text-align:right;"> Lower </th>
   <th style="text-align:right;"> Upper </th>
   <th style="text-align:right;"> Covered </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:right;"> -0.0945589 </td>
   <td style="text-align:right;"> 0.0945589 </td>
   <td style="text-align:right;"> 0.0353553 </td>
   <td style="text-align:right;"> -0.1639592 </td>
   <td style="text-align:right;"> -0.0251585 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:right;"> 0.0740058 </td>
   <td style="text-align:right;"> 0.0740058 </td>
   <td style="text-align:right;"> 0.0353553 </td>
   <td style="text-align:right;"> 0.0046055 </td>
   <td style="text-align:right;"> 0.1434062 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table>

And finally we can also make a nice plot illustrating everything that we did so far.


```r
# in the line above also the size of the figure can be adjusted
lims <- aes(ymax = results$Upper, ymin = results$Lower)
ggplot(results, aes(y=Mean, x=1:100, colour = Covered)) + 
  geom_hline(aes(yintercept = 0)) + 
  geom_pointrange(lims) + 
  xlab("Simulations") +
  ylab("Means and 95% Confidence Intervals")
```

```
## Warning: Use of `results$Upper` is discouraged. Use `Upper` instead.
```

```
## Warning: Use of `results$Lower` is discouraged. Use `Lower` instead.
```

<div class="figure">
<img src="Chapter_2_files/figure-epub3/unnamed-chunk-7-1.png" alt="Here is a plot of the CI's"  />
<p class="caption">(\#fig:unnamed-chunk-7)Here is a plot of the CI's</p>
</div>
In this case only two out of 100 CI's do not include the true population mean.


## The Central Limit Theorem

Here we will also try to illustrate the [Central Limit Theorem](https://en.wikipedia.org/wiki/Central_limit_theorem), in it's most basic form, with a very simple example.

First we draw 1000 samples (again of size 800), form , say, a *Poisson* distribution, of course we could've drawn them from a uniform or an exponential as well.



Now we calculate the mean for each sample:

```r
means <- samples_2 %>%
  lapply(., mean) %>%
  as.data.frame() %>%
  t()
```

And now we plot a histogram of the resulting means:

```r
hist(t(means))
```

<div class="figure">
<img src="Chapter_2_files/figure-epub3/unnamed-chunk-10-1.png" alt="Histogram of the sampling distribution of the mean"  />
<p class="caption">(\#fig:unnamed-chunk-10)Histogram of the sampling distribution of the mean</p>
</div>

