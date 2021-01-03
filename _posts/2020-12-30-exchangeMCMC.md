---
layout: distill_no_author_1
output:
  distill::distill_article:
    toc: true
    toc_depth: 2
title: Sampling from doubly intractable distributions - the ExchangeMCMC algorithm 
date: 29-12-2020
date_published: 0
description: A quick review

authors:  
  - name: Lorenzo Pacchiardi
    url: ""
    affiliations:
      name: Oxford

bibliography: 2020-12-30-exchangeMCMC.bib

---


# Motivation

In Bayesian Statistics, the main object of interest is the posterior distribution of the parameters $$ \theta $$ 
given some observed data $$ x $$. Specifically, given a prior distribution $$ \pi(\theta) $$ and a likelihood 
$$ p(x\vert \theta) $$, the posterior is (via Bayes' theorem): 

$$
\pi(\theta\vert x) = \frac{\pi(\theta) p(x\vert \theta)}{p(x)}, \qquad p(x) = \int \pi(\theta) p(x\vert \theta) d\theta.
$$

Markov Chain Monte Carlo (MCMC) techniques are widely used to sample from $$ \pi(\theta\vert x) $$; these usually require access to $$ \pi(\theta) $$ and $$ p (x\vert\theta) $$ but do not require evaluating $$ p(x) $$ (we show this below for the Metropolis-Hastings algorithm). 

However, there are cases in which the likelihood function is known only up to a normalizing constant: 

$$
p(x\vert \theta) = \frac {\tilde{p}(x\vert \theta)}{Z(\theta)},
$$
where $$ \tilde p(x\vert \theta) $$ is an unnormalized version of the likelihood and $$ Z(\theta) $$ is intractable. The posterior in this case is said __doubly intractable__ as there are two sources of intractability: $$ p(x) $$ and $$ Z(\theta) $$. 

In this case, MCMC cannot be straightforwardly used. Below, we describe a popular extension of Metropolis-Hastings that 
works in the case of doubly intractable distributions; but first, let us recall the original Metropolis-Hastings algorithm.  



# Metropolis-Hastings algorithm

The standard Metropolis-Hastings algorithm uses a distribution $$ q ( \cdot \vert \theta) $$ to generate a proposal 
value $$ \theta' $$ starting from the current state of the chain $$ \theta $$; then, this proposal is accepted with probability $$ \min\{1, \alpha \}$$, where:

\begin{equation}
\alpha = \frac{q(\theta\vert \theta') \pi(\theta') p(x\vert \theta')}{q(\theta'\vert \theta) \pi(\theta) p(x\vert \theta)} = \frac{q(\theta\vert \theta') \pi(\theta') \tilde {p}(x\vert \theta') }{q(\theta'\vert \theta) \pi(\theta) \tilde {p}(x\vert \theta)}  \textcolor{red}{\frac{Z(\theta)}{Z(\theta')}}.
\end{equation}

As you can see, this does not require knowing $$ p(x) $$, but instead requires the ratio $$ {Z(\theta)}/{Z(\theta')}. $$ It cannot therefore be applied in the setting of doubly-intractable distributions.

<div class="l-body">
    <img class="img-fluid" src="{{ site.baseurl }}/assets/img/blog/2020-12-30-exchangeMCMC/alg1.svg">
</div>



# The ExchangeMCMC algorithm


The ExchangeMCMC algorithm (Murray et al., 2012, <d-cite key="murray2012mcmc"></d-cite>) bypasses this issue by drawing an auxiliary observation

$$ x' \sim p(\cdot\vert \theta') $$

and defining the acceptance probability as:
 
\begin{equation}\label{Eq:acc_rate_exchange}
\alpha = \frac{q(\theta\vert \theta') \pi(\theta')p(x\vert \theta')}{q(\theta'\vert \theta)\pi(\theta) p(x\vert \theta)}\textcolor{blue}{\frac{p(x'\vert \theta)}{p(x'\vert \theta')}} = \frac{q(\theta\vert \theta') \pi(\theta')\tilde p(x\vert \theta')\textcolor{blue}{\tilde p(x'\vert \theta)}}{q(\theta'\vert \theta)\pi(\theta)\tilde p(x\vert \theta)\textcolor{blue}{\tilde p(x'\vert \theta')}}  \frac{\cancel{Z(\theta)\textcolor{blue}{Z(\theta')}}}{\cancel{Z(\theta')\textcolor{blue}{Z(\theta)}}},
\end{equation}
where the black part is the same as in the original Metropolis-Hastings algorithm, but adding the blue terms allow to evaluate $$ \alpha $$ without estimating the normalizing constants. Still, the resulting algorithm targets the correct distribution. 


<div class="l-body">
    <img class="img-fluid" src="{{ site.baseurl }}/assets/img/blog/2020-12-30-exchangeMCMC/alg2.svg">
</div>



# Some extensions


## Bridging

The acceptance rate in Eq.\eqref{Eq:acc_rate_exchange} depends on two ratios: $$ \frac{p(x\vert \theta')}{p(x\vert \theta)} $$ represents how well the proposed parameter value explains the observation with respect to the previous parameter value, while $$ \frac{p(x'\vert \theta)}{p(x'\vert \theta')} $$ measures how well the auxiliary variable (generated using $$ \theta' $$) can be explained with parameter $$ \theta $$.

Therefore, even if the former is large and $$ \theta' $$ would be a suitable parameter value, $$ \alpha $$ can still be small if the auxiliary random variable is not explained well by the previous parameter value. This can lead to slow mixing of the chain; to improve on this, Murray et al., 2012, proposed to sample a set of auxiliary variables $$ (x'_0, x'_1, \ldots, x'_K) $$ from intermediate distributions in the following way: consider a set of densities: 
$$
\tilde p_k(x	\vert \theta, \theta') = \tilde p(x\vert \theta')^{\beta_k} \tilde p(x\vert \theta)^{1- \beta_k}, \quad \beta_k = \frac{K - k + 1}{K + 1};
$$
$$ x'_0 $$ is generated from $$ p(\cdot\vert \theta') $$ as before, and then each $$ x'_k $$ is generated from $$ R(\cdot \vert  x'_{k-1}; \theta, \theta') $$, which denotes a Metropolis-Hastings transition kernel starting from $$ x'_{k-1} $$ with stationary density $$ \tilde p_k(\cdot\vert \theta, \theta') $$. Then, the acceptance rate is modified as follows: 
\begin{equation}\label{Eq:acc_rate_exchange_bridging}
\alpha = \frac{\tilde p(x\vert \theta')q(\theta\vert \theta') \pi(\theta')}{\tilde p(x\vert \theta)q(\theta'\vert \theta)\pi(\theta)} \cdot \prod_{k=0}^{K} \frac{\tilde p_{k+1}(x'_k\vert \theta, \theta') }{\tilde p_k(x'_k\vert \theta, \theta') }.
\end{equation}

Note that $$ K=0 $$ recovers the original ExchangeMCMC. This procedure, usually called _bridging_, generally improves the acceptance rate as it basically introduces a sequence of intermediate updates to the auxiliary data which by smoothening out the difference between the two distributions. 

<div class="l-body">
    <img class="img-fluid" src="{{ site.baseurl }}/assets/img/blog/2020-12-30-exchangeMCMC/alg3.svg">
</div>




## ExchangeMCMC without perfect simulations.

If it is not possible to sample from $$ p(\cdot\vert \theta') $$ as it is required in the ExchangeMCMC algorithm, Murray et al., 2012, suggested to run $$ T_{in} $$ steps of an MCMC chain on $$ x $$ targeting $$ p(\cdot\vert \theta') $$ at each step of ExchangeMCMC; if $$ T_{in} $$ is large enough, the last sample can be considered as (approximately) drawn from $$ p(\cdot\vert \theta') $$ itself and used in place of the unavailable perfect simulation.

In practice, however, this only leads to an approximate ExchangeMCMC algorithm, as at each iteration of the inner chain a finite $$ T_{in} $$ is used, so that the inner chain would not perfectly converge to its target; for this reason, even an infinitely long outer chain would not target the right posterior for any finite $$ T_{in} $$. 

Nonetheless, this approach was shown empirically to work satisfactorily <d-cite key="liang2010double"></d-cite> <d-cite key="caimo2011bayesian"></d-cite> <d-cite key="everitt2012bayesian"></d-cite>; moreover, it has been argued <d-cite key="liang2010double"></d-cite> that starting the inner chain from the observation value improves convergence.
 
Some theoretical guarantees (albeit under strongs conditions), are given in Appendix B of Everitt, 2012 <d-cite key="everitt2012bayesian"></d-cite>, which bounds the total variation distance between the target of approximate ExchangeMCMC with finite $$ T_{in} $$ and the target of the exact one, and shows that they become equal when $$ T_{in} \to \infty $$. 


<div class="l-body">
    <img class="img-fluid" src="{{ site.baseurl }}/assets/img/blog/2020-12-30-exchangeMCMC/alg4.svg">
</div>




# Related algorithms.
ExchangeMCMC algorithm is very popular due to its ease of use. However, it is certainly not the only algorithm to sample from doubly intractable distribution (see the nice review given in Park and Haran, 2018 <d-cite key="park2018bayesian"></d-cite>); some are listed next: 

<ul>
    <li>Caimo et al., 2011 <d-cite key="caimo2011bayesian"></d-cite> proposed a parallel-chain MCMC algorithm</li>
    <li>Everitt et al., 2012<d-cite key="everitt2017marginal"></d-cite> built an SMC-type algorithm which is also capable of recycling information from past simulations</li>
    <li>Finally, Liang et al., 2016 <d-cite key="liang2016adaptive"></d-cite> proposed an algorithm which is inspired from ExchangeMCMC and, in the case of impossible perfect sampling, still targets the right invariant distribution. This algorithm requires some hand-tuning and needs to keep in memory a large amount of data, which may hinder its applicability.</li>
</ul>

Differently from ExchangeMCMC, the above ones can be parallelized.
