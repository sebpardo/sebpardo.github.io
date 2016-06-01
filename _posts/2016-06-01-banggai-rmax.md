---
layout: post
title: Estimating productivity for the Banggai cardinalfish
date: "June 1, 2016"
excerpt: "Estimating the maximum intrinsic rate of population increase under different life history scenarios"
modified: 2016-06-01
---

The Banggai cardinalfish (*Pterapogon kauderni*) is an unusual teleost
in that it lacks a planktonic larval phase as males mouth brood and care
for the eggs and larvae. These traits result in that a significant
portion of eggs survive to maturity, making this species more akin to
mammals and chondrichthyans in terms of survival and reproduction, than
to the majority of other teleosts. We obtain an estimate of productivity
for this species using the approach developed for other relatively low
fecundity species (chondrichthyans), by estimating the maximum intrinsic
rate of population increase *r*<sub>*max*</sub>. This is a
simplified model based on the Euler-Lotka equation (Myers et al. 1997,
Pardo et al. in press):

*l*<sub>*α*<sub>*mat*</sub></sub>*b* = *e*<sup>*r*<sub>*max*</sub> *α*<sub>*mat*</sub></sup> − *e*<sup>−*M*</sup>(*e*<sup>*r*<sub>*max*</sub></sup>)<sup>*α*<sub>*mat*</sub> − 1</sup>

where *l*<sub>*α*<sub>*mat*</sub></sub> is the proportion of each
clutch that survives to maturity, *b* is the annual reproductive output,
*α*<sub>*mat*</sub> is age at maturity, and *M* is the instantaneous
natural mortality rate. As the name indicates, the maximum intrinsic
rate of population increase is the **maximum** possible population
growth rate in the **absence of density dependence**, and is equivalent
to the fishing mortality that will drive a species to extinction
*F*<sub>*e**x**t*</sub> (Dulvy et al. 2004).

![Banggai cardinalfish](/figure/913px-Banggai_cardinal_fish.jpg)<br>
[Banggai cardinalfish](http://www.flickr.com/photos/jonhanson/207565670/in/set-72157594219248063/)
/ [John Hanson](http://www.flickr.com/people/61952179@N00) / [CC BY-SA
2.0](https://creativecommons.org/licenses/by-sa/2.0/deed.en)

------------------------------------------------------------------------

As the equation above shows, in order to estimate
*r*<sub>*max*</sub> for we need information on age at maturity,
natural mortality, annual reproductive output, and survival to maturity.
There is some degree of uncertainty in the life history of this species,
hence one can estimate *r*<sub>*max*</sub> a number of different
ways using different life history estimates, as well as tracking males
or females. Here, we calculate a range of plausible estimates of
*r*<sub>*max*</sub> under a range different life history scenarios,
which will be progressively more conservative (i.e., that we would
expect a lower productivity).

### Life histories

We know that the Banggai cardinalfish takes between 8 and 9 months to
reach maturity (Vagelli 1999). As a first approximation, let's assume
*α*<sub>*mat*</sub> is 8.5 months, which in years equals 0.71 years:

    amat <- 8.5/12
    amat

    ## [1] 0.7083333

It has been suggested that 5% of eggs reach adulthood, from which we
could assume a *l*<sub>*α*<sub>*mat*</sub></sub> of 0.05 (note:
Conant 2014 cited CITES 2007 as the source of this estimate, but I
couldn't locate it in the CITES proposal).

    lamat <- 0.05

Vagelli (2011) recorded clutch sizes of approximately 60 eggs per clutch
(Wikipedia says an average of 41), with males brooding around 400-650
eggs in a lifetime. This is around 9-10 clutches in their lifetime, so
we can assume an interbreeding interval of 4 months, or 0.33 years.
Because we only have information on number of clutches per year from
males, we will be estimating productivity in terms of males only.

### Scenario 1

To estimate the annual rate at which males can brood clutches we need to
know the interbreeding interval (*i*, in years), clutch size (*c*, in
numbers), and sex ratio. Assuming an even sex ratio (CITES 2007), we can
estimate annual reproductive output:

    i <- 4/12
    c <- 60
    sexratio <- 0.5 

    b <- (c * sexratio)/ i
    b

    ## [1] 90

Thus, 90 males (or females as we assume equal sex ratio) are produced
per year, of which 4.5 reach adulthood (90 \* 0.05).

We assume that natural mortality *M* as the reciprocal of average
lifespan (Pardo et al. in press). Given that maximum age is 4 years and
age at maturity is 0.7 years, we can estimate *M* as:

    amax <- 4
    M <- 1 / ((amat + amax) / 2)
    M

    ## [1] 0.4247788

Now that we have values for all these parameters, we can estimate
*r*<sub>*max*</sub> for this species:

    lalpha_b <- b * lamat  

    rmax_func <- function(rmax) (exp(rmax)^amat - exp(-M) * 
                                   (exp(rmax)^(amat - 1)) - (lalpha_b))^2

    rmax1 <- nlminb(0.1, rmax_func, lower = 0, upper = 5)$par
    rmax1

    ## [1] 2.226688

Therefore, for Scenario 1, our estimated *r*<sub>*max*</sub> is 2.23
year<sup>-1</sup>.

### Scenario 2: Assuming a lower estimate of clutch size

What would *r*<sub>*max*</sub> be if we considered the smaller
clutch size reported in Wikipedia (41 vs. 60 eggs)?

    c <- 41
    b <- (c * 0.5)/ i
    lalpha_b <- b * lamat  

    rmax2 <- nlminb(0.1, rmax_func, lower = 0, upper = 5)$par
    rmax2

    ## [1] 1.755198

With the lower clutch size value used in Scenario 2, our
*r*<sub>*max*</sub> estimate is 1.76 year<sup>-1</sup>, which is moderately lower
than in Scenario 1, but still relatively high.

### Scenario 3: Assuming lower fecundity, highest natural mortality, and highest age at maturity (most conservative scenario)

Our indirect estimate of natural mortality (0.425 year<sup>-1</sup>) might be too
low, so we'll raise it to a high but plausible value, to be *M* = 2.2
year<sup>-1</sup>, based on length frequency data from a recent study (Ndobe et al.
2013). We'll also change age at maturity to be at the highest value
reported in the literature (12 months, Ndobe et al. 2013). This should
result in a lower *r*<sub>*max*</sub> estimate than both previous
scenarios:

    M <- 2.2
    amat <- 12/12
    rmax3 <- nlminb(0.1, rmax_func, lower = 0, upper = 5)$par
    rmax3

    ## [1] 1.158704

The estimated *r*<sub>*max*</sub> for this scenario is 1.16 year<sup>-1</sup>,
which is still high.

### Notes on rate-limiting sex

We don't know whether males or females are the rate-limiting sex; that
is whether reproductive output is limited by the number or males or
females (the sex ratio), or the time it takes females to be ready to
produce and pass on eggs to a male of choice or the time it takes males
to brood and care for a single clutch. Right now, all the information we
have is on the time it takes a male to raise a clutch, and thus have
modelled the productivity of males. However, further information on the
sex ratio, and the reproductive frequency of females would allows us to
model females as well, and contrast their productivity with that of
males.

------------------------------------------------------------------------

### Conclusion

Our preliminary estimation of *r*<sub>*max*</sub> for multiple
scenarios suggests that the Banggai Cardinalfish has a high
productivity. Not surprisingly, the lower end of *r*<sub>*max*</sub>
values is for the Banggai cardinalfish is higher than the majority of
elasmobranchs (Pardo et al. in press), and is also higher than the
majority of teleosts examined by Hutchings et al. (2012) (although
annual reproductive output was estimated differently in the Hutchings
paper). We calculated values ranging from 1.16 to 2.23 year<sup>-1</sup>, which
indicate that this species could, at least, potentially **triple** its
population every year at very low population sizes and in the absense of
density dependence (remember that the finite rate of increase
*λ* = *e*<sup>*r*</sup>):

    lambda <- exp(rmax3)
    lambda

    ## [1] 3.185803

### References

CITES (2007). Convention on International Trade in Endangered Species of
Wild Fauna and Flora: consideration of proposals for amendment of
Appendices I and II. Proposal 19. Fourteenth Meeting of the Conference
of the Parties. June 3-15, 2007. 12 pages.

Conant, T.A. (2014). Endangered Species Act Draft Status Review Report:
Banggai Cardinalfish, *Pterapogon kauderni*. 40 pages.

Dulvy, N.K., Ellis, J.R., Goodwin, N.B., Grant, A., Reynolds, J.D. &
Jennings, S. (2004). Methods of assessing extinction risk in marine
fishes. *Fish Fish.*, 5, 255–276.

Hutchings, J.A., Myers, R.A., García, V.B., Lucifora, L.O. & Kuparinen,
A. (2012). Life-history correlates of extinction risk and recovery
potential. *Ecol. Appl.*, 22, 1061–1067.

Myers, R.A., Mertz, G. & Fowlow, P.S. (1997). Maximum population growth
rates and recovery times for Atlantic cod, *Gadus morhua*. *Fish.
Bull.*, 95, 762–772.

Ndobe, S., Soemarno, Herawati, E.Y., Setyohadi, D., Moore, A.,
Palomares, M.L.D., et al. (2013). Life history of Banggai cardinalfish,
*Pterapogon kauderni* (Actinopterygii: Perciformes: Apogonidae), in
Banggai Islands and Palu Bay, Sulawesi, Indonesia. *Acta Ichthyol.
Piscat.*, 43, 237–250.

Pardo, S.A., Kindsvater, H.K., Reynolds, J.D. & Dulvy, N.K.. Maximum
intrinsic rate of population increase in sharks, rays, and chimaeras:
the importance of survival to maturity. *Can. J. Fish. Aquat. Sci.*, in
press. Pre-print available at: <http://dx.doi.org/10.1101/051482>.

<!--Tweddle, D. & Turner, J.L. (1977). Age, growth and natural mortality rates of some cichlid fishes of Lake Malawi. *J. Fish Biol.*, 10, 385–398. -->
Vagelli, A. (1999). The Reproductive Biology and Early Ontogeny of the
Mouthbrooding Banggai Cardinalfish, *Pterapogon Kauderni* (Perciformes,
Apogonidae). *Environ. Biol. Fishes*, 56, 79–92.

Vagelli, A.A. (2011). The Banggai cardinalfish: natural history,
conservation, and culture of *Pterapogon kauderni*. John Wiley & Sons.

<!--
Hrmmmm.... To be continued...

We first load function that estimates $r_{max}$:


```r
source("rmax_single_function.R")
```

This loads the custom made function `rmaxnls`.


```r
params <- cbind(amat, amax, lamat, b)

rmax_val <- apply(params, 1, rmaxnlm, method = "new", mortality = "lifespan", lamat = TRUE)
rmax_val
```

```
## [1] 1.320507
```

$r_{max}$ = 1.32 year\textsuperscript{-1}?!?!?!? Double-checking by calculating it without helper function:

--->
