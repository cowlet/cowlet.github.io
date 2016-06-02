---
layout: portfolio-item
title: Transformer Lifetime Estimation 
---

# Predicting Remaining Useful Life of Transformers

<div class="keywords">
keywords: predictive analytics, particle filtering, Bayesian, uncertainty
</div>

Transformers are the most expensive components of power networks. Over time,
the heat within a transformer causes its paper insulation to become brittle.
Eventually, the paper loses its mechanical strength entirely, and the
insulation disintegrates. There is no practical, cost-effective way of
measuring the remaining strength of the paper. Utilities need predictive
tools to estimate the remaining useful life (RUL) of their transformers based on
ambient conditions.

## The Data Science

There is an IEEE standard which gives an equation for the rate of transformer aging, given the ambient temperature and transformer current. However, this deterministic model doesn't take account of the uncertainties in this process (such as measurement error and initial paper strength). 

I proposed adding a probabilistic layer to this model through Bayesian
particle filtering. By quantifying the sources of uncertainty, the model can
give a probability density function (PDF) of remaining transformer life,
instead of a point estimate.

![PDFs of remaining transformer life at different points in time](/portfolio/assets/txtest.png)

## Results

I identified the key sources of uncertainty in the IEEE model, and proposed
appropriate ways of quantifying them by drawing on the most up-to-date
research in the literature. I built a version of the probabilistic model for
an in-service transformer in Cumbernauld, Scotland, and used this model to
predict remaining useful life of the transformer under normal conditions,
and under overload conditions. This gives engineers a tool to speculate
about the effects of overloading the transformer in emergency conditions. 

<figure>
<img src="/portfolio/assets/txoverload.png" alt="Current profile showing an
overload, and the corresponding reduction in transformer life">
<figcaption>Transformer current profile over a 20-hr period containing an overload, with corresponding reduction in transformer life</figcaption>
</figure>

## Resources

Papers detailing this work:

- [Prognostics of transformer paper insulation using statistical particle filtering of on-line data][journal]
- [Prognostic modeling of transformer aging using Bayesian particle filtering][conf]

[journal]:  http://strathprints.strath.ac.uk/54744/
[conf]:     http://strathprints.strath.ac.uk/49432/

