# Code for pharmacodynamic analysis
<!-- writtern for the analysis of "Nasim Roshani Asl" thesis. -->

## NCA analysis
the unit for concentration in the original data was "mIU/mL"\
it was converted to "ng/mL" for this analysis.\
furthermore Bolus dose was 20,000 IU which is equivalent to 0.1 mg.
for conversion of "mIU/mL" to "ng/mL" conversion factor of `0.001` was used.

analysis was mainly performed using `pkr` package in R.
- [Analysis code](/NCA%20analysis/pkpd%20analysis.rmd)
- [NCA results](/NCA%20analysis/result.csv)
- [NCA graphs](/NCA%20analysis/Output/)

## Population Pharmacodynamic modeling and simulation
