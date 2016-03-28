---
layout: default
title: Identifying harmonics from unconventional data
---

# Identifying network harmonics from condition data

A common health monitoring technique for high voltage equipment is to
measure small electrical currents called [partial discharge (PD) in the
insulation][dnns]. The pattern of PD can be used to diagnose the type of
defect present, which allows the utility to plan the correct repair.
However, it has recently been discovered that harmonics on the high voltage
network can also alter the PD pattern. Harmonic monitoring is expensive, but
an accurate diagnosis can't be made unless the harmonics are known.

[dnns]:     /portfolio/dnns.html

## The Data Science

I recognised that particular harmonics distort the PD pattern in predictable
ways. Also, certain harmonics are more likely to be present on the network
that others, with the 5th and 7th harmonics particularly common. Pattern
recognition techniques are typically used to diagnose the defect, so I
could use the same approach to diagnose the presence of key harmonics.

Since classifying harmonics is different from classifying defects, the most
informative features in the data may be quite different. I generated a large
set of features, and used a process of feature selection to identify those
that contributed best to harmonic classification. I built C4.5 rule
induction classifiers to identify the presence of the 5th and 7th harmonics,
with moderate accuracy on unseen data (around 60%).

<figure>
<img src="/portfolio/assets/harmonics.pdf" alt="Accuracy of classifiers on different test waveforms">
<figcaption>Accuracy of classifiers on different test waveforms</figcaption>
</figure>


## Results

While the accuracy on a single PD pattern was modest, a single pattern
represents just 80ms of data. Typically, PD will be recorded over a few
minutes to a few days, so a conclusion about harmonics can be drawn from
a number of patterns in a row. I tried to increase the accuracy by using
majority voting on sequential patterns. This worked, with accuracy tending
towards an asymptote as batch size increases.

<figure>
<img src="/portfolio/assets/harmonicbatch.pdf" alt="Accuracy of the 5th
harmonic majority voting classifier as batch size increases">
<figcaption>Accuracy of the 5th harmonic majority voting classifier on the training data as batch size increases</figcaption>
</figure>

## Resources

Papers detailing this work:

- [Identifying harmonic attributes from online partial discharge data][journal]
- [Assessing the effects of power quality on partial discharge behaviour through machine learning][conf]

[journal]:  http://strathprints.strath.ac.uk/33135/
[conf]:     http://strathprints.strath.ac.uk/26479/

