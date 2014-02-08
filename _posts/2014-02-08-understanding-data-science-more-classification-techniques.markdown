---
layout: post
title:  "Understanding data science: more classification techniques"
description: "This article shows how to use various classifiers in R, using an open bearings dataset."
---

Last time, I went step by step through [training a neural network classifier in R][ann-post]. In this article, I'll briefly look at three more classifiers:

 - Rpart decision trees,
 - Support vector machines, and
 - k-nearest neighbour classification.

[ann-post]:     {% post_url 2014-01-12-understanding-data-science-classification-with-neural-networks-in-r %}


## The bearings dataset

The diagnostic task is to classify the health of four bearings, using feature vectors derived from vibration data. There are seven health labels, with uneven numbers of samples of each class:

<div class="r_output">
<table>
<thead>
<tr>
<th>early</th>
<th>failure.b2</th>
<th>failure.inner</th>
<th>failure.roller</th>
<th>normal</th>
<th>stage2</th>
<th>suspect</th>
</tr>
</thead>
<tbody>
<tr>
<td>966</td>
<td>37</td>
<td>37</td>
<td>608</td>
<td>4344</td>
<td>317</td>
<td>2315</td>
</tr>
</tbody>
</table>
</div>

This has a couple of implications. First, to get a true reflection of how accurate a classifier is, I need to calculate _weighted accuracy_, based on percentage accuracy for each class rather than total count of correct classifications. Secondly, the training set needs to be composed of equal percentages drawn from each class, and not just a randomly selected 70% of the full dataset (since this would result in very few examples of `failure.b2` making it into the training set).

The [neural network article][ann-post] discussed these points in more detail with code samples. Here, I'll reuse the weighted accuracy function and the training set, so all classifiers here can be compared directly with the ANN and with each other.


## Technique 1: Rpart Decision Trees

A decision tree is like a flow chart of questions about the features, with answers leading to more specific questions, and finally to a classification. The process of building the tree is called _rule induction_, with rules being derived directly from the training set of data. 

Rules in decision trees are generally binary, so each will partition the data into two sets. A good rule will partition the data to minimise entropy, i.e. to maximise the certainty about which class the data belongs to. The next rule in the tree will apply only to the subset of data on that branch, which partitions the data further, and so on until enough questions have been asked to be certain about the classification.

Rpart is a decision tree implementation in R, and the [code to train an rpart tree is available here][github-rpart]. It's simple to use, since it has very few parameters. As with the ANN, I used different penalty weights for misclassifications, to help deal with the uneven numbers of datapoints in each class. I tested three weightings: 
 
 - `cw1`: all classes equally weighted; 
 - `cw2`: classes weighted by order using powers of 10;
 - `cw3`: classes weighted by 1/count of samples in the dataset.

[github-rpart]:     https://github.com/cowlet/data-science/blob/master/bearing_snippets/rpart.R

The best tree gave 90.3% accuracy, which was with `cw2` weightings (although there wasn't much difference between them). The tree itself is:

[![Visualisation of rpart tree](/assets/best-rpart-tree.png)](/assets/best-rpart-tree.png)

And the confusion matrix is:

<div class="r_output">
<table>
<thead>
<tr>
<th></th>
<th>early</th>
<th>failure.b2</th>
<th>failure.inner</th>
<th>failure.roller</th>
<th>normal</th>
<th>stage2</th>
<th>suspect</th>
</tr>
</thead>
<tbody>
<tr>
<th>early</th>
<td>285</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>5</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>failure.b2</th>
<td>0</td>
<td>11</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<th>failure.inner</th>
<td>0</td>
<td>0</td>
<td>12</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>failure.roller</th>
<td>0</td>
<td>3</td>
<td>0</td>
<td>161</td>
<td>2</td>
<td>4</td>
<td>13</td>
</tr>
<tr>
<th>normal</th>
<td>186</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>1084</td>
<td>0</td>
<td>34</td>
</tr>
<tr>
<th>stage2</th>
<td>0</td>
<td>0</td>
<td>0</td>
<td>1</td>
<td>1</td>
<td>94</td>
<td>0</td>
</tr>
<tr>
<th>suspect</th>
<td>14</td>
<td>2</td>
<td>0</td>
<td>90</td>
<td>66</td>
<td>17</td>
<td>506</td>
</tr>
</tbody>
</table>
</div>



## Technique 2: Support Vector Machines

Classification is hard because it's difficult to draw a clear boundary line between two classes. There is always some overlap between clusters of points belonging to separate classes, and it's generally the points around the boundary that are misclassified. One way of thinking about feature design is to find parameters which maximise the distance between classes, and which minimise the number of boundary points.

Support Vector Machines (SVMs) tackle this problem by increasing the dimensionality of the data. The reasoning is that if there's overlap between clusters in feature space, it should be easier to draw a line to separate the classes in a space of higher dimensionality. The dimensionality is increased using a kernel function, which is very often chosen to be a Radial Basis Function (RBF). 

In R, SVMs can be trained using the library `e1071`, and my [code for training is here][github-svm]. There are two parameters for an SVM using the RBF kernel: the cost $$C$$ of misclassifications, and the $$\gamma$$ parameter to the RBF. A simple strategy for finding the best values for these parameters is called grid search, where you set a range on each and train an SVM for every combination of values. By comparing accuracies, you can find good combinations, and set up a second grid search using finer grained steps.

[github-svm]:      https://github.com/cowlet/data-science/blob/master/bearing_snippets/svm.R

I ended up using three levels of grid search, the resulting test accuracies for which are shown below.

[![Accuracy map of SVM parameter pairs](/assets/svm-parameters.png)](/assets/svm-parameters.png)

The first sweep covered six values of $$\gamma$$ and four for $$C$$, with mostly poor accuracy. The second sweep tried seven values of $$\gamma$$ across a much tigher range, and five values for $$C$$. This included two $$C$$ values not previously tried, since the highest accuracy on sweep one was right on the edge of the tested space. Sure enough, higher values of $$C$$ gave even higher accuracy, so sweep three tried higher values again, giving the cluster of mostly red points towards the top of the plot. 

Since the highest accuracy is still on the edge of the tested space, there is the potential for increasing accuracy further with a fourth sweep. But SVM training is relatively computationally demanding and a sweep takes many hours, so I'll stick with the best model found so far.

As another parameter, I also tried varying the penalty weightings applied to misclassifications. I used `cw1`, `cw2`, and `cw3` from before, which gave the surprising result that `cw1` tended to perform best.

[![Effect of class weightings on SVM accuracy](/assets/svm-classweights.png)](/assets/svm-classweights.png)

Just in case the class weightings behaved in the opposite way for the `e1071` package than in other models, I also tried a `cw4` set, where the counts of samples were used as weights (i.e. giving a high weight to classes with lots of samples). This gave very much poorer results (black points above), confirming that the weightings worked as expected. For the best performing SVMs, giving equal weight to misclassifications genuinely works better than proportionally penalising the minority classes. 

The best SVM has an accuracy of 88.3%, with $$C = 500000$$ and $$\gamma = 0.003$$. The confusion matrix is:

<div class="r_output">
<table>
<thead>
<tr>
<th></th>
<th>early</th>
<th>failure.b2</th>
<th>failure.inner</th>
<th>failure.roller</th>
<th>normal</th>
<th>stage2</th>
<th>suspect</th>
</tr>
</thead>
<tbody>
<tr>
<th>early</th>
<td>263</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>25</td>
<td>0</td>
<td>2</td>
</tr>
<tr>
<th>failure.b2</th>
<td>0</td>
<td>10</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>2</td>
</tr>
<tr>
<th>failure.inner</th>
<td>0</td>
<td>0</td>
<td>12</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>0</td>
</tr>
<tr>
<th>failure.roller</th>
<td>1</td>
<td>0</td>
<td>0</td>
<td>133</td>
<td>3</td>
<td>9</td>
<td>37</td>
</tr>
<tr>
<th>normal</th>
<td>55</td>
<td>0</td>
<td>0</td>
<td>1</td>
<td>1195</td>
<td>0</td>
<td>53</td>
</tr>
<tr>
<th>stage2</th>
<td>0</td>
<td>0</td>
<td>0</td>
<td>9</td>
<td>0</td>
<td>87</td>
<td>0</td>
</tr>
<tr>
<th>suspect</th>
<td>0</td>
<td>0</td>
<td>0</td>
<td>24</td>
<td>53</td>
<td>0</td>
<td>618</td>
</tr>
</tbody>
</table>
</div>



## Technique 3: k-Nearest Neighbour

The final technique I'll use is k-nearest neighbour, which classifies a point based on the majority class of the nearest $$k$$ points in feature space. The library `kknn` also allows weighted k-nearest neighbour, where a kernel function is applied to weight the nearest $$k$$ datapoints. 

The two parameters for this are how many neighbouring points to take into account, $$k$$; and which kernel to weight with. The options I tried are:

 - $$k$$: 1, 3, 5, 10, 15, 20, 35, and 50
 - kernels: rectangular (unweighted), triangular, Epanechnikov, biweight, triweight, cosine, inv, Gaussian, rank, and optimal.

[Graphs of these kernels][kernels] give an idea of how neighbouring points are weighted. The [code for testing weighted k-nearest neighbour is here][github-knn].

[kernels]:      http://en.wikipedia.org/wiki/Kernel_(statistics)
[github-knn]:   https://github.com/cowlet/data-science/blob/master/bearing_snippets/knn.R

Since there is no iterative training of this technique, there is no way to incorporate penalty weightings for misclassifications. This has an interesting effect on the accuracy: as $$k$$ increases, absolute accuracy tends to increase, while weighted accuracy stays static or even decreases.

[![Effect of k on k-nearest neighbour accuracy](/assets/knn-accuracies.png)](/assets/knn-accuracies.png)


Weighted accuracy is the true reflection of the classifier's performance. It's an unfortunate by-product of how the technique works that the large numbers of points labelled normal and suspect will tend to dominate the nearest neighbour groups. You can see this most clearly in the weighted accuracy of the rectangular (unweighted) kernel: higher numbers of $$k$$ result in more normal points filling each neighbourhood, badly impacting accuracy.

The best accuracy was found to be 57.5%, with $$k = 35$$ and the triweight kernel (`w` in the graph). The confusion matrix is:

<div class="r_output">
<table>
<thead>
<tr>
<th></th>
<th>early</th>
<th>failure.b2</th>
<th>failure.inner</th>
<th>failure.roller</th>
<th>normal</th>
<th>stage2</th>
<th>suspect</th>
</tr>
</thead>
<tbody>
<tr>
<th>early</th>
<td>115</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>166</td>
<td>0</td>
<td>9</td>
</tr>
<tr>
<th>failure.b2</th>
<td>0</td>
<td>1</td>
<td>0</td>
<td>1</td>
<td>8</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<th>failure.inner</th>
<td>0</td>
<td>0</td>
<td>10</td>
<td>0</td>
<td>0</td>
<td>0</td>
<td>2</td>
</tr>
<tr>
<th>failure.roller</th>
<td>0</td>
<td>0</td>
<td>0</td>
<td>74</td>
<td>35</td>
<td>10</td>
<td>64</td>
</tr>
<tr>
<th>normal</th>
<td>48</td>
<td>0</td>
<td>0</td>
<td>2</td>
<td>1104</td>
<td>0</td>
<td>150</td>
</tr>
<tr>
<th>stage2</th>
<td>0</td>
<td>0</td>
<td>0</td>
<td>2</td>
<td>9</td>
<td>85</td>
<td>0</td>
</tr>
<tr>
<th>suspect</th>
<td>3</td>
<td>0</td>
<td>0</td>
<td>11</td>
<td>274</td>
<td>5</td>
<td>402</td>
</tr>
</tbody>
</table>
</div>



## Conclusions

Now I have four classifiers which each operate in different ways. The ANN finds feature weightings which distinguish one class from another. The SVM uses RBF functions to fit the non-linear function which separates classes. The decision tree produces rules to partition the data using entropy, considering one feature at a time. K-nearest neighbour assumes clusters of points are meaningful, and only considers data within the same region. 

By fairly extensively exercising the parameterisation options for these approaches, I have classifiers with accuracies 94.0% (ANN), 88.3% (SVM), 90.3% (rpart), and 57.5% (k-NN).

Since the ANN is best, it's natural to assume that 94.0% accuracy is as good as we can get on this problem. However, it's possible to combine groups of classifiers in such a way that their overall accuracy is higher than the best individual within the group. Even the relatively poor 57.5% classifier isn't useless! Since it behaves differently from the others, it could complement the ensemble by correctly classifying points that the others misclassify.

The choice of technique for combining different classifiers was my thesis topic, and there is [an open access paper on evidence combination for the application of power transformer defect diagnosis][journal1].

[journal1]:     http://strathprints.strath.ac.uk/11785/



