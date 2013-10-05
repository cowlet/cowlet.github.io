---
layout: post
title:  "Understanding data science: calculating useful features with R"
description: "Building a good model requires a strong set of useful features. This article gives an example of calculating a feature vector for an open dataset in R."
---

Building a good data model or classifier requires a strong set of useful features as input. [Previously, I showed how to begin analysis on a new dataset][feature-extraction] by reading in and plotting an open dataset, then calculating a simple feature vector. Here, I build on this by expanding the feature set, through assessment of the three key types of features in data.

[feature-extraction]: {% post_url 2013-09-15-understanding-data-science-feature-extraction-with-r %}


## Why calculate features?

The goal of feature extraction is to condense out important information, in order to simplify pattern recognition. Humans are very good at looking at graphs and spotting clusters, patterns, and outliers, but training a software system to do the same can be much harder. In particular, coincidental links between parameters will distract the system from the intended pattern, and the more data there is, the more likely these spurious links will arise. Calculating a set of representative features is one way to reduce the sheer number of datapoints, and therefore get the system to focus on the right pattern.

However, there still needs to be enough data to usefully distinguish between patterns of behaviour. If we take a person's temperature (a single feature) and it looks a little high, it's difficult to tell whether they are ill or have just been exercising. Instead, we need more features to provide enough context to separate out healthy and unhealthy states. Also, it's difficult to tell for a new dataset which features will prove to be best at identifying different states or behaviours. 

Therefore, a good approach is to calculate a very large feature set for some data, and use statistical methods to identify the best subset for a given task. Thinking of features to generate can be a challenge itself, but there are three broad categories to consider:

* Domain-specific features, where you use some knowledge about the problem to identify likely points of interest in the data,
* Statistical features, which require no knowledge of the domain, but characterise the data purely based on properties such as mean, median, and standard deviation,
* Visually derived features, where you see some pattern in the data and develop a method to capture it.

For the [bearing dataset from University of Cincinnati][bearingset], I'll explore each of these types of feature in turn.

[bearingset]: http://ti.arc.nasa.gov/c/3/


## Domain-specific features

[As described before][feature-extraction], there is a lot of engineering experience with bearings, since they are so widely used. As a result, it's quite well understood that particular types of bearing failure will affect the vibration profile in different ways. There are four key frequencies that are recommended for monitoring, called the ball pass outer race (BPFO), ball pass inner race (BPFI), ball spin frequency (BSF), and fundamental train frequency (FTF). 

If there is a problem such as a crack on one of the balls within the bearing (more properly called a _rolling element_ of the bearing), it will reduce the smoothness of rotation by hitting or bouncing as it comes into contact with other components. If the crack hits a neighbouring rolling element, it will generate increased vibration at the ball spin frequency (BSF), since once per rotation the crack will come back into contact with the neighbour. Alternatively, if the crack is in the _outer race_ (the outer casing surrounding and containing all the rolling elements), it will increase vibration at the BPFO, since this is the frequency at which rolling elements come into contact with a point on the outer race. ([More explanation of these four frequencies is given by the Mobius Institute][mobius].)

To calculate these four frequencies for the dataset bearings, we need some specific information about the structure and size of the bearing components. This can be found in a [paper published by the team who ran the experiment][qiu], and reproduced below. We also need the [equations for calculating BPFO, BPFI, BSF, and FTF][equns], which are widely available online. The R code for the calculations is:

[mobius]:   http://www.mobiusinstitute.com/articles.aspx?id=2088
[equns]:    http://www.ntnamericas.com/en/website/documents/brochures-and-literature/tech-sheets-and-supplements/frequencies.pdf
[qiu]:      http://www.sciencedirect.com/science/article/pii/S0022460X0500221X

{% highlight r %}
Bd <- 0.331 # ball diameter, in inches
Pd <- 2.815 # pitch diameter, in inches
Nb <- 16 # number of rolling elements
a <- 15.17*pi/180 # contact angle, in radians
s <- 2000/60 # rotational frequency, in Hz

ratio <- Bd/Pd * cos(a)
ftf <- s/2 * (1 - ratio)
bpfi <- Nb/2 * s * (1 + ratio)
bpfo <- Nb/2 * s * (1 - ratio)
bsf <- Pd/Bd * s/2 * (1 - ratio**2)
{% endhighlight %}

Which gives the values:

<pre class="terminal">
<code>
            ftf     bpfi     bpfo      bsf
  [1,] 14.77522 296.9299 236.4035 139.9167
</code>
</pre>


* Amplitudes of the 4 key frequencies
* RMS



## Statistical features

* Quantiles, mean, sd, skew, kurtosis.


## Visually derived features

* 5 strongest frequencies (not amplitudes)
* Power in frequency bands

## Results

Some graphs/outcomes

## Conclusions

Summary and next steps.




