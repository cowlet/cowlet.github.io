---
layout: default
title: Deep neural networks for insulation diagnosis
---

# Diagnosing faults in insulation using deep neural networks

Insulation is used in all electrical equipment to prevent short-circuit and
other network faults. Insulation defects can lead to small electrical
discharges which partially cross the insulation (called partial discharge,
or PD). If the defect is not repaired, the rate and size of PDs will
increase over time, and eventually lead to a full breakdown of the
equipment. Utilities need to know what type of defect is causing the PD, in
order to plan the correct repair.

## The Data Science

The standard approach to online PD diagnosis is to collect samples of PD
data, calculate a set of features, and [use one or more classifiers to
diagnose the defect][evcomb]. The typical size of a feature vector is very
large (50 to 100 features), and the relationship of each feature to the
diagnosis is not very clearly understood.

[evcomb]:   http://strathprints.strath.ac.uk/11785/

In this research, I used a deep belief network approach to increase the
accuracy of the diagnosis. As a secondary effect, I could inspect the
features learned by the first layer of the deep network, and compare them
against the typical features derived from expert judgement.

<div style="float: left">
<figure>
<img src="/portfolio/assets/numneurons.png" alt="Recall accuracy of two layer networks with different numbers of neurons" width="300px">
<figcaption>Accuracy of two layer networks with different numbers of neurons</figcaption>
</figure>
</div>

<div style="float: auto">
<figure>
<img src="/portfolio/assets/numlayers.png" alt="Recall accuracy of deep networks with different numbers of layers" width="300px">
<figcaption>Accuracy of deep networks with different numbers of layers</figcaption>
</figure>
</div>


## Results

The weighting of each neuron was visualised, to see which parts of the PD
image it was responding to. This showed that some neurons emphasised
particular quadrants of the image, which is similar to the expert-derived
features. Other neurons responded to precise patterns associated with
particular defects. This was particularly interesting, because it is the
approach that a human expert would take to data analysis, but it is very
different from the typical features calculated for diagnosis by shallow
learning.

<figure>
<img src="/portfolio/assets/neuronpics.png" alt="Examples of neuron activation, visualized by input weightings. Each image represents one neuron">
<figcaption>Examples of neuron activation, visualized by input weightings. Each image represents one neuron</figcaption>
</figure>

## Resources

The paper detailing this work:

- [Deep neural networks for understanding and diagnosing partial discharge data][conf]

[conf]:     http://strathprints.strath.ac.uk/53898/

