---
layout: post
title:  "Data preparation"
---

The first step of working with data is to get it into whatever tool or environment you're working in. This can take a surprising length of time, since even the cleanest, best formatted data will have some curious feature such as a weird timestamp format, and in most instances the data will have gaps and bad readings that will have to be handled somehow.

The data I'm using is from the [Prognostics Data Repository hosted by NASA][nasa], and specifically the [Bearing data set, IMS, University of Cincinnati][bearingset] from J. Lee, H. Qiu, G. Yu, J. Lin, and Rexnord Technical Services, 2007 (warning: 780MB download). This data relates to four bearings installed on a loaded rotating shaft, with a constant speed of 2000rpm. The bearings have a design life of 100,000,000 revolutions. Assuming constant use and speed, this is 50,000 minutes, 833 hours, or 34.7 days. The test set-up is shown below (from [Qiu et al][qiu]):

[nasa]:      http://ti.arc.nasa.gov/tech/dash/pcoe/prognostic-data-repository/
[bearingset]: http://ti.arc.nasa.gov/c/3/
[qiu]:       http://www.sciencedirect.com/science/article/pii/S0022460X0500221X

![Bearing experiment set-up, Qiu et al.]({{ site.url }}/assets/qiu-et-al.jpg)

Bearings are key components of any type of rotating machinery, from consumer level devices such as a desk fan, up to industrial plant such as in nuclear power stations. Anything with a motor or generator rotates a shaft, and the health of the bearings determines how smoothly and efficiently that shaft rotates. Poor lubrication of the bearing will generate extra friction, which reduces efficiency and can also damage the bearing or shaft. In the worst cases, the bearing can scuff or crack, and metal particles break free to hit and damage other parts of the machine.  

A common method of monitoring bearings is to measure vibration, since the more smoothly the system is operating, the lower the level of vibration. For this reason, the test set contains measurements from two accelerometers per bearing (one each for the x- and y-axes). Data was recorded in 1 second bursts every 5 or 10 minutes, and the sampling rate was 20kHz. 

When you download and unzip the [data set][bearingset], it contains a PDF describing these experiment details, and three RAR files. Un-rarring 1st\_test.rar gives a directory containing 2,156 files. Each file is named with its timestamp, and contains an 8 x 20,480 matrix of tab-separated values. The 8 columns correspond to the accelerometers (2 each for 4 bearings), and the rows are the 20kHz samples from 1 second of operation.

This raises the first question about the data. If the sampling rate is 20kHz and each file contains 1s of data, there should be 20,000 rows of data. Since there are 20,480, one of these pieces of information about the data is wrong. Is it the sampling rate or the time period? It's more likely that the sampling rate is correct and that each file contains just over 1s of data. For high frequency data capture, it's imperative that the sampling rate is correctly adhered to. If a 20kHz setting is actually sampling at 20.48kHz, it introduces significant error into the experiment. On the other hand, 20,480 samples at a rate of 20kHz gives a time period of 1.024s, which is close enough to be rounded down to "1 second" in the text documentation. It's important to check that all the information you have about some data fits together, because discrepancies can indicate a misunderstanding, which can in turn lead to meaningless models.

<h2>Importing the data</h2>

Being fairly confident that I understand what this data is now, it's time to read it into R. A pattern I use regularly is to set a variable `basedir` with the path to the directory I'm working in, then combine this with a specific filename using `paste0`. (The `paste0` function is strangely named, but it just joins strings together with 0 characters between them.) Since the source data is in text format, with no column headers, and columns separated by tabs, the read in code is like this:

{% highlight r %}
basedir <- "/Users/vic/Projects/bearings/bearing_IMS/1st_test/"
data <- read.table(paste0(basedir, "2003.10.22.12.06.24"), header=FALSE, sep="\t")
head(data)
{% endhighlight %}

Calling `head` on the data is just to check that the read has done what we expect, and should give this:

<pre class="terminal">
<code>
          V1     V2     V3     V4     V5     V6     V7     V8
    1 -0.022 -0.039 -0.183 -0.054 -0.105 -0.134 -0.129 -0.142
    2 -0.105 -0.017 -0.164 -0.183 -0.049  0.029 -0.115 -0.122
    3 -0.183 -0.098 -0.195 -0.125 -0.005 -0.007 -0.171 -0.071
    4 -0.178 -0.161 -0.159 -0.178 -0.100 -0.115 -0.112 -0.078
    5 -0.208 -0.129 -0.261 -0.098 -0.151 -0.205 -0.063 -0.066
    6 -0.232 -0.061 -0.281 -0.125  0.046 -0.088 -0.078 -0.078
</code>
</pre>

These column names are not very intuitive, so we can rename them like this:

{% highlight r %}
colnames(data) <- c("b1.x", "b1.y", "b2.x", "b2.y", "b3.x", "b3.y", "b4.x", "b4.y")
{% endhighlight %}

<p></p>

<h2>Initial analysis</h2>

There is a vast array of possible analysis and modelling techniques we could apply to this data, and 2,155 more files of 1s bursts to read in. However, at this stage I don't have a good feel for what the data looks like or how it behaves, so I can't yet decide the best way to format, store, or clean the data. Data mining is an iterative process where you look at the data from a certain angle to generate ideas for more interesting angles to consider next. Simplest is always the best way to start:


{% highlight r %}
summary(data$b1.x)
plot(data$b1.x, type="l") # make it a line plot
{% endhighlight %}

<pre class="terminal">
<code>
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    -0.72000 -0.14600 -0.09500 -0.09459 -0.04200  0.38800 
</code>
</pre>

![Plot of bearing 1 x axis vibration]({{ site.url }}/assets/first-b1x.png)

Looking just at the x-axis of bearing 1 to start with, we can see that the mean vibration value is less than 1, and mostly within the band between 0.0 and -0.2. The max and min values (0.388 and -0.720) are outliers, but not orders of magnitude different from the rest of the data, and therefore can't be discounted as bad data. The spikes in value seem regularly spaced, and a spike in negative value always seems to correspond with an extreme positive value, which tends to suggest these outliers are true measurements. The same analysis of the other columns shows very similar patterns, with a slight tendency for the y axes to have a mean closer to 0, but higher extreme values. 

Each 1s plot represents a single snapshot in the life of that bearing. I have to decide what sort of analysis approach I'm going to take with these bearings, since that will impact what representation is best for the data. Here I'll outline the two broad approaches which come to mind.

The first I'll call the traditional engineering approach. Since 20,480 is a very high number of data points to represent one measurement, and we have over 2000 more such snapshots, the traditional approach is to try and condense this data down through feature extraction. We could calculate for each file a handful of representative features, such as mean, max, and min values, and the relative size of key harmonic frequencies. With 2156 files, we would end up with 2156 rows of (say) 8 columns of features, and can easily perform modelling on this condensed dataset.

An alternative could be called the Big Data approach, where 20,480 x 2156 is only 44 million datapoints, and with a good pipeline of tools and a bit of time, we can perform analytics on the full data without needing to condense. It is only relatively recently that this approach has become feasible, so there is less experience and guidance around about how to do this efficiently. 

On the plus side, feature extraction aims to increase the information content in individual data points, by drawing signal out of noise. As long as your features are representative of the process you are trying to model, nothing is lost in the condensing process, but the modelling itself become much easier. On the other hand, if your features don't relate well to the underlying process, you may be losing valuable information from the source data.

To begin with, I'll take the engineering approach. 


<h2>Formatting the full dataset</h2>

To collect the data into a format useful for further analysis, we need to process the 2156 time ordered source files into 4 files of bearing-specific data. Each bearing file will contain 2156 rows (one per source file), containing the timestamp and key features calculated from both the x- and y-axes. 

Conventional wisdom for bearing analysis says that the most important features are the amplitude of vibration at key frequencies, relating to rotation and spin of the bearing elements. For now I'll take more of a data-driven approach and focus on patterns in the data, and not use domain knowledge to calculate specific frequencies.

A vibration signal can be decomposed into its frequency components using the Fast Fourier Transform (FFT). Simply calling the R function `fft` returns data in a novice-unfriendly format, but it's straightforward to turn it into a more intuitive version:

{% highlight r %}
b1.x.fft <- fft(data$b1.x)
# Ignore the 2nd half, which are complex conjugates of the 1st half, 
# and calculate the Mod (magnitude of each complex number)
amplitude <- Mod(b1.x.fft[1:(length(b1.x.fft)/2)])
# Calculate the frequencies
frequency <- seq(0, 10000, length.out=length(b1.x.fft)/2)
# Plot!
plot(amplitude ~ frequency, t="l")
{% endhighlight %}

![FFT of bearing 1 x-axis vibration]({{ site.url }}/assets/fft-b1x.png)

Great! You can see that all the good stuff is going on down at the lower frequencies, although there is a ripple of activity just above and below 4kHz, and around 8kHz. For now, let's focus on the lower frequencies. 

{% highlight r %}
plot(amplitude ~ frequency, t="l", xlim=c(0,1000), ylim=c(0,500))
axis(1, at=seq(0,1000,100), labels=FALSE)  # add more ticks
axis(1, at=seq(0,100,10), labels=TRUE) # add labels only every 10th
{% endhighlight %}

![Plot of low frequencies of bearing 1 x-axis FFT]({{ site.url }}/assets/fft-zoomed-b1x.png)

The tallest spikes by far are just below 1kHz. Generally the dc term (zero frequency) of an FFT can be ignored, as it always dominates and doesn't really contain information. There is also a large spike just below 500Hz, and two around 50Hz. Tabulating the top 20 frequencies gives:

{% highlight r %}
sorted <- sort.int(amplitude, decreasing=TRUE, index.return=TRUE)
top15 <- sorted$ix[1:15] # indexes of the largest 15
top15f <- frequency[top15] # convert indexes to frequencies
{% endhighlight %}

<pre class="terminal">
<code>
     [1]    0.00000  986.42446  993.26106  493.21223  979.58785  994.23772  969.82127  971.77459
     [9]   57.62281  978.61119  921.96504   49.80955 4420.35355 3606.79754 4327.57105
</code>
</pre>

So those outliers we can see are at 49.8Hz, 57.6Hz, and 493Hz. Interestingly, the second and fourth largest frequencies have a harmonic relationship (493 * 2 = 986), which strongly suggests they are linked. 

For now, I'll take the frequencies of the largest 15 harmonics to be our features.



The timestamp is contained within the filename. Assuming we have the filename as a string in `filename`, it can be parsed like this:

{% highlight r %}
timestamp <- strptime(filename, format="%Y.%m.%d.%H.%M.%S")
{% endhighlight %}


Finally, the timestamp and features need to be written to a new matrix, and a new row added every time a file is processed. The code for this is below:

TODO




