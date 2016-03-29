---
layout: portfolio
title: Victoria Catterson
description: "Highlights of my data science projects"
---

# Portfolio

## [Wave height forecasting for offshore turbine access][waveheight]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: forecasting, neural nets, SVMs, ensembles, metrics
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/RMSE.png" alt="Wave height accuracy" width="300px">
</figure>
</div>

<div style="float: auto">
Engineers need to periodically access offshore wind turbines to perform maintenance. This work developed a suite of wave height forecasters to predict whether waves will stay below the access limit, and an improved metric for assessing the strength of a forecaster.
</div>

[waveheight]:   /portfolio/waveheight.html

## [Predicting remaining useful life of transformers][transformers]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: predictive analytics, particle filtering, Bayesian, uncertainty
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/txoverload.png" alt="Transformer life prediction" width="300px">
</figure>
</div>

<div style="float: auto">
Transformers are the most expensive components of power networks. This
research resulted in a probabilistic model for predicting remaining
transformer life, based on the ambient conditions it operates in. This can
be used to speculate about the effects of emergency overloading, as well as
predicting normal behaviour.
</div>

[transformers]: /portfolio/transformers.html

## [Defect diagnosis using deep neural networks][dnns]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: deep learning, neural nets, feature analysis
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/numlayers.png" alt="DNN accuracy" width="300px">
</figure>
</div>

<div style="float: auto">
Insulation is used in all electrical equipment to prevent short-circuit
faults. This work improved the accuracy of diagnosis of insulation defects
by using a deep approach. It also led to insights about important parts of
the data, through a comparison of features learned by the deep network and
typical engineered features.
</div>

[dnns]:         /portfolio/dnns.html

## [Identifying harmonics from condition data][harmonics]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: rule induction, decision trees, majority voting, data reuse
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/harmonicbatch.png" alt="Harmonics accuracy" width="300px">
</figure>
</div>

<div style="float: auto">
The data used to diagnose insulation defects can be distorted by harmonics
on the power network. This research proposed an alternative to expensive
harmonic monitoring, by diagnosing the presence of harmonics from the same
condition data. This gives a clearer picture of equipment health, by
separating harmonic effects from defect effects.
</div>

[harmonics]:    /portfolio/harmonics.html

## [Detecting anomalous transformer behaviour][cad]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: correlation, anomaly detection, reducing false positives
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/CAD_arch.png" alt="CAD block diagram" width="300px">
</figure>
</div>

<div style="float: auto">
Older transformers develop a signature pattern of behaviour that is specific
to each one. Anomaly detection is more useful than diagnostics as
transformers age, since it's more important to spot changes in behaviour
than to repeatedly classify minor problems. This work applied Conditional
Anomaly Detection to keep two transformers in service for 18 months, while
minimising false positives.
</div>

[cad]:          /portfolio/cad.html

## [A popular introduction to data science][cowlet]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: features, k-means, neural nets, decision trees, SVMs, kNN
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/quantile.png" alt="Bearing features" width="300px">
</figure>
</div>

<div style="float: auto">
A growing number of people are interested in data science, but academic
papers and formal training can be difficult ways to break in. My personal
site shows <a href="http://cowlet.org">how to get started with data science
in R</a>, through a series of articles covering feature extraction and
selection, clustering, and classification using multiple techniques, all
applied to fault detection in bearings.
</div>

[cowlet]:       /index.html

## [A book chapter on data analytics][sghandbook]

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: regression, classification, prognostics, diagnostics
</div>

<div style="float: right">
<figure>
<img src="/portfolio/assets/sgh_cover.jpg" alt="Book cover" width="150px">
</figure>
</div>

<div style="float: auto">
I was asked to contribute to the forthcoming <a
href="http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118755480.html">Smart
Grid Handbook</a> by writing a chapter on "Data Analytics for Transmission
and Distribution". I drew on my experience of developing and deploying data
analytics in this field to highlight the increasing role that online
analytics can play, and some lessons learned from each case study.
</div>

[sghandbook]:   http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118755480.html

