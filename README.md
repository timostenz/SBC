# Simulation-based Calibration
## Efficient simulation-based calibration for Bayesian models
SBC provides tools to easily validate and offer corrections on prior, likelihood, and computation algorithms based on the self-recovering property of Bayesian models. This package contains tools such as SBC rank histograms, ECDF plots, and their summary statistics which can be used to assess computational faithfulness. Please refer to the included vignette for detailed usage.

## Varieties of calibrations: scope of this package
Calibration is a model development bootstrap and its method and target varies. Target is chosen as modeler's quantity of interest and directly affects the calibrated result as reward in reinforcement learning. Method depends on how much you marginalized or conditioned the full joint space to test coverage. Scope of this package is checked below.

## Target
- [x] a. Prediction, p(y.new): Predictive checks which compares the distribution of replicated y and real y.

- [x] b. Posterior: p(theta.new|theta = c): Prior and likelihood are tested assuming the consistency of computation. Any Bayesian model has the *self recovering property*, in which averaging over posterior distributions fitted with samples from the prior predictive distribution will always be equal to the prior distribution. SBC uses the above principle and evaluates the combination of the prior and likelihood model under a fixed computation algorithm. Users should choose one computation algorithm in advance, such as full HMC, ADVI, Laplace approximation. Properties of potential posterior distributions resulting from these calibration simulations allows us to identify common pathologies, such as overfitting and poor identifiability, that limit the utility of any resulting inferences. 

- [x] c. Posterior computations: This quantifies how faithfully our computational tools represent the model that they’re fitting. One usecase would be testing approximation algorithms. Approximation based Bayesian computation is very promising but one limitation is that it can be hard to diagnose its reliability. For example, full HMC benchmark is needed to measure its error. SBC which evaluates how well an algorithm samples from the posterior distribution, given a model and a prior could be an alternative tool for measuring reliability.

Note. calibration matters but so does precision.

## Method
Examples are for calibrating forecast probability and p.hat is E(y.new|y) i.e. posterior expectation.
- [x] a. Unconditional coverage: E(y.new) = E(p.hat) is both a Bayesian and frequentist property. For both modes of inference, unconditional coverage will occur if all the assumptions are true. 

- [x] b. Coverage conditional on data: E(y.new|p.hat) = p.hat for any p.hat is Bayesian. 

- [ ] c. Coverage conditional on theta: E(y.new|theta) = E(p.hat|theta) is frequentist.

See Andrew Gelman's [writing](https://statmodeling.stat.columbia.edu/2012/12/06/yes-checking-calibration-of-probability-forecasts-is-part-of-bayesian-statistics/) for further detail.

## Installation
```
devtools::install_github("hyunjimoon/SBC")
```
## Stan file manual
One stan file includes three blocks:

1. pri_sim: `sample_theta_tilde_stan`
simulate N parameter values from given prior distribution.
```{stan}
generated quantities {
  real beta_;
  real alpha_;
  beta_ = normal_rng(0, 10);
  alpha_ = normal_rng(0, 10);
  ...
}
```
2. data_sim: `sample_y_tilde`
simulate y from each simulated parameter values in 1.
```{stan}
generated quantities {
  ...
  vector[N] y_;
  for (n in 1:N){
    y_[n] = normal_rng(X[n] * beta + alpha, 1.2);
  }
  }
```
3. post_sim: `sample_theta_bar_y`
return parameter samples that best explain the simulated y in 2.
```{stan}
model {
  beta ~ normal(0,10);
  alpha ~ normal(0,10);
  y ~ normal(X * beta + alpha, 1.2);
}
```
Further examples are in [tests](https://github.com/hyunjimoon/SBC/tree/master/tests) folder.
---
### Currently supports:
* Rank Histogram
* ECDF plot
* Uniformity checks
* ~~Centered Histogram plot~~
* Support for automated predictive sampling of Stan models in canonical form yhat ~ P(y | theta)
* Multi-parameter SBC plots
---
### TODO:
* ECDF\_diff plot
* More envelope metrics
* Add verbose diagnostics, akin to stan's get\_hmc\_diagnostics
* Inferential Calibration
* Decision Calibration

### References:
Theoretical support
* [Validating Bayesian Inference
Algorithms with Simulation-Based
Calibration](https://arxiv.org/pdf/1804.06788.pdf) Talts, Betancourt, Simpson, Vehtari, Gelman, 2018
* [Rank-Normalization, Folding, and Localization: An Improved R-hat for Assessing Convergence of MCMC](https://arxiv.org/abs/1903.08008) Vehtari, Gelman, Simpson, Carpenter, Bürkner, 2021
* [Graphical Test for Discrete Uniformity and its Applications in Goodness of Fit Evaluation and Multiple Sample Comparison](https://arxiv.org/abs/2103.10522)  Säilynoja, Bürkner, Vehtari, 2021
* [Bayes factor workflow](https://arxiv.org/pdf/2103.08744.pdf) Schad, Nicenboim, Bürkner, Betancourt, Vasishth, 2021
* [ECDF with codes](https://avehtari.github.io/rhat_ess/rhat_ess.html)

Application support
* [Cognitive science, response time fitting](https://link.springer.com/content/pdf/10.3758/s13428-019-01318-x.pdf)
* [Bioinformatics, effect of mutation prediction](https://www.biorxiv.org/content/10.1101/2020.10.27.356758v1.full.pdf)
* [Earth science, earthquake prediction](https://gmd.copernicus.org/articles/11/4383/2018/gmd-11-4383-2018.pdf )

FAQ
> How does calibration relate to prediction accuracy?

Comparing the ground truth and the simulated result is a backbone of calibration and comparison target greatly affects the calibrated (i.e. trained) result, similar to reward in reinforcement learning. In this sense, if the U(a(y), theta) term is designed for prediction, the model will be calibrated to have best predictive result as possible.
