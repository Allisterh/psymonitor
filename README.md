
<!-- README.md is generated from README.Rmd. Please edit that file -->

# psymonitor <img src="man/figures/logo.png" align="right" height=139/>

[![lifecycle](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![Travis build
status](https://travis-ci.org/itamarcaspi/psymonitor.svg?branch=master)](https://travis-ci.org/itamarcaspi/psymonitor)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/itamarcaspi/psymonitor?branch=master&svg=true)](https://ci.appveyor.com/project/itamarcaspi/psymonitor)

The *psymonitor* package provides an R implementation of the popular
real-time monitoring strategy proposed by Phillips, Shi and Yu
(2015a,b;PSY), along with a new bootstrap procedure designed to mitigate
the potential impact of heteroskedasticity and to effect family-wise
size control in recursive testing algorithms (Phillips and Shi,
forthcoming). This methodology has been shown effective for bubble and
crisis detection (PSY, 2015a,b; Phillips and Shi, 2017) and is now
widely used by academic researchers, central bank economists, and fiscal
regulators.

## Installation

You can install the **development** version from
[GitHub](https://github.com/itamarcaspi/psymonitor/)

``` r
# install.packages("devtools")
devtools::install_github("itamarcaspi/psymonitor")
```

## Example

For the illustration purposes we will use data on the credit risk in the
European sovereign sector, that is proxied by an index constructed as a
GDP weighted 10-year government bond yield of the GIIPS (Spain, Ireland,
Italy, Greece, and Portugal) countries, and comes with the ‘psymonitor’
package.

Let’s walk through some basics. First load packages and get some data.

``` r
library(psymonitor)

data(spread)
head(spread)
```

    ##         date     value
    ## 1 1997-06-01 1.2225196
    ## 2 1997-07-01 1.0259384
    ## 3 1997-08-01 1.0510309
    ## 4 1997-09-01 0.8448181
    ## 5 1997-10-01 0.7406163
    ## 6 1997-11-01 0.7833504

Next, estimate the psy test statistic using PSY() and its corresponding
bootstrap-based critical values using wmboot().

``` r
y        <- spread$value
obs      <- length(y)
swindow0 <- floor(obs * (0.01 + 1.8 / sqrt(obs)))
Tb       <- 24 + swindow0 - 1
dim      <- obs - swindow0 + 1

bsadf          <- PSY(y)
quantilesBsadf <- wmboot(y, Tb = Tb, nboot = 99, nCores = 2) #Note that the number of cores is arbitrarily set to 2.
```

Next, identify crisis periods, defined as periods where the test
statistic is above its corresponding critical value, using the locate()
function.

``` r
monitorDates <- spread$date[swindow0:obs]
quantile95   <- quantilesBsadf %*% matrix(1, nrow = 1, ncol = dim)
ind95        <- (bsadf > t(quantile95[2, ])) * 1
periods      <- locate(ind95, monitorDates)
```

Finally, print a table that holds the identified crisis periods with the
help of the disp() function.

``` r
crisisDates <- disp(periods, obs)
print(crisisDates)
```

``` 
   start        end
```

1 2008-03-01 2008-03-01 2 2008-09-01 2009-04-01 3 2010-05-01 2012-08-01

-----

## References

  - Phillips, P. C. B., & Shi, S.(2017). Detecting financial collapse
    and ballooning sovereign risk. Cowles Foundation Discussion Paper
    No. 2110.
  - Phillips, P. C. B., & Shi, S.(forthcoming). Real time monitoring of
    asset markets: Bubbles and crisis. In Hrishikesh D. Vinod and C.R.
    Rao (Eds.), *Handbook of Statistics Volume 41 - Econometrics Using
    R*.
  - Phillips, P. C. B., Shi, S., & Yu, J. (2015a). Testing for multiple
    bubbles: Historical episodes of exuberance and collapse in the S\&P
    500. *International Economic Review*, 56(4), 1034–1078.
  - Phillips, P. C. B., Shi, S., & Yu, J. (2015b). Testing for multiple
    bubbles: Limit Theory for Real-Time Detectors. *International
    Economic Review*, 56(4), 1079–1134.
