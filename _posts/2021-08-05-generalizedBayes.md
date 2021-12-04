---
layout: distill
output:
  distill::distill_article:
    toc: true
    toc_depth: 2
title: Generalizing Bayesian Inference 
date: 05-08-2021
date_published: 0
description: "Updating a 250 years old theorem for the 21st century"

authors:  
  - name: Lorenzo Pacchiardi
    url: ""
    affiliations:
      name: Oxford

bibliography: 2021-08-05-generalizedBayes.bib


toc:
  - name: The original Bayes theorem
    # if a section has subsections, you can add them as follows:
    subsections:
      - name: Properties of Bayes posterior
      - name: So what is wrong?
  - name: Generalizations of Bayes posterior
    subsections:
      - name: Power likelihood
      - name: Loss-based Bayesian inference
      - name: Generalized Variational Inference
      - name: Recap
  - name: Conclusion

---

If you are reading this, you probably know what Bayes' theorem is. Here, we are concerned with the use of Bayes' theorem to perform inference for parameters of a statistical model (a.k.a. **Bayesian inference**). 

Recently, several generalizations of Bayesian inference have been proposed. The aim of this post is to survey some of these generalizations and 
overview the pros and cons of each with respect to using the original Bayes theorem. 
Hopefully, I'll be able to put some order in this rapidly expanding literature and provide some intuition on why we need to move beyond the original Bayes theorem. 

**TL; DR:** Extensions of Bayesian inference work better in some cases which are not contemplated by using the original Bayes' theorem (primarily, model misspecification), and provide more justification.

# The original Bayes theorem

<div style="text-align:center;font-size:15px;">
<div class="l-body">
    <img class="img-fluid" src="https://upload.wikimedia.org/wikipedia/commons/d/d4/Thomas_Bayes.gif">
</div>
<i>
A portrait of Thomas Bayes (maybe). 
Source: <a href="https://commons.wikimedia.org/wiki/File:Thomas_Bayes.gif">Unknown author</a>, Public domain, via Wikimedia Commons.</i>
</div>

<br>
Let's first have a look at Bayes' original theorem <d-footnote>This is actually the form that was given to it by Laplace; click <a href="https://www.lesswrong.com/posts/RTt59BtFLqQbsSiqd/a-history-of-bayes-theorem" target="_blank" > here </a> for a nice history of the theorem. </d-footnote>:

$$
\pi(\theta\vert x) = \frac{\pi(\theta) p(x\vert \theta)}{p(x)}, \qquad p(x) = \int \pi(\theta) p(x\vert \theta) d\theta.
$$

Here, I am using $$ \theta $$ to denote a _parameter_ on which we want to perform inference, while $$x$$ is the _data_ that we observe. 

As the standard textbook description of Bayesian inference says, Bayes' theorem provides a _posterior belief_ $$ \pi(\theta\vert x) $$, by updating the _prior belief_ $$ \pi(\theta) $$ with the information on $$\theta$$ carried by $$x$$. Crucially, all information on $$\theta$$ contained in $$x$$ is represented by the _likelihood_ $$ p(x\vert \theta) $$, such likelihood being the probabilistic model from which **we believe** $$ x $$ was generated. This is the gist of Bayesian inference.

Actually, the denominator in the definition of the posterior is independent on $$\theta$$. Exploiting this fact, it is common to write: 

$$
\pi(\theta\vert x) \propto \pi(\theta) p(x\vert \theta),
$$

where the proportional sign refers to both sides of the equation being considered as function of $$\theta$$<d-footnote>When some posterior expectations need to be computed, the function on the right-hand side needs to be normalized (ie divided by the normalizing constant $p(x)$). However, computing the normalizing constant is not needed when sampling from the posterior with MCMC methods, for instance.</d-footnote>.

Additionally, Bayes' theorem is very modular, allowing to sequentially incorporate new information into our belief: say that we previously had data $$x_1$$, from which we obtained $$\pi(\theta\vert x_1)$$. If we now get an independent observation $$x_2$$, we can apply Bayes' theorem again by using $$\pi(\theta\vert x_1)$$ as a prior and get:  

$$
\pi(\theta\vert x_1, x_2) \propto  \pi(\theta\vert x_1) p(x_2\vert \theta).
$$

So, if we get $$n$$ independent observations, say $$\mathbf {x} = (x_1, x_2, \ldots, x_n) $$, the posterior belief is: 

$$
\pi(\theta\vert \mathbf x) = \frac{\pi(\theta) \prod_{i=1}^n p(x_i\vert \theta)}{p(\mathbf x)} \propto \pi(\theta) \prod_{i=1}^n p(x_i\vert \theta).
$$

## Properties of Bayes posterior

The posterior obtained from Bayes' theorem satisfies some nice properties; first, there are properties that are intrinsic to Bayes' update rule:

- __coherence:__ in whatever order you process the observations $$x_i$$, you end up with the same posterior belief<d-footnote>This property may be given different names in other works, as for instance <i>Bayesian additivity</i>.</d-footnote>. 
- __Likelihood principle:__ As the observations $$\mathbf x$$ only appear through the likelihood in the definition of the posterior, Bayes' theorem satisfies the likelihood principle, which says that (from [Wikipedia](https://en.wikipedia.org/wiki/Likelihood_principle)):
    > given a statistical model, all the evidence in a sample relevant to model parameters is contained in the likelihood function.

If you further assumes that the observations $$ \mathbf x$$ are generated from $$p(\cdot\vert\theta_0)$$ for some parameter value $$\theta_0$$ (i.e., the model is _well specified_), Bayes' posterior satisfies some additional properties:

- according to information theory arguments in Zellner <d-cite key="zellner1988optimal"></d-cite>, Bayes theorem is the __optimal way to process information__, in the sense that its use does not discard any information present in the data about the parameters.

- An analogue to the Central Limit Theorem in frequentist statistics applies to Bayes' posterior, called __Bernstein-von Mises theorem__. It goes like this: as the number of observations $$n$$ goes to infinity, the posterior converges to a normal distribution which is centered in $$\theta_0$$ (and whose variance decreases as $$1/n$$ and is asymptotically equivalent to the sampling variance of the maximum likelihood estimator of $$\theta$$)<d-footnote>The theorem of course holds under some regularity conditions, such as $\pi(\theta_0)>0$; click <a href="https://encyclopediaofmath.org/wiki/Bernstein-von_Mises_theorem#References" target="_blank" > here </a> for an introduction and some further references. </d-footnote>. That means that Bayesian inference is, asymptotically, equivalent to maximum likelihood estimation, as it recovers the exact parameter value $$\theta_0$$. 


## So what is wrong?

To recap what we said above, the main motivation behind Bayes' theorem is that, if the model $$p(x\vert \theta)$$ is well specified, the posterior distribution is a coherent way to learn about the true parameter value (to which it converges in the limit of infinite data); additionally, that is the best possible way to process information even with a finite amount of observations (by Zellner's argument). 

However, these arguments vacillate when the model is not an accurate representation of the distribution $$g(\cdot)$$ from which observations were generated (or data-generating process, DGP). By this, we mean that there do not exist any parameter value such that $$p(\cdot\vert\theta)=g(\cdot)$$. If that is the case, two things happen:

- Zellner's argument simply does not hold anymore <d-cite key="holmes2017assigning"></d-cite>.
- A Bernstein von-Mises result still holds, but now the asymptotic normal distribution will be centered in the parameter value
$$\theta^\star = \arg \min_{\theta} D_{\text{KL}}(g(\cdot)\vert \vert p(\cdot|\theta)),$$
where $$D_{\text{KL}}$$ is the Kullback-Leibler (KL) divergence<d-footnote> Notice that $\theta^\star$ is also the parameter value to which the frequentist maximum likelihood estimate converges</d-footnote>. 

More in general, we need to ask what is the aim of Bayesian inference in such a **misspecified** setting. In fact, with Bayesian inference we do not learn about the _true_ parameter value anymore, as such thing does not exist. Rather, the standard Bayes' posterior learns about the parameter value such that the misspecified model is as close as possible to the DGP in the specific sense of the KL divergence. 

That may still be what you want to do in some cases, but I argue below that using the KL divergence may behave poorly in some misspecified cases; instead, some generalized Bayesian inference allow the user to choose the way in which you approximate the DGP with the probabilistic model (for instance replacing the KL with other divergences <d-cite key="pacchiardi2021generalized"></d-cite><d-cite key="jewson2018principles"></d-cite>). Others instead completely dispose of a probabilistic model.

### Issues with the KL divergence

Learning $$\theta^\star$$ according to the KL divergence may not behave well when the model is misspecified; for instance, consider the following DGP: 

$$ g(x) = 0.1\ h(x) + 0.9\ p(x|\theta_0); $$

this means that 90% of observations are generated from the model for a given parameter value $$\theta_0$$, while the other 10% are generated from the distribution $$h$$. If $$h$$ has heavier tails than $$p$$, $$\theta^\star$$ can be far away from $$\theta_0$$, as the KL divergence gives large importance to the tail behavior of the distributions<d-cite key="jewson2018principles"></d-cite>. With a finite amount of samples, that translates into saying that Bayes' posterior is highly sensitive to outliers in the data. 

With more general misspecifications, things may go wrong in different ways. 

### Prior and computational limits

Finally, Bayesian inference implicitly assumes that the prior is a good representation of previous knowledge, and that enough computational power is available to sample from the posterior (which, in some cases, it is very expensive to do) <d-cite key="knoblauch2019generalized"></d-cite>.   

Some of the generalized Bayesian approaches <d-cite key="knoblauch2019generalized"></d-cite> explicitly include the fact that these assumptions may be broken in the definition of the inference strategy, as we will see below.


# Generalizations of Bayes posterior

In the following, I review extensions to Bayes theorem which are more justified and may have better performances in a misspecified setting (or can even be used without a probabilistic model); some will also tackle directly the issues regarding prior and computational power.

Across these works, a recurrent underlying question is: what is the actual aim of inference when we cannot specify the model correctly?  

**Disclaimer**: this overview is non-exhaustive and strongly biased due to papers I've read and my personal research activity. 


## Power likelihood

A first idea to tackle (mild) model misspecification is to reduce the importance of the likelihood term in the definition of Bayes' posterior. This can be done by raising the likelihood function to a power $$\alpha \in [0,1]$$, where $$\alpha=1$$ recovers Bayes' posterior and $$\alpha=0$$ corresponds to sticking to the prior. The resulting posterior is therefore:  


$$
\pi_\alpha(\theta\vert x) \propto \pi(\theta) p(x\vert \theta)^\alpha,
$$


This idea has been discussed in detail in<d-cite key="holmes2017assigning"></d-cite> (and previous papers referred there). The authors of <d-cite key="holmes2017assigning"></d-cite> also propose a way to automatically tune $$\alpha$$; this way works by matching the expected information gain in two experiments, one involving the exact data-generating process and the other one involving instead the best model approximation. This strategy recovers $$\alpha=1$$ in the well specified case, and sets $$\alpha<1$$ otherwise. Other methods for tuning $$\alpha$$ are reviewed in <d-cite key="wu2020comparison"></d-cite>.

With respect to standard Bayes' posterior, this strategy does not change the parameter value on which learning is performed, but it only changes the speed of learning (ie the rate of concentration of the posterior distribution with an increasing number of observations). Also, it still satisfies the likelihood principle and coherence.  


## Loss-based Bayesian inference

A larger leap is taken in <d-cite key="bissiri2016general"></d-cite>. There, the authors replace the likelihood term with the exponential of a loss function: 

\begin{equation}\label{Eq:bissiri}
\pi_{\ell,w}(\theta\vert x) \propto \pi(\theta) \exp\{ - w \cdot \ell (\theta,x)\},
\end{equation}

where $$w$$ is a weight which balances the influence of the loss and prior term. With this update, as argued in <d-cite key="bissiri2016general"></d-cite>, you learn about the parameter value minimizing the expected loss: 

$$
\int \ell (\theta,x) g(x) dx,
$$

where $$g$$ is the data-generating process. 

In <d-cite key="bissiri2016general"></d-cite>, they show how the update rule above (Eq. \eqref{Eq:bissiri}) can be derived axiomatically from the task of learning about the parameter value minimizing the expected loss, by assuming the observed data to be independent from the prior and inference to be **coherent** in the sense defined above (ie invariant to the order in which the observations were obtained). 

An advantage to this approach is that you do not need to specify a probabilistic model (ie likelihood) here; inference can be performed on "parameter" values that are solely defined through the loss function.  

Of course, the power likelihood posterior is obtained as a special case by setting $$\ell(\theta, x) = - \log p(x\vert \theta)$$, and $$w=\alpha$$; further setting $$w=1$$ gives back the original posterior.

However, with a generic loss, setting the value of $$w$$ is an arbitrary choice. The larger $$w$$, the faster the concentration of the posterior with the increase of the number of observations. In <d-cite key="bissiri2016general"></d-cite>, some heuristic techniques to tune $$w$$ are discussed; in general, however, the choice of $$w$$ is a subjective choice of the user, which may be driven by practical and computational reasons as well. 

Finally, notice that a Bernstein-von Mises result still hold for this more general posterior (of course under some regularity conditions). Some very general and applicable formulations are given in <d-cite key="miller2019asymptotic"></d-cite>. 


### Scoring rules-based Bayesian inference

So, the loss-based approach in Eq. \eqref{Eq:bissiri} works without a probabilistic model. But what if you have a misspecified model which you believe carries some meaning about the process you are studying?

As mentioned above, using the original Bayes' posterior may not be a wise choice. In order to perform inference in a sounded way, an idea is to use the loss-based approach and express the loss $$\ell$$ as a function of the probabilistic model: 

$$
\ell(\theta, x) = S(p(\cdot|\theta), x),
$$

where $$S$$ is a function of a probability distribution and an observation, which is usually called _scoring rule_ in the literature<d-cite key="gneiting2007strictly"></d-cite>. Therefore, this yields the following update: 

$$
\pi_{S,w}(\theta\vert x) \propto \pi(\theta) \exp\{ - w \cdot S (p(\cdot\vert\theta),x)\}.
$$

This approach has been investigated in some recent works, amongst which <d-cite key="pacchiardi2021generalized"></d-cite>, <d-cite key="jewson2018principles"></d-cite> , <d-cite key="loaiza2019focused"></d-cite> and <d-cite key="giummole2019objective"></d-cite>. In this way, you learn about the parameter value minimizing the expected scoring rule over the data generating process $$g$$: 

$$
\int S (p(\cdot|\theta),x) g(x) dx.
$$

For some scoring rules $$S$$, the above expectation corresponds to a statistical divergence (for instance the energy distance and MMD <d-cite key="pacchiardi2021generalized"></d-cite>). Therefore, you can learn about the parameter value for which the model is closer to the data generating process according to different divergences from the KL one (which, as discussed at the beginning, is what the original Bayes' posterior leads to). If you are interested for instance in prediction tasks (for which scoring rules are often used as evaluation criteria), this approach is a way to obtain better predictions with a misspecified model in a Bayesian setting<d-cite key="loaiza2019focused"></d-cite>. 

Clearly, this still gives a coherent update, but the likelihood principle is not satisfied anymore (as we use the likelihood, but do not just evaluate it at the observation); however, the likelihood principle itself does not seem very reasonable if the model is misspecified in the first place<d-cite key="jewson2018principles"></d-cite>.


## Generalized Variational Inference
One more step towards generality and we find the approach presented in <d-cite   key="knoblauch2019generalized"></d-cite>.  

The idea is to start from the variational formulation of Bayes' posterior which is attributed to Donsker and Varadhan <d-cite key="donsker1975asymptotic"></d-cite>; say we have $$n$$ independent observations $$\mathbf {x} = (x_1, x_2, \ldots, x_n) $$; then, it can be shown that Bayes' posterior is defined by:  

$$
\pi(\cdot|\mathbf x) = \underset{q \in \mathcal P (\Theta)}{\operatorname{argmin}}\left\{\mathbb{E}_{q(\theta)}\left[-\sum_{i=1}^{n} \log \left(p\left(x_{i} \mid {\theta}\right)\right)\right]+D_{\text{KL}}(q \| \pi)\right\},
$$

where $$\Theta$$ denotes the values that $$\theta$$ can take, $$\mathcal P (\Theta)$$ is the set of all probability distributions on $$\Theta$$, and $$D_{\text{KL}}(q \| \pi)$$ is the KL divergence between the distribution $$q$$ and the prior $$\pi$$. 

This formulation leads to an _optimization-centric_ view of Bayesian inference, as the authors of <d-cite key="knoblauch2019generalized"></d-cite> put it; additionally, the loss-based posterior in Eq. \eqref{Eq:bissiri} <d-cite key="bissiri2016general"></d-cite> can be obtained in a similar fashion by just replacing the negative log-likelihood with the generic loss function $$\ell$$: 

$$
\pi_{\ell,w}(\cdot|\mathbf x) = \underset{q \in \mathcal P (\Theta)}{\operatorname{argmin}}\left\{\mathbb{E}_{q(\theta)}\left[w \sum_{i=1}^{n} \ell(\theta, x_i) \right]+D_{\text{KL}}(q \| \pi)\right\}.
$$

This formulation open the doors to additional extensions by changing $$D_{\text{KL}}$$ in the optimization objective above to a different divergence, say $$D$$. Further, you can also restrict the minimization problem to a constrained space of distributions $$\mathcal \Pi \subseteq \mathcal P(\Theta)$$. The resulting update rule obtained in this way is termed _the Rule of Three (RoT)_, as you obtain a posterior distribution by specifying the loss, the space of distributions $$\mathcal \Pi$$ and the divergence $$D$$: 

$$
\pi_{\ell, D, \mathcal \Pi}(\cdot|\mathbf x) = \underset{q \in \mathcal \Pi }{\operatorname{argmin}}\left\{\mathbb{E}_{q(\theta)}\left[w \sum_{i=1}^{n} \ell(\theta, x_i) \right]+D(q \| \pi)\right\} \stackrel{\text{def}}{=} P(\ell, D, \Pi).
$$

However, it is in general impossible to obtain a closed form solution for the above problem. Additionally, you do not have a coherent update anymore. 

In the paper <d-cite key="knoblauch2019generalized"></d-cite>, they show how the above problem generalizes standard Variational Inference (VI, in which you select the distribution in a given class which minimizes the KL divergence from the exact posterior), hence the name of the approach. Therefore, the RoT unifies under the same hat variational inference, loss based posteriors and the standard Bayes' posterior. It is important to notice how, instead, VI approaches which employ different divergences from the KL (which they call divergence VI) are not included<d-footnote>This means that standard VI (with the KL) is theoretically optimal with respect to divergence VI as the former directly approximates the exact posterior. That however is restricted to the case in which the likelihood and prior are well specified, or to the case in which the variational family $\Pi$ is large enough; if that is not the case, it may be that divergence VI gives better results (as it is sometimes observed in the literature; see Section 2.4 in <d-cite key="knoblauch2019generalized"></d-cite>).</d-footnote>. 

The rest of the work discusses in very much detail (more than 100 pages!) the properties of the resulting posterior. The key idea is that with this approach you can not only perform inference with a generic loss (as it was already with the loss-based approach by <d-cite key="bissiri2016general"></d-cite> in Eq. \eqref{Eq:bissiri}), but also choose which properties of the prior to consider by picking the right $$D$$, and finally reducing the computational burden posed by an unconstrained $$\mathcal P(\Theta) $$ by choosing a suitable space of probability distributions $$\Pi$$. That therefore tackles the other issues with standard Bayesian inference that I mentioned above.

They also provide an axiomatic derivation of the RoT; interestingly, among the required axioms is a generalized likelihood principle, which states that all information on $$\theta$$ contained in $$\mathbf x$$ is accessible through evaluation of the loss $$\ell$$ at the datapoints. 


## Recap

I have gone up the ladder of generality, from standard Bayes' posterior to the General Variational Inference formulation; at each step, some features of standard Bayes' posterior are lost and some others are retained, but a larger set of tools become available. Those may work better for instance with misspecified models, or in case you do not even want to specify a full probabilistic model, or finally if a full Bayesian analysis is too costly and you want to instead resort to a variational approach. 

The following Venn's diagram represents the relation between the different techniques presented here: 

<div class="l-body">
    <img class="img-fluid" src="{{ site.baseurl }}/assets/img/blog/2021-08-05-generalizedBayes/venn_diagram.svg">
</div>


# Conclusion

I have followed here a possible generalization route, but that is by no means the only possibility and my overview does not include all methods which are a superset of standard Bayes' posterior - I have for instance excluded the approach taken in <d-cite key="fong2021martingale"></d-cite> which puts the focus on the predictive distribution rather than the likelihood, or the one in <d-cite key="masegosa2019learning"></d-cite>, which uses PAC-Bayesian bounds to define distributions which do not concentrate on one single parameter value in the limit of infinite data if the model is misspecified, thus obtaining better predictive performance.  

Still, I think the approaches reviewed here are thought-provoking and allow to see Bayesian inference under a different light. I also feel they bring the original axiomatic theory closer to a pragmatical toolbox.

Of course, I do not think now that standard Bayesian inference has to be thrown to the garbage; there is no need to list the practical results that have been possible with it. Still, it is good to know there are ways around (or beyond) in case a standard Bayesian analysis doesn't work. Additionally, some of the techniques developed to work with standard posteriors (for instance MCMC and VI) can be ported to generalized posteriors.   

I am sure there will be other extensions proposed in the future, so keep an eye out if you are interested!