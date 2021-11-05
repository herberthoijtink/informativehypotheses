
# (PART) `R` package `bain` {-}

# Introduction to `bain` 

`bain` is an acronym for **"Bayesian informative hypotheses evaluation"**.
It uses the *Bayes factor* [@kass1995bayes] to evaluate hypotheses specified using *equality* and *inequality* constraints among (linear combinations of) parameters in a wide range of statistical models. 

## Available tutorials

Two tutorials are retrievable from the `bain` [website](https://informative-hypotheses.sites.uu.nl/software/bain/) under the section **Tutorials**.

The first introducing Bayesian evaluation of informative hypotheses is provided by @hoijtink2019tutorial. By reading the tutorial in combination with executing the analyses contained in `easyBFtutorial.R` and `BFtutorial.R` (available on the aforementioned website) you will quickly learn the basics of Bayesian hypothesis evaluation. 

The second containing introduction to Bayesian evaluation of informative hypotheses in the context of structural equation models is provided by @van2021teacher^[Users are advised to read these tutorials and this vignette before using `bain`.]. 


An overview of all other relevant  papers concerning informative hypotheses testing and the package `bain` can be found [here](https://informative-hypotheses.sites.uu.nl/publications/).

## Usage

This is how a general call to `bain` looks like:

 `results  <- bain(x, hypothesis, fraction = 1, ...)`


### Arguments

- `x`
An R object containing the outcome of a statistical analysis. Currently, the following objects can be processed:

1) `t_test()` objects (Student's t-test, Welch's t-test, paired samples t-test, one-group t-test, equivalence test). Note that, `t_test` can be used in the same way as `t.test`.
2) `lm()` objects (ANOVA, ANCOVA, multiple regression).
3) `lavaan` objects generated with the `sem()`, `cfa()`, and
`growth` functions.
4) A named vector containing the estimates resulting from a statistical analysis. Using this option triggers the `bain.default()` method. Note that, named means that each estimate has to be named such that it can be referred to in hypotheses.
5) Wrapper functions for repeated measures ANOVA(...) and linear two-level models built with `lmer` (TBA when finished).

- `hypothesis` A character string containing the informative hypotheses to evaluate 
(see section 3.2.2).

- `fraction = 1` A number representing the fraction of information in the data used to construct the prior distribution (see, for example, @gu2018approximated). The default value 1 denotes the minimal fraction, 2 denotes twice the minimal fraction, etc. The examples in chapter 5 show how the `fraction` can be employed to execute a sensitivity analysis. See also @hoijtink2019tutorial for more details.

- `...`	Additional arguments (see next chapter).


### The specification of `hypotheses` 

`hypotheses` is a character string that specifies which (informative)
hypotheses have to be evaluated. A simple example is `hypotheses <- "a >
b > c; a = b = c;"` which specifies two hypotheses using three estimates with
names "a", "b", and "c", respectively.

The hypotheses specified have to adhere to the following rules (`bain` may still run if you
deviate from the rules, however, the output will be nonsense):

* When using `bain` with a `lm` or `t_test` or `lavaan` object, 
(unique abbreviations of) the names
displayed by `coef(x)` have to be used (but see the section \@ref(lavaan) for additional instructions if multiple group
analyses are executed and/or parameters are labeled). If, 
for example, the names are `cat` and `dog`, `c`
and `d ` would be unique abbreviations. If, for example, the names are `cato` 
and `cata` the whole
names have to be used.
* When using `bain` with a named vector, parameters are referred to using 
the names specified in `names()`.
* Linear combinations of parameters must be specified adhering to the
following rules:

a) Each parameter name is used at most once.
b) Each parameter name may or may not be pre-multiplied with a number.
c) A constant may be added or subtracted from each parameter name.
d) A linear combination can also be a single number.

      Examples are: `3 * a + 5`; `a + 2 * b + 3 * c - 2`; `a - b`; and `5`.

* (Linear combinations of) parameters can be constrained using <, >, and
=. For example, `a > 0` or
`a > b = 0` or `2 * a < b + c > 5`.
* The ampersand `&` can be used to combine different parts of a hypothesis.
For example, `a > b & b > c` which is equivalent to `a > b > c` or
`a > 0 & b > 0 & c > 0`.
* Sets of (linear combinations of) parameters subjected to the same
constraints can be specified using (). For
example, `a > (b,c)` which is equivalent to `a > b & a > c`.
* The specification of a hypothesis is completed by typing ; For example,
`hypotheses <- "a > b > c; a = b = c;"`, specifies two hypotheses.
* Hypotheses have to be compatible, non-redundant and possible. What
these terms mean will be elaborated below.


**The set of hypotheses has to be compatible.** For the statistical
background of this requirement see @gu2018approximated. Usually the
sets of hypotheses specified by researchers are compatible, and if not,
`bain` will return an error message. The following steps can be used to
determine if a set of hypotheses is compatible:

1)	Replace a range constraint, e.g., `1 < a1 < 3`, by an equality
constraint in which the parameter involved is equated to the midpoint of the
range, that is, `a1 = 2`.
2) Replace in each hypothesis the < and > by =. For example, 
`a1 = a2> a3 > a4`  becomes `a1 = a2 = a3 = a4`.
3) The hypotheses are compatible if there is at least one solution to the
resulting set of equations. For the two hypotheses considered under 1. and
2., the solution is `a1 = a2 = a3 = a4 = 2`. An example of two non-compatible
hypotheses is `hypotheses <- "a = 0; a > 2;"` because there is no
solution to the equations `a=0` and `a=2`.



**Each hypothesis in a set of hypotheses has to be non-redundant.** A
hypothesis is redundant if it can also be specified with fewer constraints.
For example, `a = b & a > 0 & b > 0` is redundant because it can also be
specified as `a = b & a > 0`. `bain` will work correctly if
hypotheses specified using only < and > are redundant. `bain` will
return an error message if hypotheses specified using at least one = are
redundant.


**Each hypothesis in a set of hypotheses has to be possible.** An
hypothesis is impossible if estimates in agreement with the hypothesis do not
exist. For example: values for `a, b, c` in agreement with `a > b > c &
a < c` do not exist. It is the responsibility of the user to ensure that the
hypotheses specified are possible. If not, `bain` will either return an
error message or render an output table containing `Inf`'s.


### Output

The commands `bain()` or `results<-bain()` followed by
`results` or `print(results)` will render the default (most
important) output from `bain`. These concern for each hypothesis
specified in `hypotheses` the fit, complexity, Bayes factor versus 
the unconstrained hypothesis, Bayes factor versus its
complement, posterior model probability (based on equal prior model
probabilities) excluding the unconstrained hypothesis, posterior model
probability including the unconstrained hypothesis, and posterior model
probability of each hypothesis specified and their joint complement. 
Note that, all the posterior model probabilities are computed from
the Bayes factors of each hypothesis versus the unconstrained hypothesis.
In @hoijtink2019tutorial it is elaborated how these quantities 
(and the other
output presented below) should be interpreted. Additionally, using
`summary(results, ci=0.95)`, a descriptives matrix can be obtained
in which for each estimate, the name, the value, and a 95\% central
credibility interval is presented. 

The following commands can be used to
retrieve the default and additional information from the `bain` output
object:


* `results$fit` renders the default output, `results$fit$Fit`
contains only the column containing the fit of each hypothesis. In the last
command `Fit` can be replaced by `Com`, `BF`, `BF.u`, `BF.c`, `PMPa`,
`PMPb`, `PMPc` to obtain the information in the corresponding columns of the
default output. Note that, in the columns `BF` and `BF.c` the Bayes factor of the
hypothesis at hand versus its complement is displayed. In the column `BF.u` the
Bayes factor of the hypothesis at hand versus the unconstrained hypothesis is
displayed. `PMPa` renders the posterior model probabilities (based on equal
prior model probabilities) of the hypotheses specified. `PMPb` renders 
the posterior model probabilities (based on equal
prior model probabilities) of the hypotheses specified plus the unconstrained 
hypothesis.  `PMPc` renders the posterior model probabilities (based on equal
prior model probabilities) of the hypotheses specified plus the complement of 
the union of these hypotheses. If, in the latter case, the complexity of the 
complement of the union of all hypotheses specified is smaller than .05, the
hypotheses specified (almost) completely cover the parameter space. In this 
case `PMPc` is not provided and instead `PMPa` should be used.
* `results$BFmatrix` contains the matrix containing the mutual Bayes
factors of the hypotheses specified in `hypotheses`.
* `results$b` contains for each of the groups in the analysis the
fraction of information of the data in the group at hand used to specify the
covariance matrix of the prior distribution.
* `results$prior` contains the covariance matrix of the prior
distribution.
* `results$posterior` contains the covariance matrix of the
posterior distribution.
* `results$call` displays the call to `bain`.
* `results$model` displays the named vector or the R object that is input to `bain`.
* `results$n` displays the sample sizes per group.
* `results$independent_restrictions` displays the number of
independent constraints in the set of hypotheses under consideration. Note that,
in @gu2018approximated and @hoijtink2019bayesian the
definition given was misprinted (besides R and S also r and s should have been
added to the definition).
* `results$fit$Fit_eq` displays the fit of the equality constrained
part of each hypothesis. Replacing `Fit_eq` by `Fit_in`, renders
the fit of the inequality constrained part of an hypothesis conditional on
the fit of the equality constrained part. `Com_eq`, and `Com_in`,
respectively, are the complexity counterparts of `Fit_eq`, and
`Fit_in`.

Note that, if you have specified two hypotheses that both have a small
`BF.u` (close to zero), then there is no support in the data for these 
hypotheses. In these cases the corresponding entry in `results$BFmatrix` 
(the Bayes factor comparing both hypotheses) is very unstable and should
only be interpreted if repeated analyses using different seeds 
(see `set.seed()`) render about the same results.

#### Example output

The following example is aimed at illustrating the output from `bain`. The specifics on how to actually use `bain` for testing hypotheses about the parameters of different statistical models are given in the next chapter.


In this example we will use `bain` to test the following two (informative) hypotheses concerning the parameters of a multiple linear regression model, with three (continuous) predictors: 
 
 - $H_1: \beta_1 > 0;\; \beta_2 > 0;\; \beta_3 >0$ 
 - $H_2: \beta_1 = \beta_2 < \beta_3$ 
 


(1) The default output:

```r
print(results)
```

```
## Bayesian informative hypothesis testing for an object of class lm (continuous predictors):
## 
##    Fit   Com   BF.u   BF.c    PMPa  PMPb 
## H1 0.871 0.032 27.295 204.167 0.747 0.727
## H2 2.637 0.285 9.247  9.247   0.253 0.246
## Hu                                  0.027
## 
## Hypotheses:
##   H1: age>0&peabody>0&prenumb>0
##   H2: age=peabody<prenumb
## 
## Note: BF.u denotes the Bayes factor of the hypothesis at hand versus the unconstrained hypothesis Hu. BF.c denotes the Bayes factor of the hypothesis at hand versus its complement.
```
The output consists of several parts:
 
 (a) On the top, it is stated from which type of `R` object the parameter estimates come from;
 (b) The most important part of the output are the results from `bain` presented in the middle. Here, for each hypothesis, the fit, complexity, the BF of the hypothesis against the unconstrained, the BF of the hypothesis against it's compliment, and the posterior model probabilities are printed in each row. For example, for $H_1 =\beta_1 > 0;\; \beta_2 > 0;\; \beta_3 >0$, the fit and complexity are equal to 0.871 and 0.032, respectively, resulting in a $BF_{1u}$ = 27.3. The value for $BF_{1u}$ suggests that $H_1$ is 27 times as likely compared to $H_u$. The value for `PMPa` suggests that the posterior probability of $H_1$ *given* the data is around 0.75, assuming equal prior probabilities. The value for `PMPb` suggests that the posterior probability of $H_1 + H_u$  *given* the data is around 0.73, assuming equal prior probabilities. If we would like to obtain the BF for $H_1$ against $H_2$ ($BF_{12}$), then we just need to take the ratio of the two BF's against the unconstrained hypothesis. In this case, $$BF_{12} = \frac{27.3}{9.2} \approx 3$$
 
 which means that the support in the data is 3 times larger for $H_1$ compared to $H_2$;
 
 (c) The tested hypotheses are printed in the output, just below the results;
 (d) The output ends with a note explaining the difference between the two types BF's printed in the main part of the results (b).
 
(2) Descriptive matrix:

```r
summary(results)
```

```
##   Parameter   n   Estimate          lb        ub
## 1       age 240 0.05797594 -0.04043979 0.1563917
## 2   peabody 240 0.14781779  0.03457188 0.2610637
## 3   prenumb 240 0.56740695  0.46067277 0.6741411
```

(3) Obtaining the separate values contained within the `results` object. Here we only present how to obtain the value for the fit, of course, as mentioned above different values contained within the `results` object can be obtained by changing what comes after the `$` in the code below:  

```r
results$fit
```

```
##      Fit_eq    Com_eq    Fit_in     Com_in       Fit        Com         BF
## H1 1.000000 1.0000000 0.8705764 0.03189549 0.8705764 0.03189549 204.167441
## H2 2.636926 0.5703344 1.0000000 0.50000000 2.6369262 0.28516720   9.246948
## Hu       NA        NA        NA         NA        NA         NA         NA
##         PMPa       PMPb      BF.u       BF.c
## H1 0.7469474 0.72705089 27.294657 204.167441
## H2 0.2530526 0.24631200  9.246948   9.246948
## Hu        NA 0.02663711        NA         NA
```

