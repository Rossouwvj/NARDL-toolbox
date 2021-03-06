NARDL functions
================

-   [1 Overview of functions](#overview-of-functions)
-   [2 Examples of the functions](#examples-of-the-functions)
    -   [2.1 `estimateNARDL`](#estimatenardl)
    -   [2.2 `NARDL_auto_lag`](#nardl_auto_lag)
    -   [2.3 `mplier_base`](#mplier_base)
-   [3 References](#references)

# 1 Overview of functions

The current toolbox consists of three functions.

-   `estimateNARDL`
-   `NARDL_auto_lag`
-   `mplier_base`

The `estimateNARDL` function is essentially a string builder that
creates a formula with a specific lag structure imposed. Using this
function it is much faster to get results when there are a large number
of variables. The function depends on the `dynlm` and `stringr`
packages.

The `NARDL_auto_lag` function performs automatic lag selection for all
the parameters in the NARDL model based on AIC or BIC. It depends on the
`estimateNARDL` function.

The `mplier_base` function creates the multipliers to plot.

# 2 Examples of the functions

Load in packages and functions

``` r
if (!require('pacman')) install.packages('pacman'); library('pacman')
```

    ## Loading required package: pacman

``` r
p_load(dynlm, stringr, zoo)
source("./estimateNARDL.R")
source("./NARDL_auto_lag.R")
source("./mplier_base.R")
```

## 2.1 `estimateNARDL`

The function will estimate an NARDL that is of the following form:

![ \\Delta y\_t  = c + \\rho y\_{t-1} + \\mathbf{\\theta^{+'}x^{+}\_{t-1}} + \\mathbf{\\theta^{-'}x^{-}\_{t-1}} + \\sum\_{j=1}^{p-1}\\gamma\_j \\Delta y\_{t-j} + \\sum\_{j=0}^{q-1}(\\mathbf{\\varphi\_j^{+'} \\Delta x^{+}\_{t-j} + \\varphi\_j^{-'} \\Delta x^{-}\_{t-j}}) + \\varepsilon\_t](https://latex.codecogs.com/png.latex?%20%5CDelta%20y_t%20%20%3D%20c%20%2B%20%5Crho%20y_%7Bt-1%7D%20%2B%20%5Cmathbf%7B%5Ctheta%5E%7B%2B%27%7Dx%5E%7B%2B%7D_%7Bt-1%7D%7D%20%2B%20%5Cmathbf%7B%5Ctheta%5E%7B-%27%7Dx%5E%7B-%7D_%7Bt-1%7D%7D%20%2B%20%5Csum_%7Bj%3D1%7D%5E%7Bp-1%7D%5Cgamma_j%20%5CDelta%20y_%7Bt-j%7D%20%2B%20%5Csum_%7Bj%3D0%7D%5E%7Bq-1%7D%28%5Cmathbf%7B%5Cvarphi_j%5E%7B%2B%27%7D%20%5CDelta%20x%5E%7B%2B%7D_%7Bt-j%7D%20%2B%20%5Cvarphi_j%5E%7B-%27%7D%20%5CDelta%20x%5E%7B-%7D_%7Bt-j%7D%7D%29%20%2B%20%5Cvarepsilon_t " \Delta y_t  = c + \rho y_{t-1} + \mathbf{\theta^{+'}x^{+}_{t-1}} + \mathbf{\theta^{-'}x^{-}_{t-1}} + \sum_{j=1}^{p-1}\gamma_j \Delta y_{t-j} + \sum_{j=0}^{q-1}(\mathbf{\varphi_j^{+'} \Delta x^{+}_{t-j} + \varphi_j^{-'} \Delta x^{-}_{t-j}}) + \varepsilon_t")

It also supports the addition of independent variables that are not
decomposed. For example:

![ \\Delta y\_t  = c + \\rho y\_{t-1} + \\mathbf{\\theta^{+'}x^{+}\_{t-1}} + \\mathbf{\\theta^{-'}x^{-}\_{t-1}} + \\sum\_{j=1}^{p-1}\\gamma\_j \\Delta y\_{t-j} + \\sum\_{j=0}^{q-1}(\\mathbf{\\varphi\_j^{+'} \\Delta x^{+}\_{t-j} + \\varphi\_j^{-'} \\Delta x^{-}\_{t-j}}) + \\mathbf{\\beta z\_{t-1}} + \\sum\_{j=1}^{r-1} \\mathbf{\\phi\_j \\Delta z\_{t-j}} +\\varepsilon\_t](https://latex.codecogs.com/png.latex?%20%5CDelta%20y_t%20%20%3D%20c%20%2B%20%5Crho%20y_%7Bt-1%7D%20%2B%20%5Cmathbf%7B%5Ctheta%5E%7B%2B%27%7Dx%5E%7B%2B%7D_%7Bt-1%7D%7D%20%2B%20%5Cmathbf%7B%5Ctheta%5E%7B-%27%7Dx%5E%7B-%7D_%7Bt-1%7D%7D%20%2B%20%5Csum_%7Bj%3D1%7D%5E%7Bp-1%7D%5Cgamma_j%20%5CDelta%20y_%7Bt-j%7D%20%2B%20%5Csum_%7Bj%3D0%7D%5E%7Bq-1%7D%28%5Cmathbf%7B%5Cvarphi_j%5E%7B%2B%27%7D%20%5CDelta%20x%5E%7B%2B%7D_%7Bt-j%7D%20%2B%20%5Cvarphi_j%5E%7B-%27%7D%20%5CDelta%20x%5E%7B-%7D_%7Bt-j%7D%7D%29%20%2B%20%5Cmathbf%7B%5Cbeta%20z_%7Bt-1%7D%7D%20%2B%20%5Csum_%7Bj%3D1%7D%5E%7Br-1%7D%20%5Cmathbf%7B%5Cphi_j%20%5CDelta%20z_%7Bt-j%7D%7D%20%2B%5Cvarepsilon_t " \Delta y_t  = c + \rho y_{t-1} + \mathbf{\theta^{+'}x^{+}_{t-1}} + \mathbf{\theta^{-'}x^{-}_{t-1}} + \sum_{j=1}^{p-1}\gamma_j \Delta y_{t-j} + \sum_{j=0}^{q-1}(\mathbf{\varphi_j^{+'} \Delta x^{+}_{t-j} + \varphi_j^{-'} \Delta x^{-}_{t-j}}) + \mathbf{\beta z_{t-1}} + \sum_{j=1}^{r-1} \mathbf{\phi_j \Delta z_{t-j}} +\varepsilon_t")

The function can take any number of independent variables and any number
of decomposed terms and requires the following arguments:

-   intercept: A Boolean value that controls if an intercept is included
    or not. Defualt is set to `TRUE`
-   dep: A `zoo` object that contains the dependent variable.
-   indep: A `zoo` object that contains all the independent variables.
-   decomp: A string object that contains the column names of the
    independent variables for which the decompositions will be done.
-   p: A vector of lags that will be included for the dependent variable
    (in levels)
-   qp: A vector of lags of the independent variables that forms the
    positive decompositions (in levels)
-   qn: A vector of lags of the independent variables that forms the
    negative decompositions (in levels)
-   r\_mat: A matrix of lags of the independent variables that are not
    decomposed (in levels). Each row represents a variable while each
    column is the lag to be included. If all the independent variables
    are decomposed then this field can be left blank.

To illustrate the use of this function we replicated the results from
Shin, Yu, and Greenwood-Nimmo (2014). The paper and associated data is
available [here](http://www.greenwoodeconomics.com/publications.html).
Loading in the data:

``` r
 USdata <- readRDS("./USdata.rds")
```

To replicate the results from Shin, Yu, and Greenwood-Nimmo (2014) we
use `estimateNARDL` with the following arguments:

``` r
# Convert data to zoo object
  USdata_zoo <- as.zoo(USdata[,-1], order.by = USdata$date)  

# Set function parameters
  dep <- USdata_zoo[,"UN", drop=FALSE]
  indep <- USdata_zoo[,c("IP"), drop=FALSE]
  decomp <- "IP"
  
# Estimate NARDL
  nardl <-   estimate_nardl(dep = dep, indep = indep, decomp = decomp, 
                   p=c(1,11), qn=c(0,4), qp=c(0,2))
  summary(nardl)
```

    ## 
    ## Time series regression with "zoo" data:
    ## Start = 1983-02-01, End = 2003-11-01
    ## 
    ## Call:
    ## dynlm(formula = f_combined, data = data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.37102 -0.07813 -0.00349  0.07612  0.39243 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   0.38048    0.09978   3.813 0.000174 ***
    ## L(UN, 1)     -0.05794    0.01373  -4.220 3.46e-05 ***
    ## L(IPp, 1)    -0.56553    0.16898  -3.347 0.000949 ***
    ## L(IPn, 1)    -1.67009    0.49555  -3.370 0.000875 ***
    ## L(d(UN), 1)  -0.18785    0.05617  -3.344 0.000958 ***
    ## L(d(UN), 11)  0.10698    0.05380   1.988 0.047907 *  
    ## L(d(IPp), 0) -8.31065    2.21901  -3.745 0.000226 ***
    ## L(d(IPp), 2) -4.71267    1.98254  -2.377 0.018235 *  
    ## L(d(IPn), 0) -8.37419    4.27017  -1.961 0.051025 .  
    ## L(d(IPn), 4) -9.17566    3.61589  -2.538 0.011796 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1291 on 240 degrees of freedom
    ## Multiple R-squared:  0.3206, Adjusted R-squared:  0.2951 
    ## F-statistic: 12.58 on 9 and 240 DF,  p-value: 2.478e-16

## 2.2 `NARDL_auto_lag`

The function requires the following arguments:

-   dep: A `zoo` object that contains the dependent variable.
-   indep: A `zoo` object that contains all the independent variables.
-   decomp: A string object that contains the column names of the
    independent variables for which the decompositions will be done.
-   p\_max: The maximum number of lags that will be included for the
    dependent variable (in levels)
-   q\_max: The maximum number of lags of the independent variables that
    are decomposed (in levels)
-   r\_max: The maximum number of lags of the independent variables that
    are not decomposed (in levels). If there are no independent
    variables that are not decomposed r\_max can be ignored.
-   selection: The selection criteria to be used in text. Supports ???AIC???
    or ???BIC.???

Example of the function:

``` r
   NARDL_auto <- NARDL_auto_lag(dep = dep, indep = indep, decomp = decomp, p_max = 5,q_max = 5)
```

The function will return a list that contains two objects. The first
object is the model with lag structure that minimizes the specified
criteria. The second object is a matrix or 3-D array that contains the
information criteria for various lag lengths. The estimated model can be
viewed by:

``` r
 summary(NARDL_auto[[1]])
```

    ## 
    ## Time series regression with "zoo" data:
    ## Start = 1982-08-01, End = 2003-11-01
    ## 
    ## Call:
    ## dynlm(formula = f_combined, data = data)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.41468 -0.08429  0.00180  0.07031  0.41904 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   0.22287    0.09350   2.384 0.017920 *  
    ## L(UN, 1)     -0.03127    0.01258  -2.486 0.013610 *  
    ## L(IPp, 1)    -0.33537    0.16567  -2.024 0.044035 *  
    ## L(IPn, 1)    -1.01566    0.48919  -2.076 0.038934 *  
    ## L(d(UN), 1)  -0.23087    0.06265  -3.685 0.000282 ***
    ## L(d(UN), 2)  -0.09406    0.06306  -1.491 0.137144    
    ## L(d(UN), 3)   0.08487    0.05779   1.469 0.143252    
    ## L(d(UN), 4)   0.08376    0.05635   1.486 0.138502    
    ## L(d(UN), 5)   0.11310    0.05658   1.999 0.046723 *  
    ## L(d(IPp), 0) -9.78831    2.20680  -4.436  1.4e-05 ***
    ## L(d(IPp), 1) -1.55087    2.29683  -0.675 0.500182    
    ## L(d(IPp), 2) -5.27990    2.30155  -2.294 0.022648 *  
    ## L(d(IPn), 0) -8.90549    4.45948  -1.997 0.046952 *  
    ## L(d(IPn), 1) -7.22381    4.44541  -1.625 0.105469    
    ## L(d(IPn), 2) -3.75604    4.43773  -0.846 0.398176    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.1337 on 241 degrees of freedom
    ## Multiple R-squared:  0.3279, Adjusted R-squared:  0.2888 
    ## F-statistic: 8.397 on 14 and 241 DF,  p-value: 1.124e-14

To obtain the 3-D array that contains the information criteria for
various lags:

``` r
 NARDL_auto[[2]]
```

    ##      [,1]      [,2]      [,3]      [,4]      [,5]
    ## [1,]    0    0.0000    0.0000    0.0000    0.0000
    ## [2,]    0 -274.7839 -284.1245 -279.1638 -280.7443
    ## [3,]    0 -277.4917 -283.6188 -280.4627 -281.2123
    ## [4,]    0 -278.3930 -284.1579 -280.8216 -279.8421
    ## [5,]    0 -282.8386 -287.1431 -283.8180 -281.0295

## 2.3 `mplier_base`

This function requires the following parameters

-   model: That is an lm object created by `estimateNARDL`,
    `NARDL_auto_lag`, or the `dynlm` package.
-   dep: A `zoo` object that contains the dependent variable.
-   indep: A `zoo` object that contains all the independent variables.
-   decomp: A string object that contains the column names of the
    independent variables for which the decompositions will be done.
-   l: The maximum number of lags specified in `NARDL_auto_lag` or in
    `nardl_GETS`
-   k: The number of independent variables. Note that
    ![x^{+}](https://latex.codecogs.com/png.latex?x%5E%7B%2B%7D "x^{+}")
    and
    ![x^{-}](https://latex.codecogs.com/png.latex?x%5E%7B-%7D "x^{-}")
    are treated as separate variables.
-   h: The horizon over which multipliers will be computed

**NB**: l needs to be the maximum number of lags of
![p](https://latex.codecogs.com/png.latex?p "p"),![q](https://latex.codecogs.com/png.latex?q "q")
or ![r](https://latex.codecogs.com/png.latex?r "r").

Using the `mplier` function:

``` r
 mplier <-  mplier_base(nardl, dep = dep, decomp = decomp, k=2,l=12,h=80)    
```

For the plot we append the `mplier` matrix with a zeros at the start so
that the line on the graphic starts from zero on the y-and x-axis.

``` r
 mplier <- rbind(c(0,0),
                 mplier)
   
  plot(mplier[,1],ylim = c(-20,30), type = "l", ylab = "dynamic multiplier", xlab = "h")  
  lines(-mplier[,2])
```

![](NARDL_functions_examples_files/figure-gfm/plot1-1.png)<!-- -->

# 3 References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-shin2014modelling" class="csl-entry">

Shin, Yongcheol, Byungchul Yu, and Matthew Greenwood-Nimmo. 2014.
???Modelling Asymmetric Cointegration and Dynamic Multipliers in a
Nonlinear ARDL Framework.??? In *Festschrift in Honor of Peter Schmidt*,
281???314. Springer.

</div>

</div>
