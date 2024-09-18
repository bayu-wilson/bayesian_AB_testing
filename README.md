# Bayesian A/B Testing using PyMC
This notebook is written by Bayu Wilson but many ideas are based off of a tutorial on the PyMC website: [Introduction to Bayesian A/B testing](https://www.pymc.io/projects/examples/en/latest/causal_inference/bayesian_ab_testing_introduction.html#references)

<p align="center">
  <img src="https://github.com/user-attachments/assets/e9d3700e-6630-457f-b9a0-1db7c8c887c2" alt="AB testing graphic"/><br>
  <em>Figure 1: A/B testing graphic generated using Microsoft Designer (which in turn uses DALL-E). The "BUY NOW" button in the "B" variant is much bigger than in the "A" variant. We will be using Bayesian inference to test if we can confidently say that increasing the button size will improve key performance metrics. </em>
</p>

### Background
In this hypothetical project, I am working for a marketing team for a company that sells products online. We are seeking ways to improve the current marketing campaign to address 3 important key performance indicators (KPIs): 
1. Purchase propensity (probability of customer buying something)
2. Mean Purchase Value 
3. Revenue per visitor (RPV)

The marketing team decides that a larger "BUY NOW" button may help improve these KPIs. The plan is to run a series of A/B tests, with and without the larger "BUY NOW" button, and then use Bayesian inference to quantify the probability that the new variant improves the KPI and, if it does, by how much.

We will be using PyMC. PyMC is a probabilistic programming library for Python that allows users to build Bayesian models with a simple Python API and fit them using Markov chain Monte Carlo (MCMC) methods.

### 1. Purchase propensity / Bernoulli conversion probability
Let $\theta_A, \theta_B$ be the true conversion rates for variants A and B respectively. The beta distribution is used to set the prior Bernoulli probabilities. Then the binomial distribution will be the likelihood. Using Bayes theorem:

$$ P(\theta_i|y) \propto P(\theta_i)P(y|\theta_i)$$ 

$$ \mathrm{Posterior} \propto \mathrm{Prior} \times \mathrm{Likelihood}$$ 

$$  \theta^\mathrm{estimate}_i  \propto \mathrm{Beta}(\alpha,\beta) \times \mathrm{Binomial}(N,y,\theta_i)$$ 

where $\alpha$ and $\beta$ set how well we know the prior and the Binomial distribution is used for the likelihood because we are dealing with the number of successes ($y$) in a sequence of $N_i$ independent experiments with a boolean outcome. 

A good way to quantify the difference between the variants is to use the relative uplift in bernoulli covergence rates:

$$ \mathrm{reluplift}_B^\mathrm{Bern} = \theta^\mathrm{estimate}_B/\theta^\mathrm{estimate}_A-1$$


### 2. Mean Purchase Value / value conversion estimate

Next, let $\lambda_A$ and $\lambda_B$ be the true value conversions for variants A and B respectively. Now the Gamma distribution is the congugate prior and the exponential distribution is the likelihood. Again using Bayes theorem

$$ P(\lambda_i|y) \propto P(\lambda_i)P(y|\lambda_i)$$ 

$$ \mathrm{Posterior} \propto \mathrm{Prior} \times \mathrm{Likelihood}$$ 

$$  \lambda^\mathrm{estimate}_i  \propto \mathrm{Gamma}(\alpha_G,\beta_G) \times \mathrm{exponential}(N,y,\lambda_i)$$ 

The difference between variants can use the relative uplift again but for the value conversion.

$$ \mathrm{reluplift}_B^\mathrm{value} = \lambda^\mathrm{estimate}_B/\lambda^\mathrm{estimate}_A-1$$

### 3. Revenue per visitor (RPV)

In the previous 2 sections, we used Bayes theorem to estimate the bernoulli and value conversion. Both of these are useful but arguably the most important KPI would be the revenue per visitor (RPV).

$$ RPV = \mathrm{ProbabilityOfPaying} \times \mathrm{AvgAmountSpentWhenPaying}$$

$$ \mu^{estimate}_i = \theta^{estimate}_i \times \frac{1}{\lambda^{estimate}_i}$$

where $\mu^{estimate}_i$ represents the estimated average revenue per visitor. We can additionally calculate the relative uplift.

$$ \mathrm{reluplift}_B = \frac{\mu_B}{\mu_A}-1 $$

### Then we generate data, find the likelihoods, combine likelihoods with prior distributions, then sample the posterior probability distribution of the KPI's of interest using pyMC

Here is one of the main results:

<p align="center">
  <img src="https://github.com/user-attachments/assets/5a8f464c-17c2-457b-bf06-1d834abc04f1" alt="AB testing posteriors"/><br>
  <em>Figure 2: Posteriors of the three KPI's of interest. Here we are using 100,000 samples per variant.</em>
</p>

#### Main takeaways
- Bayesian A/B testing methodologies can be useful to quantifiably say whether or not the B variant improved KPI's
- It seems that 100,000 samples per variant is enough such that the 94% high-density interval (HDI) is above zero in the relative uplift distributions for the three KPI's. Additionally, for the RPV relative uplift, we can use the HDI to say that there is a 94% chance that the new variant increased the RPV by 6-17% (true value is 16%).




