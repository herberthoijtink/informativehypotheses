# Sensitivity analysis using `bain`
Load `bain`:

```r
library(bain)
```

Please see @hoijtink2019tutorial for further elaborations.

Note that, most the code below can be replaced by a call to the function `bain_sensitivity`. See the help file for this function for further instructions (call `?bain_sensitivity`).

## The `t-test` example from 3.1.1



First, we test the same hypotheses as in 3.1.1 using the minimal fraction (*J=1*) which is by default implemented in `bain`.

```r
# set a seed value
set.seed(100)
# test hypotheses with bain. The names of the means are x and y.
results <- bain(ttest, "x = y; x > y; x < y", fraction = 1)
```


```r
print(results)
```

```
## Bayesian informative hypothesis testing for an object of class t_test:
## 
##    Fit   Com   BF.u   BF.c   PMPa  PMPb 
## H1 0.183 0.016 11.583 11.583 0.853 0.794
## H2 0.777 0.500 1.554  3.481  0.114 0.107
## H3 0.223 0.500 0.446  0.287  0.033 0.031
## Hu                                 0.069
## 
## Hypotheses:
##   H1: x=y
##   H2: x>y
##   H3: x<y
## 
## Note: BF.u denotes the Bayes factor of the hypothesis at hand versus the unconstrained hypothesis Hu. BF.c denotes the Bayes factor of the hypothesis at hand versus its complement.
```

Now we execute the sensitivity analysis by interactively changing the `fraction` argument to be equal to 2 and 3, respectively:


```r
results2 <- bain(ttest, "x = y; x > y; x < y", fraction = 2)
results3 <- bain(ttest, "x = y; x > y; x < y", fraction = 3)
```


```r
print(results2)
```

```
## Bayesian informative hypothesis testing for an object of class t_test:
## 
##    Fit   Com   BF.u  BF.c  PMPa  PMPb 
## H1 0.183 0.022 8.190 8.190 0.804 0.732
## H2 0.777 0.500 1.554 3.481 0.152 0.139
## H3 0.223 0.500 0.446 0.287 0.044 0.040
## Hu                               0.089
## 
## Hypotheses:
##   H1: x=y
##   H2: x>y
##   H3: x<y
## 
## Note: BF.u denotes the Bayes factor of the hypothesis at hand versus the unconstrained hypothesis Hu. BF.c denotes the Bayes factor of the hypothesis at hand versus its complement.
```

```r
print(results3)
```

```
## Bayesian informative hypothesis testing for an object of class t_test:
## 
##    Fit   Com   BF.u  BF.c  PMPa  PMPb 
## H1 0.183 0.027 6.687 6.687 0.770 0.690
## H2 0.777 0.500 1.554 3.481 0.179 0.160
## H3 0.223 0.500 0.446 0.287 0.051 0.046
## Hu                               0.103
## 
## Hypotheses:
##   H1: x=y
##   H2: x>y
##   H3: x<y
## 
## Note: BF.u denotes the Bayes factor of the hypothesis at hand versus the unconstrained hypothesis Hu. BF.c denotes the Bayes factor of the hypothesis at hand versus its complement.
```
As it is clear by looking at the results, the $BF_{0u}$ changes from 11.58 when `fraction` = 1 to 8.19 and 6.68, when `fraction` = 2 and 3, respectively. This stems from the fact that the value of the **complexity** (which is the proportion of the prior distribution that is supported by the (*null*) hypothesis at hand) changes, thus changing the resulting BF's.
