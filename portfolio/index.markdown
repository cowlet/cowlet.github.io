---
layout: portfolio
title: Victoria Catterson
description: "Highlights of my data science projects"
---

<div id="particles-js">
<div class="header">
  <span class="title"><a href="/">Victoria Catterson</a></span>
</div>
<div class="container about">
<div class="photo"><img src="/assets/victoria.jpg" alt="Photo of Victoria Catterson" /></div>
<div class="bio">
<p>I'm a data scientist with experience of translating engineering problems into machine learning solutions. Here is a sample of the interesting projects I've worked on.</p>
<p>There's some <a href="https://github.com/cowlet">related code on my GitHub page</a>. If you have any questions, send me an email at <a href="mailto:vic@cowlet.org">vic@cowlet.org</a>.</p>
</div>
</div>
</div>

<div class="container">
<div class="portfolio-item">
<h2><a href="/portfolio/waveheight.html">Wave height forecasting for offshore turbine access</a></h2>

<div class="keywords">
keywords: forecasting, neural nets, SVMs, ensembles, metrics
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/RMSE.png" alt="Wave height accuracy" width="300px">
</figure>
</div>

<p>
Engineers need to periodically access offshore wind turbines to perform maintenance. This work developed a suite of wave height forecasters to predict whether waves will stay below the access limit, and an improved metric for assessing the strength of a forecaster.
</p>
</div>

<div class="portfolio-item">
<h2><a href="/portfolio/transformers.html">Predicting remaining useful life of transformers</a></h2>

<div class="keywords">
keywords: predictive analytics, particle filtering, Bayesian, uncertainty
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/txoverload.png" alt="Transformer life prediction" width="300px">
</figure>
</div>

<p>
Transformers are the most expensive components of power networks. This
research resulted in a probabilistic model for predicting remaining
transformer life, based on the ambient conditions it operates in. This can
be used to speculate about the effects of emergency overloading, as well as
predicting normal behaviour.
</p>
</div>

<div class="portfolio-item">
<h2><a href="/portfolio/dnns.html">Defect diagnosis using deep neural networks</a></h2>

<div class="keywords">
keywords: deep learning, neural nets, feature analysis
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/numlayers.png" alt="DNN accuracy" width="300px">
</figure>
</div>

<p>
Insulation is used in all electrical equipment to prevent short-circuit
faults. This work improved the accuracy of diagnosis of insulation defects
by using a deep approach. It also led to insights about important parts of
the data, through a comparison of features learned by the deep network and
typical engineered features.
</p>
</div>

<div class="portfolio-item">
<h2><a href="/portfolio/harmonics.html">Identifying harmonics from condition data</a></h2>

<div class="keywords">
keywords: rule induction, decision trees, majority voting, data reuse
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/harmonicbatch.png" alt="Harmonics accuracy" width="300px">
</figure>
</div>

<p>
The data used to diagnose insulation defects can be distorted by harmonics
on the power network. This work proposed an alternative to costly
harmonic monitoring, by diagnosing the presence of harmonics from the same
condition data. This gives a clearer picture of equipment health,
separating harmonic effects from defect effects.
</p>
</div>

<div class="portfolio-item">
<h2><a href="/portfolio/cad.html">Detecting anomalous transformer behaviour</a></h2>

<div class="keywords">
keywords: correlation, anomaly detection, reducing false positives
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/CAD_arch.png" alt="CAD block diagram" width="300px">
</figure>
</div>

<p>
Older transformers develop a signature pattern of behaviour that is specific
to each one. Anomaly detection is more useful than diagnostics in this case,
since it's more important to spot changes in behaviour than to repeatedly
classify minor problems. This work applied Conditional Anomaly Detection to
keep two transformers in service for 18 months, while minimising false
positives.
</p>
</div>

<div class="portfolio-item">
<h2><a href="/">A popular introduction to data science</a></h2>

<div class="keywords">
keywords: features, k-means, neural nets, decision trees, SVMs, kNN
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/quantile.png" alt="Bearing features" width="300px">
</figure>
</div>

<p>
A growing number of people are interested in data science, but academic
papers and formal training can be difficult ways to break in. My personal
site shows <a href="http://cowlet.org">how to get started with data science
in R</a>, through a series of articles covering feature extraction and
selection, clustering, and classification using multiple techniques. These
are demonstrated on an open dataset of faults within bearings.
</p>
</div>

<div class="portfolio-item">
<h2><a href="http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118755480.html">A book chapter on data analytics</a></h2>

<div class="keywords">
keywords: regression, classification, prognostics, diagnostics
</div>

<div style="float: right; margin-left: 20px">
<figure>
<img src="/portfolio/assets/sgh_cover.jpg" alt="Book cover" width="150px">
</figure>
</div>

<p>
I was asked to contribute to the forthcoming <a
href="http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118755480.html">Smart
Grid Handbook</a> by writing a chapter on "Data Analytics for Transmission
and Distribution". I drew on my experience of developing and deploying data
analytics in this field to highlight the increasing role that online
analytics can play, and some lessons learned from each case study.
</p>
</div>
</div>
