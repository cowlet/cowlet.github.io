---
layout: default
title: Conditional Anomaly Detection 
---

# Detecting Anomalous Transformer Behaviour

<div class="keywords" style="font-size: 16px; color: #a09; line-height: 1em; margin-bottom: 1em">
keywords: correlation, anomaly detection, reducing false positives
</div>

A transformer can operate in the power network for 60 years or more, and
over that time develop a signature pattern of behaviour. Standard diagnostic
tools will flag these quirks as requiring repair, but as long as the
behaviour of the transformer isn't changing, this aged behaviour is not
serious enough to fix. Anomaly detection is more useful than diagnostics for
older transformers, so that any deviations away from its usual pattern of
quirky behaviour are caught and flagged for repair. 

## The Data Science

Anomaly detection techniques have a key disadvantage: they can generate lots
of false positives where behaviour has changed, but not due to a developing
fault. Changes in the transformer's environment, such as hot days, high
winds, and high current flow, can all affect the transformer's behaviour in
ways that may look like a fault.

I selected Conditional Anomaly Detection (CAD) as an appropriate technique
for this application. CAD learns two probabilistic models -- one of the
environmental parameters, and one of the condition indicator parameters --
then learns the correlations between the two. Anomalies are only flagged
when abnormal condition indicators occur in a normal environment.

![Block diagram of the CAD model](/portfolio/assets/CAD_arch.png)

## Results

I applied CAD to two transformers on the network in England, which were
reaching the end of their design life. Online monitoring showed no anomalous
behaviour within the transformers, and they remained operational for an
extra 18 months. Two months of data were used to train the CAD model, which
detected 21 anomalies over the next 16 months. These anomalies were all due
to sensor and logging issues, where plausible but incorrect values were
recorded for transformer condition parameters.

<figure>
<img src="/portfolio/assets/CAD_output.png" alt="Model output showing four
anomalies">
<figcaption>Model output showing four anomalies, where a low probability can
be seen in the final output of the CAD model (fCAD) and the condition
indicator model (P(ind)), but the corresponding probability of the
environmental conditions (P(env)) is normal.</figcaption>
</figure>

## Resources

Papers detailing this work:

- [Online conditional anomaly detection in multivariate data for transformer monitoring][journal]
- [On-line transformer condition monitoring through diagnostics and anomaly detection][conf]

[journal]:  http://strathprints.strath.ac.uk/14806/
[conf]:     http://strathprints.strath.ac.uk/26475/

