---
title: "Bayesian A/B Testing"
# title: "Unveiling the Power of Bayesian A/B Testing: An Example-driven Exploration"
date: 2023-05-29
draft: false
author: "Weishan He"
tags:
  - Bayesian Inference
  - A/B Testing
  - Python
# image: /images/mathjax.png
description: "In this article, I aim to guide you through the essential process of Bayesian A/B testing using an example along with some practical codes."
toc: true
mathjax: true
---

During my time as a Data Scientist at Andela, I had the opportunity to explore the realm of Bayesian A/B testing, **an approach renowned for its robust performance in small sample sizes**. Like many others, my introduction to experimentation was through Frequentist A/B testing. As I familiarized myself with concepts like p-values and statistical power, it took some time for me to streamline the bayesian framework of conducting experiments. In this article, I aim to guide you through the essential process of Bayesian A/B testing using an example along with some practical codes.

The Bayesian A/B testing, intuitively speaking, revolves around updating our prior beliefs with new information, which is the data we collect through experimentation. Let's consider a scenario where you are working as a Data Scientist for an online travel startup. One day, the product manager approaches you with the idea of testing whether removing a specific page from the sign-up process could potentially increase the sign-up rate. However, since the website caters to business clients like hotels, the traffic volume is relatively low.

Given these circumstances, you decide to employ Bayesian A/B testing to investigate this hypothesis. The advantage of using Bayesian A/B testing in this scenario is that it doesn't impose strict sample size requirements. Additionally, it allows for the updating of posterior distributions as data accumulates. This flexibility makes Bayesian A/B testing a great choice for your experiment in order to make data-driven decisions for the startup.

## Step 1: Determine the randomized unit
To conduct the experiment, the first step is to determine the randomized unit. Since our goal is to improve the sign-up rate, we focus on new customers as the target for this test.

## Step 2: Select the metrics of interest
Next, we should define the metrics of interest. **Considering the limited sample size, it is advisable to focus primarily on the metrics at the top of the customer journey funnel.** Therefore, we select sign up rate as our primary metric, which is calculated by dividing the number of individuals who successfully complete the sign-up process by the total number of new website visitors.

## Step 3: Choose the proper prior distribution for your metric of interest
**Selecting an appropriate prior distribution poses a significant challenge when conducting Bayesian inference. It is at this juncture that the realms of business and statistics intersect.** As a data scientist, effective communication with stakeholders is crucial in determining the typical distribution of the sign-up rate. Industry benchmarks, research results, practical constraints and more can come in handy for reference. [This medium article](https://medium.com/the-researchers-guide/finding-the-best-distribution-that-fits-your-data-using-pythons-fitter-library-319a5a0972e9) also gives some guidance to find the distribution that fits your data using a python package called Fitter.

In addition to flexibility, Bayesian approach is also known for its rather rigorous computation. Fear not! Let me introduce you to the concept of **conjugate priors**, which significantly ease the burden of updating the prior distributions. When the posterior distribution remains within the same family as the prior distribution, we refer to such priors as conjugate priors. 

In the context of A/B testing, our metrics of interest often fall into two categories: binary (such as click-through rate and sign-up rate) or continuous (such as average revenue and total revenue). I have compiled a table below, presenting the commonly used conjugate priors and their corresponding updating functions for these two data types. If you want to dive deeper into the subject of conjugate priors, I would recommend referring to [the class materials on this topic from MIT](https://ocw.mit.edu/courses/18-05-introduction-to-probability-and-statistics-spring-2014/pages/class-slides/).


|Data Types| Conjugate Priors | Updating Function|
|:---------|:-----------------|:-----------------|
|Binary - binomial distribution|Beta distribution|\\(\alpha + \sum_{i=1}^{n}x_i \\), \\(\beta + \sum_{i=1}^{n}N_i - \sum_{i=1}^{n}x_i\\)|
|Continuous - normal distribution (CLT)|Normal distribution assuming known variance|\\( m'' = \frac{\sigma^2 m' + n(\sigma')^{2}m}{n(\sigma')^{2} + \sigma^2} \\), \\((\sigma'')^2 = \frac{\sigma^2(\sigma')^2}{n(\sigma')^2 + \sigma^2} \\), \\( m''\\) and \\((\sigma'')^2 \\) are the posterior mean and variance, \\(m'\\) is the prior mean, \\(m\\) is the sample mean, \\(n\\) is the sample size |

## Step 4: Apply Monte Carlo simulation to obtain key statistics
Once we obtain the posterior distribution, we can proceed to calculate essential statistics that help inform recommendations based on the results of Bayesian A/B testing. These statistics include the posterior mean, credible interval, and probability of being the best.

The first statistic, the posterior mean, represents the average value of the metric of interest derived from the posterior distribution.

The second is the credible interval. 95% credible interval is the center 95% of the data gained from the posterior distribution. One question that naturally arises is what is the difference between credible intervals in Bayesian approach and confidence intervals in frequentist approach. A frequentist 95% confidence intervals mean that with a large number of repeated samples, the 95% confidence intervals would include the true value of the metric of interest. While 95% credible intervals mean that given the observed data, the metric of interest has 95% probability of falling within this range. 

Interestingly, people usually interpret the frequentist confidence intervals the same way as Bayesian credible intervals, which is wrong. Bayesian credible intervals are constructed based on the understanding that the estimated parameters are random variables with a distribution. Therefore, a credible interval represents an interval in the domain of the posterior distribution where an unobserved parameter value is likely to fall with a particular probability. This differs from the frequentist approach, where the true value is considered fixed and assigning a probability to it is not applicable. By embracing the belief that the estimated parameter is a random variable with a distribution, Bayesian approach provides a more intuitive interpretation of credible intervals.

Lastly, to determine the probability of being the best, we can employ Monte Carlo simulation by sampling with replacement from the posterior distribution. The probability of being the best can be calculated as follow:
$$ \text{probability of being the best = number of wins / number of trials} $$

In this context, "wins" refers to the number of instances where the treatment or variant being evaluated outperforms the alternatives, and "trials" represents the total number of simulated experiments conducted.

[Here is some sample code](https://github.com/WeishanHe/causal_inference/blob/main/bayesian_ab_testing_example.ipynb) to get these statistics.

## Step 5: Making Conclusions
Probability of being the best is a critical statistic utilized for decision-making, often guiding launch decisions in various companies. Some organizations set a threshold of at least 90% for the winning probability before proceeding with a particular variant. However, it is essential to note that the threshold can be adjusted based on the specific stage of the company. For instance, if a company is in the early stages of building its platform and aims to incorporate new features while taking calculated risks, it may be reasonable to adopt a lower threshold. This approach allows for more experimentation and exploration of potential avenues for growth. On the other hand, a more mature company may prefer to limit the number of features unless they possess the potential to deliver significant impact. In such cases, a higher threshold for the winning probability is preferred, ensuring that only the most promising variants are pursued. **Ultimately, the choice of threshold depends on the company's objectives, risk appetite, and growth stage**, allowing for customized decision-making based on the unique context and priorities of the organization.

In addition to probability of being the best, we can also answer the following questions empowered by the posterior distribution:
1. **Will we do any damage by launching? (Type S)**:
By examining the area of posterior distribution below 0, we can assess the likelihood of negative impacts or potential harm associated with launching a particular variant. This valuable insight allows us to mitigate risks and make informed decisions that prioritize user experience and business success.
2. **Are we off by a magnitude? (Type M)**: The posterior mean enables us to evaluate the magnitude of differences between variants. We can analyze the extent to which a particular variant outperforms or underperforms others, allowing us to gauge the practical significance of the observed effects.
3. **The probability of falling below a minimal threshold**: We can utilize the posterior distribution to estimate the probability of a metric falling below a predetermined minimum threshold. This analysis helps us identify potential risks associated with performance levels that do not meet the desired standards or requirements.

Please stay tuned for the next post, where I will dive into a challenging topic of Bayesian A/B testing: determining when to conclude the experiments without strict sample size limitations. Don’t miss out on unlocking the power of decision-making in this flexible field!

---
## References
1. [Finding the Best Distribution that Fits Your Data Using Pythons Fitter Library](https://medium.com/the-researchers-guide/finding-the-best-distribution-that-fits-your-data-using-pythons-fitter-library-319a5a0972e9)
2. [Talking Bayes to Business: A/B Testing Use Case [Video]](https://www.youtube.com/watch?v=J6kqvWnUE2Q&t=1s)
3. [Bayesian AB Testing and Its Benefits](https://towardsdatascience.com/bayesian-a-b-testing-and-its-benefits-a7bbe5cb5103)

