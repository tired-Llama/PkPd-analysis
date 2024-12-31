# population Pharmacokinetic modeling of Eprex

## load packages

``` r
library(openxlsx)
library(dplyr)
library(tidyr)
library(knitr)
library(ggplot2)
```
## load and prepare data

``` r
data  <- read.xlsx("NCA analysis/Kinetic_clean.xlsx")
```

```
## Error in read.xlsx.default("NCA analysis/Kinetic_clean.xlsx"): File does not exist.
```

``` r
df <- data  %>% mutate(across(seq(5,17,2),.fns = as.double))  %>% 
    mutate(gender_recoded = as.factor(gender))  %>%  
    mutate(c0 = level0, dose = 20000)  %>% 
    reshape(varying = c('level0','level1','level2','level3','level6','level8','level10'), 
        times = c(0,1,2,3,6,8,10), 
        v.names = "concentration", 
        idvar = 'ID',
        drop = c(seq(4,18,2)),
        direction = "long")  %>% 
    arrange(ID)  %>% 
    drop_na(concentration) %>% 
    mutate(TIME= time*24) %>% 
    mutate(DV = concentration*0.005) %>% 
    mutate(AMT = 0.1, EVID = if_else(time == 0,101,0)) %>% 
    select(ID,TIME,DV,AMT,EVID)
```

```
## Warning: There were 6 warnings in `mutate()`.
## The first warning was:
## ℹ In argument: `across(seq(5, 17, 2), .fns = as.double)`.
## Caused by warning:
## ! NAs introduced by coercion
## ℹ Run `dplyr::last_dplyr_warnings()` to see the 5 remaining warnings.
```

``` r
# View(df)
# glimpse(df)
```


``` r
# Define the PK model
# Load required libraries
library(nlmixr2)
# library(RxODE)
 
# Define the one-compartment IV bolus model
one_comp_iv_model <- function() {
  ini({
    # Fixed effects (population-level estimates)
    cl <- 1;        # Clearance (L/hr)
    v <- 10;        # Volume of distribution (L)
    # Inter-individual variability (IIV)
    eta.cl ~ 0.1;   # Variability in clearance
    eta.v ~ 0.1;    # Variability in volume
    # Residual unexplained variability (RUV)
    add.err <- 0.1; # Additive error model
  })
  model({
    # Define individual parameters with variability
    cl_i <- cl * exp(eta.cl); # Individual clearance
    v_i <- v * exp(eta.v);    # Individual volume
    # Initial condition for drug amount in the central compartment
    centr(0) <- 0.1; # Dose in mg
    # PK equations
    d/dt(centr) = -cl_i / v_i * centr; # Drug elimination
    # Plasma concentration
    cp = centr / v_i;
    # Residual variability model
    cp ~ add(add.err);
  })
}
```


``` r
# Fit the model using nlmixr2
fit <- nlmixr2(one_comp_iv_model, df, est = "saem")
```

```
##  
##  
##  
## 
```

```
## Error in parse(text = paste(.ret, collapse = "\n"), keep.source = FALSE): <text>:1:1: unexpected '<'
## 1: <
##     ^
```

``` r
# Summary of the fit
kable(summary(fit))
```



|   |      ID   |     TIME      |      DV       |     PRED         |     RES        |    IPRED        |     IRES        |    IWRES        |    eta.cl       |    eta.v        |      cp         |    centr         |     cl_i        |     v_i          |     tad    |   dosenum     |
|:--|:----------|:--------------|:--------------|:-----------------|:---------------|:----------------|:----------------|:----------------|:----------------|:----------------|:----------------|:-----------------|:----------------|:-----------------|:-----------|:--------------|
|   |5      : 6 |Min.   : 24.00 |Min.   :0.0055 |Min.   :0.000e+00 |Min.   :0.00550 |Min.   :0.000000 |Min.   :-0.85996 |Min.   :-3.33283 |Min.   :-11.9167 |Min.   :-7.22626 |Min.   :0.000000 |Min.   :0.0000000 |Min.   :0.000009 |Min.   : 0.006928 |Min.   : 24 |Min.   :0.0000 |
|   |9      : 6 |1st Qu.: 48.00 |1st Qu.:0.1000 |1st Qu.:0.000e+00 |1st Qu.:0.09999 |1st Qu.:0.000132 |1st Qu.: 0.01571 |1st Qu.: 0.06087 |1st Qu.: -7.4113 |1st Qu.:-5.10414 |1st Qu.:0.000132 |1st Qu.:0.0003619 |1st Qu.:0.000846 |1st Qu.: 0.058618 |1st Qu.: 48 |1st Qu.:1.0000 |
|   |1      : 5 |Median : 72.00 |Median :0.2015 |Median :5.356e-07 |Median :0.20150 |Median :0.007882 |Median : 0.10122 |Median : 0.39230 |Median : -3.7058 |Median :-0.81767 |Median :0.007882 |Median :0.0144682 |Median :0.034396 |Median : 4.205327 |Median : 72 |Median :1.0000 |
|   |2      : 5 |Mean   : 97.24 |Mean   :0.4773 |Mean   :1.211e-04 |Mean   :0.47717 |Mean   :0.392141 |Mean   : 0.08514 |Mean   : 0.32998 |Mean   : -4.1223 |Mean   :-2.44397 |Mean   :0.392141 |Mean   :0.0364527 |Mean   :0.615912 |Mean   : 5.225459 |Mean   : 96 |Mean   :0.8448 |
|   |3      : 5 |3rd Qu.:144.00 |3rd Qu.:0.4874 |3rd Qu.:1.819e-05 |3rd Qu.:0.48737 |3rd Qu.:0.438226 |3rd Qu.: 0.21258 |3rd Qu.: 0.82387 |3rd Qu.: -0.7731 |3rd Qu.: 0.06417 |3rd Qu.:0.438226 |3rd Qu.:0.0622193 |3rd Qu.:0.645909 |3rd Qu.:10.157320 |3rd Qu.:144 |3rd Qu.:1.0000 |
|   |8      : 5 |Max.   :240.00 |Max.   :3.2100 |Max.   :6.181e-04 |Max.   :3.20938 |Max.   :3.609965 |Max.   : 0.66700 |Max.   : 2.58499 |Max.   :  0.9583 |Max.   : 0.29786 |Max.   :3.609965 |Max.   :0.1643530 |Max.   :3.648452 |Max.   :12.831168 |Max.   :240 |Max.   :1.0000 |
|   |(Other):26 |NA             |NA             |NA                |NA              |NA               |NA               |NA               |NA               |NA               |NA               |NA                |NA               |NA                |NA's   :9   |NA             |


``` r
# Goodness-of-fit plots
# pdf("population pk/plot_history.pdf")
# plot(fit)
# dev.off()
```

``` r
knit("population pk/pop pk modeling.rmd","population pk/pop pk modeling.md")
```

```
## Warning in file(con, "r"): cannot open file 'population pk/pop pk
## modeling.rmd': No such file or directory
```

```
## Error in file(con, "r"): cannot open the connection
```
