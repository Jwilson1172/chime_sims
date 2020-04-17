# Bayes CHIME Explainer

Many factors surrounding the transmission, severity of infections, and remaining susceptibility of local populations to COVID-19 remain highly uncertain. However, as new data on hospitalized cases becomes available, we wish to incorporate this data in order to update and refine our projections of future demand to better inform capacity planners. To that end we have extended CHIME to increase the epidemiological process realism and to coherently incorporate new data as it becomes available. This extension allows us to transition from a small number of scenarios to assess best and worst case projections based on parameter assumptions, to a probabilistic forecast representing a continuous distribution of likely scenarios.

### Discrete-time SEIR model

The first extension we have included is to explicitly model the incubation period between when an individual is exposed to when they become infectious. This model consists of individuals who are either *Susceptible* (SS), *Exposed* (EE), *Infected* (II), or *Recovered* (RR).

The epidemic proceeds via a growth and decline process. This is the core model of infectious disease spread and has been in use in epidemiology for many years. The dynamics are given by the following 4 equations.

St+1=(−β(St/N)It)+StSt+1=(−β(St/N)It)+St.Et+1=(β(St/N)It−γE)+EtEt+1=(β(St/N)It−γE)+Et.It+1=(αEt−γIt)+ItIt+1=(αEt−γIt)+It.Rt+1=(γIt)+RtRt+1=(γIt)+Rt

where

S+E+I+R=NS+E+I+R=N

From this model we then estimate hospital and vent census by estimating what proportion of each newly infected case will require hospitalization, and what proportion of those will require ventilation, along with how long each of these resources will be required per patient.

We can think of the collection of unknown parameters that we need to estimate in order to get a projection as the set θθ. For any given set of input parameters, we get a unique projection of daily admissions and census for each level of care required. To date, we have used our best estimates of what we believe to be the value of each of these parameters, then running a variety of scenarios against the bounds of plausible parameter estimates.

### Bayesian Extension

Now that we've started accumulating significant data, we can integrate this data into our model to inform the relative likelihood of various parameter combinations. From these newly data-informed distribution of parameters, we then produce a *distribution* of projections. The result is a forecast which provides probability distributions over likely future outcomes that are informed by a combination of what we've seen so far, and what we know about input parameters from other locations.

Formally, we're modeling the probability distribution of parameters θθ by incorporating what we believe about the parameters (P(θ)P(θ)) and what we have observed so far (the census of hospitalized COVID-19 patients to date, Ht<=nowHt<=now, and the census of ventilated COVID-19 patients to date, Vt<=0Vt<=0. Using Bayes theorem, we get:

P(θ|Ht<=0,Vt<=0)∝P(Ht<=0,Vt<=0|θ)P(θ)P(θ|Ht<=0,Vt<=0)∝P(Ht<=0,Vt<=0|θ)P(θ)

From which we can then project distributions of future outcomes Ht>0Ht>0, Vt>0Vt>0 by simulating the SIR forward in time using our newly data-informed distribution of parameters.

#### Regularization (experimental)

Our model has many parameters, and we could easily overfit our time series unless we provide some constraints on the flexibility of our chosen solution. To that end, we've implemented an empirical Bayesian regularization algorithm in which each prior's range is implicitly shrunken towards its median, to a degree chosen by goodness of fit to one-weeks worth of holdout data. This component of the modeling process is experimental and can be skipped by providing the default value of 0.05.

Unlike regularization algotithms that shrink parameters to zero (as is common in elastic net regression), this algorithm shrinks parameters to the (non-zero) median of their prior distribution. This provides tighter credible intervals around future forecasts while reducing the ability of the model to overfit the observed time series. Resultant forecasts have intervals that are smaller than the true range of our uncertainty, but which may be more useful for forecasting in the short term.

### Output interpretation

The outputs of BayesCHIME are *probability distributions* over future paths of the regional epidemic. These distributions can be used to ask a variety of questions, for example:

- What is the probability that more than xx ventilators will be needed?
- What is the probability that more than xx ventilators will be needed by some specific date?
- What is the maximum census we can expect with 90% confidence in 7 days?
- How likely is a recent flattening trend to continue?

### Limitations

While the approach we are taking with BayesCHIME aims to incorporate data as it arrives while accounting for remaining uncertainties regarding parameter values, it is still nevertheless based on a relatively simplified model of the disease dynamics and progression. A non-exhaustive list of the complexities which we are not accounting for include:

- **Contact Structure Between Individuals.** The SEIR model assumes homogeneous mixing of individuals, though we know that people do not interact randomly. Instead contact between individuals would be better represented as a network structure.
- **Social Distancing Dynamics.** We are modeling the effect of social distancing (and other non-pharmacological interventions such as mask wearing and hand washing) as a proportional effect on the transmission rate. Additionally, we are assuming that the reduction in transmissible contacts reaches some new equilibrium and continues at this level indefinitely. The long terms dynamics are much more likely to reflect some form of oscillation as contact levels increase, leading to new surges, leading to returns to reduced contact levels, etc.
- **Hospital Progression.** Our model treats the levels of acuity of COVID-19 patients as independent spans of time. There is no hospital progression component accounting for sequences such as: admission -> ICU -> vent -> general medicine floor -> discharge.
- **Natural Variation in RtRt.** Many factors go into the reproductive number which are not endogenous to the model. Seasonal variation in infectiousness is observed in a variety of viral outbreaks, and virus mutation over time can change the virulence, among other factors.