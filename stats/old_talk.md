<!-- .slide: data-background="assets/isb/data-midnight.jpg" class="dark" -->

# Statistical Models for Microbiome Data

## A story in formulas, GIFs, and memes...

### Christian Diener, Gibbons Lab

<img src="assets/isb/logo.png" width="40%">

<br>
<div class="footer">
<a href="https://creativecommons.org/licenses/by-nc/4.0/"><i class="fa fa-bullhorn"></i>CC-BY-NC</a>
<a href="https://gibbons.isbscience.org/"><i class="fa fa-globe"></i>gibbons.isbscience.org</a>
<a href="https://github.com/gibbons-lab"><i class="fa fa-github"></i>gibbons-lab</a>
<a href="https://twitter.com/thaasophobia"><i class="fa fa-twitter"></i>@thaasophobia </a>
</div>

---

<img src="assets/math.gif" width="50%">

---

# The Jon Snow models

<img src="assets/know_nothing.gif" width="50%">

We know nothing about the data but want to do statistics anyway.

---

## Mann-Whitney U-test

<img src="assets/hare.jpg" width="30%"><br>
<img src="assets/race.png" width="80%">

Non-parametric and great power, *but* the statistic of interest is weird and becomes
even weirder in regression.

---

## Maybe go parametric?

<img src="assets/weird.jpg" width="30%">

---

## Log-normal models

Abundances are continuous and positive, so if we log-transform it makes them normal, maybe?

`$$
\log(a_i) \approx \mathcal{N}(\mu, \sigma^2) \\
\mu = \alpha + \boldsymbol{\beta}\mathbf{x}
$$`

For instance, for two comparison groups we would do:

`$$ \mu = \alpha + \beta\cdot\mathbf{1}_{\{\text{group=2}\}}  $$`

Intercept covers the reference groups, $\beta$ is the difference between groups.

---

## Transformations

The log-transform makes little difference to categorical models but *adds additional assumptions*
to regressions. Regression with those transformations is often called *generalized linear modeling*.

Abundance grows exponentially with the covariate. Is that biological?

In practice, any log-based transform works. This can be coupled with compositional analysis,
for instance by using a CLR-transform:

`$$CLR(a_i) := \log(a_i) - \overline{\log(a_i)}$$`

---

## Can we do better though?

<img src="assets/know_some.gif" width="30%">

Generative models tend to perform better.

---

## Let's abstract sequencing...

<img src="assets/16S.png" width="100%">

---

## Binomial regression

If we see sequencing as a random process trying to hit a particular DNA fragment(s) (or parts)
we get a binomial process.

`$$
c_i \approx \mathcal{B}(n, p)
$$`

$n$ is the number of reads.

We have $\mu = np$ and $\sigma^2 = np(1-p)$, so in general $\sigma^2 \leq \mu$.

In a regression we want to make $p$ dependent on our covariates, how do we do that?

---

## Transformations part deux

<img src="assets/logit.png" width="30%">

`$$
c_i \approx \mathcal{B}(n, p) \\
\log(\frac{p}{1-p}) = \alpha + \boldsymbol{\beta}\mathbf{x}
$$`

---

## Poisson regression

In practice n is huge and p is tiny. So a Poisson process models this well.

`$$
c_i \approx \mathcal{P}(\lambda) \\
\log(\lambda) = \alpha + \boldsymbol{\beta}\mathbf{x}
$$`

So we get back our exponential/log scale.

We have $\mu = \lambda$ and $\sigma^2 = \lambda$, so in general $\sigma^2 = \mu$.

This *inflexibility of the variance* becomes a problem.

---

## Overdispersion

Frequentist statistics is not great with dispersion/variance in count data.

<img src="assets/overdispersion.jpg" width="50%">

---

## Hierarchical models

<img src="assets/hier.jpg" with="30%">

What if the parameter in the model comes from a distribution itself?

---

## Poisson-Gamma distribution

<img src="assets/gamma.png" width="30%">

`$$
c_i \approx \mathcal{P}(\lambda) \\
\lambda \approx \Gamma(k, s)
$$`

Very flexible and awesome to simulate sequencing reads. Hard to fit and thus
not awesome for regression.

---

## Negative Binomial regression

`$$
c_i \approx \mathcal{NB}(\mu, \phi) \\
\log(\mu) = \alpha + \boldsymbol{\beta}\mathbf{x}
$$`

We have $\mu = \mu$ and $\sigma^2 = \mu + \phi\cdot\mu^2$, so in general $\sigma^2 \geq \mu$.

The parameter $s$ thus can fit overdispersion.

---

## Beta-Binomial regression

Same idea but use the Binomial distribution with a beta distribution for the $p$ parameter.

<img src="assets/beta.png" width="30%">

`$$
c_i \approx \mathcal{B}(n, p) \\
p \approx \text{Beta}(m, \phi) \\
\log(\frac{m}{m-1}) = \alpha + \boldsymbol{\beta}\mathbf{x}
$$`

We have $\mu = n\cdot m$ and $\sigma^2 = nm(1-m)\cdot(1 + (n-1)\phi)$, so also $\sigma^2 \geq \mu$.

Overdispersion is fitted by $\phi$.

---

> Turtles all the way down. - Susan Holmes

<img src="assets/turtles.gif" width="30%">

If we would also put distributions on the regression parameters we would go fully Bayesian
but this requires MCMC methods to fit (for instance using STAN).

---

# What breaks the models?

<div style="display: flex; justify-content: space-around; align-items: center;">

<img src="assets/bycicle.jpg" width="30%">

<div>

- library size
- compositionality
- wrong quantity tested
- methodological bias
- p-value correction, FDR control choice

</div>

</div>

---

<!-- .slide: data-background="var(--primary)" class="dark" -->

## Summary

| Method              | Transformation | Overdispersion | estimated quantity | Language  | Packages             |
|---------------------|----------------|----------------|--------------------|-----------|----------------------|
| Rank methods        | ranks          | no             | ordering/rank sum  | Python, R | scipy                |
| Log-Normal          | log            | yes            | log-fold change    | Python, R | statsmodels, LIMMA, ANCOM-BC |
| Binomial            | logit          | no             | odds-ratio         | Python, R | statsmodels          |
| Poisson             | log            | no             | log-fold change    | Python, R | statsmodels          |
| Negative Binomial   | log            | yes            | log-fold change    | Python, R | statsmodels, VGAM, DESeq2, edgeR |
| Beta-Binomial       | logit          | yes            | odds ratio         | R         | corncob, VGAM        |


---

<!-- .slide: data-background="var(--primary)" class="dark" -->

<img src="assets/match_much.gif" width="30%"><br><br>

### And we are done :clap:

# Thanks!

