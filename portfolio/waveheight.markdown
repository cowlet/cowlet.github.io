---
layout: default
title: Wave height forecasting
---

# Wave height forecasting for offshore access

<div class="keywords">
keywords: forecasting, neural nets, SVMs, ensembles, metrics
</div>

Engineers need to periodically access offshore wind turbines to perform maintenance. The day hire of a crew transfer ship is approximately $2500, but the vessel can only sail when waves are below 1.5m in height. Wind farm operators use wave height forecasting to judge whether waves will stay below the limit for the duration of the round trip. An inaccurate forecast will have a financial penalty, either through a wasted vessel hire or a missed opportunity to return the turbine to peak efficiency. 

## The Data Science

I built wave height forecasters using neural networks, support vector machines, and an ensemble, and compared them against others developed by colleagues. My ensemble forecaster significantly outperformed all individual forecasters when calculating metrics such as Root Mean Squared Error (RMSE). However, we developed an improved metric specific to forecasting offshore access due to wave height.

![Root Mean Squared Error of each forecaster at different prediction
horizons](/portfolio/assets/RMSE.png)

## Results

Forecasting of wave height for vessel access is more accurately treated as a classification problem (is wave height above the limit?) than a regression problem (what is the wave height in metres?). When considered this way, the size of the regression error is irrelevant, as long as it is on the correct side of the access limit. The financial penalties for an error are also different, depending on whether it is a "false above threshold" prediction or "false below". The new economic metric takes these factors into account, and finds that the SVM outperforms other methods, but not by much.

<figure>
<img src="/portfolio/assets/Econ.png" alt="Economic score of each
forecaster">
<figcaption>Economic Forecasting Metric (EFM) score for each forecaster,
showing Cost of False Below (CFB) and Cost of False Above (CFA) penalties</figcaption>
</figure>

## Resources

Papers detailing this work:

- [An economic impact metric for evaluating wave height forecasters for offshore wind maintenance access][journal]
- [Wave height forecasting to improve off-shore access and maintenance scheduling][conf]

[journal]:  http://strathprints.strath.ac.uk/51047/
[conf]:     http://strathprints.strath.ac.uk/44744/

