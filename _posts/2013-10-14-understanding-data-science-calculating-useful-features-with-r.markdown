---
layout: post
title:  "Understanding data science: calculating useful features with R"
description: "Building a good model requires a strong set of useful features. This article gives an example of calculating a feature vector for an open dataset in R."
---

Building a good data model or classifier requires a strong set of useful features as input. [Previously, I showed how to begin analysis on a new dataset][feature-extraction] by reading in and plotting an open dataset, then calculating a simple feature vector. Here, I build on this by expanding the feature set, through assessment of the three key types of features in data.

[feature-extraction]: {% post_url 2013-09-15-understanding-data-science-feature-extraction-with-r %}


## Why calculate features?

The goal of feature extraction is to condense out important information, in order to simplify pattern recognition. Humans are very good at looking at graphs and spotting clusters, patterns, and outliers, but training a software system to do the same can be much harder. In particular, coincidental links between parameters will distract the system from the intended pattern, and the more data there is, the more likely these spurious links will arise. Calculating a set of representative features is one way to reduce the sheer number of datapoints, and therefore get the system to focus on the right pattern.

However, there still needs to be enough data to usefully distinguish between patterns of behaviour. If we take a person's temperature (a single feature) and it looks a little high, it's difficult to tell whether they are ill or have just been exercising. Instead, we need features to provide enough context to separate out healthy and unhealthy states. Also, it's difficult to tell for a new dataset which features will prove to be best at identifying different states or behaviours. 

Therefore, a good approach is to calculate a very large feature set for some data, and use statistical methods to identify the best subset for a given task. Thinking of features to generate can be a challenge itself, but there are three broad categories to consider:

* Domain-specific features, where you use some knowledge about the problem to identify likely points of interest in the data,
* Statistical features, which require no knowledge of the domain, but characterise the data purely based on properties such as mean, median, and standard deviation,
* Visually derived features, where you see some pattern in the data and develop a method to capture it.

For the [bearing dataset from University of Cincinnati][bearingset], I'll explore each of these types of feature in turn. To get started, follow the [previous article][feature-extraction] or use this snippet to read in the first file of data:

{% highlight r %}
basedir <- "/Users/vic/Projects/bearings/bearing_IMS/1st_test/"
data <- read.table(paste0(basedir, "2003.10.22.12.06.24"), header=FALSE, sep="\t")
colnames(data) <- c("b1.x", "b1.y", "b2.x", "b2.y", "b3.x", "b3.y", "b4.x", "b4.y")
plot(data$b1.x, t="l")
{% endhighlight %}

This plot should look like this:

![Plot of bearing 1 x axis vibration](/assets/first-b1x.png)


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

I'll use a similar function to [the last article to calculate the FFT][feature-extraction], but this time return the full FFT spectrum:

{% highlight r %}
fft.spectrum <- function (d)
{
	fft.data <- fft(d)
	# Ignore the 2nd half, which are complex conjugates of the 1st half, 
	# and calculate the Mod (magnitude of each complex number)
	return (Mod(fft.data[1:(length(fft.data)/2)]))
}
{% endhighlight %}

A slightly complex aspect of the FFT is that it will return the same number of frequency buckets or bins as half the number of input points, while the frequency range covered will be from 0Hz to half the sampling frequency (called the [Nyquist frequency][nyquist]). For the bearing dataset, the number of data points is not exactly the same as the frequency range, and so a frequency bin does not correspond to a whole Hertz. Specifically, there are 20,480 data points per file, which is halved to get the number of frequency bins (10,240). But the sampling frequency of the data is 20kHz, which gives a Nyquist frequency of 10kHz. 

[nyquist]:      http://en.wikipedia.org/wiki/Nyquist_frequency

In practical terms, this means that every frequency bin is 10,000/10,240 = 0.9766Hz. It's easiest to write a helper function to convert a frequency in Hertz into a bin number (index number into the FFT spectrum array):

{% highlight r %}
freq2index <- function(freq)
{
	step <- 10000/10240 # 10kHz over 10240 bins
	return (floor(freq/step))
}
{% endhighlight %}

Now I can put these two functions together with the key bearing frequencies (in Hertz) calculated earlier, to generate the first four features. 

{% highlight r %}
fft.amps <- fft.spectrum(data$b1.x)
features <- c(fft.amps[freq2index(ftf)], 
              fft.amps[freq2index(bpfi)], 
              fft.amps[freq2index(bpfo)], 
              fft.amps[freq2index(bsf)])
{% endhighlight %}

For the first bearing data file, this gives the following amplitudes:

<pre class="terminal">
<code>
[1] 10.474550  6.952257  6.448870 25.312705
</code>
</pre>

In addition to key frequencies, engineers are often concerned with the root mean squared (RMS) value of a sinusoidal signal. When the signal contains both positive and negative values, the RMS can be more informative than a mean value, which often comes out around zero. 

The RMS can be thought of as a statistical measure, but since it is an engineer's preferred averaging technique, I'll count it as a domain knowledge feature. It can be calculated and appended to the feature list like this:

{% highlight r %}
features <- append(features, sqrt(mean(data$b1.x**2)))
{% endhighlight %}


## Statistical features

Without knowing anything about the data or domain, there are standard statistical features which can characterise the shape of the data. Broadly speaking, these features can tell you how closely the data matches a [Normal distribution (bell curve)][normal].

[normal]:       http://en.wikipedia.org/wiki/Normal_distribution

The first two are the mean and standard deviation (or variance) of the data. The next is the [skewness][skew], or whether the majority of the datapoints are greater or less than the mean value. Another is [kurtosis][kurt], or how peaked or flat a curve the data fits to.

[skew]:         http://en.wikipedia.org/wiki/Skewness
[kurt]:         http://en.wikipedia.org/wiki/Kurtosis

Next up are quantiles, where the data is separated into four equally sized sets. The boundary values of the sets form five features. The minimum and maximum values in the dataset are always the first and last features, while the median of the dataset is always the third feature. The second and fourth can be thought of as the medians of the lower half (data between the minimum and median points) and the upper half (between the median and the maximum) of the dataset.

The R code to calculate these features is quite short and neat, but the skewness and kurtosis require a library:

{% highlight r %}
library(e1071)
features <- append(features, 
                   c(quantile(data$b1.x, names=FALSE), 
                     mean(data$b1.x), 
                     sd(data$b1.x), 
                     skewness(data$b1.x), 
                     kurtosis(data$b1.x)))
{% endhighlight %}


## Visually derived features

The starting point for data science is always to visualise the data, which involves creating lots of plots and looking for patterns. Sometimes, I'll see a pattern which seems to correspond to the underlying thing I'm looking for, which is bearing failure in this case. If I'm really lucky that day, this will be something simple like a variable increasing around the time of a known failure. More likely, it will be some sort of shape or change which can be quite hard to describe, but feels promising.

For the last article, the features chosen were the frequencies of the strongest five vibration components, as calculated by FFT. In particular, the second strongest component looked promising as a feature, since bearings 3 and 4 (which failed) looked different from bearings 1 and 2, and these differences only became apparent during the final half of the experiment. Without knowing exactly when the failures started, I have to rely on hints like this, and so I'll add these features to the overall set.

For the final group of features, I again looked at some plots and used a bit of intuition. I noticed that the domain-specific bearing frequencies (shown with dashed lines below) were often not the highest spikes in the FFT profile:

{% highlight r %}
frequency <- seq(0, 10000, length.out=length(data$b1.x)/2)
plot(fft.amps[1:(length(data$b1.x)/2)] ~ frequency, t="l", ylab="Relative power")
abline(v=bsf,col="green",lty=3)
abline(v=bpfo,col="blue",lty=3)
abline(v=bpfi,col="red",lty=3)
abline(v=ftf,col="brown",lty=3)
{% endhighlight %}

![Bearing 1 x axis FFT at the start of experiment](/assets/fft-b1x-start.png)

Zooming in to frequencies between 0 and 1000Hz makes this even clearer:

![Bearing 1 x axis FFT up to 1000Hz](/assets/fft-b1x-start-zoom.png)



For data taken near the start of the experiment, this is not very surprising: a fresh bearing shouldn't generate fault-related vibration frequencies. So let's consider bearing 3, which was known to fail with an inner race fault, and should therefore strongly affect the BPFI frequency. Here's the FFT for bearing 3, two files before failure (BPFI is the right-most line in red):

![Bearing 3 x axis FFT at the end of experiment](/assets/fft-b3x-end.png)

Zooming in again:

![Bearing 3 x axis FFT up to 1000Hz](/assets/fft-b3x-end-zoom.png)

The BPFI has clearly increased, but it's only one change amongst many. The profile as a whole looks very different, with many areas dense with spikes that were previously much more flat. This change in profile seems like something that should be captured by a feature.

However, it's not immediately obvious how to describe this change. There are more spikes, but what counts as a spike is relative to the rest of the profile, not an absolute threshold. It could be said to look more "bushy" or noisy, but while this might explain things to a human, we have to get to the bottom of what bushiness means if we're to calculate a feature for it.

Returning to an engineering understanding of what the data means, this represents much higher levels of vibration than earlier traces. It's not that one or two main frequencies have increased, but more that clusters of frequencies have all increased together. The wear and debris in the faulty bearing is causing more vibration across the spectrum, and it can be described as increasing the power of vibration in various bands of frequencies.

Therefore, it leads me to think the frequency spectrum should be split into bands, and the power of all the components within those bands calculated. 

* Use engineering judgement plus data visualisation to select bands
* Code to calculate



## Results

Some graphs/outcomes

## Conclusions

Summary and next steps.




