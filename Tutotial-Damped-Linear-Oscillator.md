Fit Damped Linear Oscillator model with OpenMx
================
\[Mar J.F Ollero\]
(<a href="mailto:marjfollero@gmail.com)-" class="uri">mailto:marjfollero@gmail.com)-</a>
Autonomous University of Madrid, Spain

This page contains an illustrative example on how to fit the **Damped
Linear Oscillator model** to a single individual’s negative affect
measures.

The data used in this tutorial is available for download
[here]('DLO_Tutorial_Individual.Rdata'). *R* and *Rstudio* are also
needed and can be downloaded from
[here](https://www.rstudio.com/products/rstudio/download/).

Once *R* and *Rstudio* are installed. *OpenMx* package is required, to
install and load the package:

``` r
install.packages ("OpenMx")
```

``` r
library ("OpenMx")
```

Now, to load the data and see the first lines of it:

``` r
load ('DLO_Tutorial_Individual.Rdata')
head (simdata)
```

    ##            y1     times id
    ## 5   3.1197310 0.2083333  1
    ## 6   0.9832255 0.2500000  1
    ## 32  1.2577379 1.3333333  1
    ## 36  0.6383892 1.5000000  1
    ## 50 -2.2059128 2.0833333  1
    ## 53 -1.9651488 2.2083333  1

The data set has 3 variables:

-   **y1** is the score in negative affect at one point in time.
-   **times** is the moment when the measurement occasion was taken.
    This data set contains two measurement occasions per day within the
    same 12 hours everyday. The individual was measured for 60 days.
-   **id**, is a variable that indicates to which subject the
    measurements correspond. This data belongs to one individual.

Here is a figure that depicts the scores of the individual under study
over time:  
![](test.tiff)

Now to estimate the Damped Linear Oscillator model, the following code
is used:

``` r
cdim <- list("y1", c('Position', 'Velocity'))
xdim <- 2
udim <- 1
ydim <- 1

amat <- mxMatrix('Full', xdim, xdim, c(F, T, F, T), c(0, -.7, 1, -.3), name='A', 
                 labels = c(NA, "k", NA, "c"), 
                 lbound = c (NA, -2, NA, -2), 
                 ubound= c (NA, 0, NA, 0))

bmat <- mxMatrix('Full', xdim, udim, c (F, F) , c (1,0), name='B')

cmat <- mxMatrix('Full', ydim, xdim, free=F, values=c(1, 0), name='C', dimnames=cdim)

dmat <- mxMatrix('Zero', udim, udim, name='D')

qmat <- mxMatrix('Diag', xdim, xdim, free=c(F,T), values=c(0,5), name='Q', lbound=1e-20, labels= c(NA, "q"))

rmat <- mxMatrix('Diag', ydim, ydim, free=T, values=0, name='R', labels= c ("r"), lbound=1e-20, ubound=1)

xmat <- mxMatrix('Full', xdim, 1, free=c(T,F), values=c(0, 1), name='x0', labels= c("ini", NA))

pmat <- mxMatrix('Diag', xdim, xdim, free=FALSE, values=1, name='P0')

umat <- mxMatrix('Zero', udim, 1, name='u')

tmat <- mxMatrix('Full', 1, 1, name='time', labels='data.times')

dlo <- mxModel("DampedLinearOscillator", 
               amat, bmat, cmat, dmat, qmat, rmat, xmat, pmat, umat, tmat,
               mxExpectationStateSpaceContinuousTime('A', 'B', 'C', 'D', 'Q', 'R', 'x0', 'P0', 'u', t='time'),
               mxFitFunctionML())

dlod <- mxModel(dlo, mxData(simdata, 'raw'))
dlo_out <- mxRun(dlod)
```

And to access the results:

``` r
summary(dlo_out)
```

    ## Summary of DampedLinearOscillator 
    ##  
    ## free parameters:
    ##   name matrix row col   Estimate  Std.Error A lbound ubound
    ## 1    k      A   2   1 -0.3462621 0.04224284       -2      0
    ## 2    c      A   2   2 -0.1411508 0.08523269       -2      0
    ## 3    q      Q   2   2  1.8917323 0.79512043    1e-20       
    ## 4    r      R   1   1  0.7343751 0.13179590    1e-20      1
    ## 5  ini     x0   1   1  1.9910705 1.18205801                
    ## 
    ## Model Statistics: 
    ##                |  Parameters  |  Degrees of Freedom  |  Fit (-2lnL units)
    ##        Model:              5                    115              415.4067
    ##    Saturated:              2                    118                    NA
    ## Independence:              2                    118                    NA
    ## Number of observations/statistics: 120/120
    ## 
    ## Information Criteria: 
    ##       |  df Penalty  |  Parameters Penalty  |  Sample-Size Adjusted
    ## AIC:       185.4067               425.4067                 425.9330
    ## BIC:      -135.1549               439.3441                 423.5365
    ## CFI: NA 
    ## TLI: 1   (also known as NNFI) 
    ## RMSEA:  0  [95% CI (NA, NA)]
    ## Prob(RMSEA <= 0.05): NA
    ## To get additional fit indices, see help(mxRefModels)
    ## timestamp: 2022-06-29 15:18:17 
    ## Wall clock time: 0.666621 secs 
    ## optimizer:  SLSQP 
    ## OpenMx version number: 2.19.8 
    ## Need help?  See help(mxSummary)
