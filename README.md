# Code for pharmacodynamic analysis
<!-- writtern for the analysis of "Nasim Roshani Asl" thesis. -->

## NCA analysis
the unit for concentration in the original data was "mIU/mL"\
it was converted to "ng/mL" for this analysis.\
furthermore Bolus dose was 20,000 IU which is equivalent to 0.1 mg.\
for conversion of "mIU/mL" to "ng/mL" conversion factor of `0.001` was used.

analysis was mainly performed using `pkr` package in R.
- [Analysis code](/NCA%20analysis/pkpd%20analysis.md)
- [NCA results](/NCA%20analysis/result.csv)
- [NCA graphs](/NCA%20analysis/Output/)

## Population Pharmacodynamic modeling and simulation

[Modeling code](/population%20pk/pop%20pk%20modeling.md)\
[Model plots](/population%20pk/plot_history.pdf)


Sources:
- [Nonlinear Mixed-Effects Model Development and Simulation Using nlmixr and Related R Open-Source Packages](https://ascpt.onlinelibrary.wiley.com/doi/10.1002/psp4.12445)
- [Running PK models with nlmixr](https://nlmixrdevelopment.github.io/nlmixr/articles/running_nlmixr.html)
- [nlmixr: an R package for population PKPD modeling](https://nlmixrdevelopment.github.io/nlmixr/)

1. Clearance (CL)\
Definition: The volume of plasma cleared of the drug per unit time (L/hr).\
Role: Governs how quickly the drug is eliminated from the body.\
Adjustment:\
Increase CL: Faster drug elimination, shorter half-life.\
Decrease CL: Slower drug elimination, longer half-life.\
Estimation:CL=Dose/AUC (area under the concentration-time curve).

2. Volume of Distribution (V)\
Definition: The apparent volume into which the drug is distributed (L).\
Role: Determines the initial plasma concentration.\
Adjustment:\
Increase V: Lower peak plasma concentration.\
Decrease V: Higher peak plasma concentration.\
Estimation:V=Dose/C0  (initial concentration).

3. Elimination Rate Constant (k)\
Definition: The rate at which the drug is eliminated (hr−1).\
Role: Controls the exponential decline of the plasma concentration.\
Adjustment:\
Increase k: Faster elimination.\
Decrease k: Slower elimination.\
Relationship:k=CL/V.

4. Inter-Individual Variability (IIV)\
Definition: Accounts for differences in PK parameters between individuals.\
Role: Adds variability for parameters (V) using exponential models.\
Adjustment:\
Increase η: Greater variability between individuals.\
Decrease η: More homogeneous population.

5. Residual Unexplained Variability (RUV)\
Definition: Accounts for random error in observed concentrations.\
Role: Defines the error model (e.g., additive, proportional).\
Adjustment:\
Increase add.err: Less precision in observed data.\
Decrease add.err: Higher precision.