---
layout: post
title: Bayesian von Bertalanffy growth model for the Spinetail Mobula
excerpt: "Using Stan"
date: "June 6, 2015"
modified: 2015-06-06
---

This is the workflow to fit a von Bertalanffy growth model on length-at-age data from the Spinetail Mobula, *Mobula japanica*. The raw data was kindly provided by Oscar Sosa-Nishizaki who co-authored the published age and growth study on this species:

Cuevas-Zimbrón, E., Sosa-Nishizaki, O., Pérez-Jiménez, J., & O’Sullivan, J. (2013). An analysis of the feasibility of using caudal vertebrae for ageing the spinetail devilray, *Mobula japanica* (Müller and Henle, 1841). *Environmental Biology of Fishes*, 96(8), 907–914. <http://dx.doi.org/10.1007/s10641-012-0086-2>


### R workflow



First we transform the data from a data frame to a list:

``` r
japanica <- read_csv("japanica.csv")
growth_dat <- with(japanica, list(DW = DW, age = age, N = length(DW)))
```



Specifying the model in `stan`:


``` r
model <- "
// von Bertalanffy growth model fitted using Rstan
// 
//

data {
  int<lower=1> N;     // rows of data
  int<lower=0> age[N];  // predictor
  int<lower=0> DW[N];  // response
}

parameters {
  real<lower=0> sigma; // error term
  real<lower=0> k;
  real<lower=0> Linf;
  real<lower=0> L0;
 }

model {
  real mu[N];

  for (i in 1:N) {
    mu[i] <- Linf * (1 - ((1 - (L0/Linf)) * exp(-k * age[i])));
    DW[i] ~ normal(mu[i], sigma);
  }

  // priors
  sigma ~ normal(0, 30);
  k ~ normal(0.2, 1);
  Linf ~ normal((3100 * 0.9), 100);
  L0 ~ normal(880, 200);
}
"
```

Note above that I have set particularly narrow priors for the values of both $L_{\infty}$ and $L_{0}$ based on maximum size information from FishBase. Given that the maximum reported size is 3100 mm DW, the prior for $L_{\infty}$ is set as a normal distribution with mean 0.9 * 3100 and a narrow standard deviation. Similarly, $L_{0}$ is set as a normal distribution of mean 880 and a st. dev. of 200, which covers the range of known sizes at birth (850-930 mm DW).

**DISCLAIMER: I think the Bayesian model and it's distributions are set up well, however I am not familiar with these analyses (yet) and there is a decent chance I have it all wrong.** 

Running the model:


```r
fit <- stan(model_code = model, data = growth_dat, 
            iter = 2000, chains = 4)
```


Visualizing the output:


```r
par(mfrow = c(1,1))
plot(fit)
```

![](/figure/unnamed-chunk-6-1.png) 

Posteriors look legit, all within reasonable ranges. Rhat values close to 1 indicate that the chains have converged.


```r
traceplot(fit, inc_warmup=T)
```

![plot of chunk unnamed-chunk-7](/figure/unnamed-chunk-7-1.png) 

Chains do look converged.


```r
e <- extract(fit, pars = c("k", "Linf", "L0", "sigma"))

iter <- 10000
priors <- list("k" =   rnorm(iter, 0.2, 1),
               "Linf" = rnorm(iter, (3100 * 0.9), 100),
               "L0" = rnorm(iter, 880, 200),
               "sigma" = rnorm(iter, 0, 30))

par(mfrow = c(2,2))
for (i in 1:4) {
  plot(density(e[[i]]), main = names(e)[i])
  lines(density(priors[[i]]), col = "red")
}
legend("topright", lty = 1, col = c("black", "red"), 
       legend = c("Posterior", "Prior"))
```

![plot of chunk unnamed-chunk-8](/figure/unnamed-chunk-8-1.png) 

It seems that both $L_{\infty}$ and $L_{0}$ are strongly influenced by their prior, but $k$ and sigma are not. Estimated $L_{\infty}$ is around 200 mm less than the mean of the prior. It is likely that the length-at-age data is too sparse to to "overcome" the information from the priors, and thus it will always be strongly influenced by them.

Table of posterior values:


```r
monitor(as.array(fit), digits_summary=2)
```

```
## Inference for the input samples (4 chains: each with iter=1000; warmup=500):
## 
##          mean se_mean     sd    2.5%     25%     50%     75%   97.5% n_eff
## sigma  151.82    0.41  10.96  130.52  144.37  151.67  158.79  174.92   721
## k        0.18    0.00   0.03    0.12    0.16    0.18    0.20    0.25   297
## Linf  2615.40    5.98 107.02 2419.08 2541.48 2608.42 2684.76 2836.70   321
## L0     980.43    5.90 109.02  766.73  907.56  982.42 1055.13 1176.83   342
## lp__  -313.01    0.07   1.55 -316.94 -313.73 -312.69 -311.89 -311.13   471
##       Rhat
## sigma 1.00
## k     1.02
## Linf  1.01
## L0    1.02
## lp__  1.00
## 
## For each parameter, n_eff is a crude measure of effective sample size,
## and Rhat is the potential scale reduction factor on split chains (at 
## convergence, Rhat=1).
```


Correlation between posterior values

```r
pairs(do.call(cbind, e))
```

![plot of chunk unnamed-chunk-10](/figure/unnamed-chunk-10-1.png) 

As an expected from model parameterization, $k$ and $L_{\infty}$ are negatively correlated, while $L_{\infty}$ and $L_{0}$ are somewhat positvely correlated. This means it is likely that the prior for $L_{\infty}$ is also influencing the posterior of $k$.


