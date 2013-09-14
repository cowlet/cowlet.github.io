---
layout: post
title:  "Data preparation"
---

Conventional wisdom for bearing analysis says that the most important features are the amplitude of vibration at key frequencies: the ball pass outer race (BPFO), ball pass inner race (BPFI), ball spin frequency (BSF), and fundamental train frequency (FTF). (For [detailed explanation][mobius] and [equations][equns], please follow these links.) We can calculate these frequencies for the experimental bearings, using [data contained in this paper][phm-paper].

[mobius]:   http://www.mobiusinstitute.com/articles.aspx?id=2088
[equns]:    http://www.ntnamericas.com/en/website/documents/brochures-and-literature/tech-sheets-and-supplements/frequencies.pdf
[ph-paper]: https://www.phmsociety.org/sites/phmsociety.org/files/phm_submission/2009/phmc_09_018.pdf

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




So those outliers we can see are at 49.8Hz, 57.6Hz, and 493Hz. None of these are obvious harmonics of the four key frequencies. However, the fourth largest frequency is a harmonic of the second largest (493 * 2 = 986), which suggests they are linked. 

A number of the other top 20 frequencies are near 986Hz -- calculating the differences gives:

<table>
  <tr>
    <th>Frequency f</th>
    <th>f - 986</th>
  </tr>
  <tr> <td>993.3</td> <td>6.836</td></tr>
  <tr> <td>979.6</td> <td>-6.836</td></tr>
  <tr> <td>994.2</td> <td>7.813</td></tr>
  <tr> <td>978.6</td> <td>-7.813</td></tr>
  <tr> <td>995.2</td> <td>8.790</td></tr>
  <tr> <td>971.8</td> <td>-14.65</td></tr>
  <tr> <td>969.8</td> <td>-16.60</td></tr>
</table>





