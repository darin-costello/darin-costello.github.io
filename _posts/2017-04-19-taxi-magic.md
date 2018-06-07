---
layout: post
title: "Natural Nodes: Finding Patterns in NYC Taxi trips"
date: 2017-04-19
categories: taxi datamining clustering unsupervised
math: true
---

Introduction
=============

For my undergraduate big data capstone project [Spencer Galbraith][spence-gal], [Jonghyun Choi][jon-choi], and I worked on a project for Lawrence Livermore National Laboratory. Given a cleaned version of the [New York City taxi trip data][nyc-data] our goal was to find "natural" nodes, such that global anomalous behavior can be detected by observing the flow between nodes. I focused on created the nodes, while Spencer and Jon focused on detecting the anomalies.

Feel free to skip to the [results](#Results).

Data
=======

There was an enormous amount of data. It spans over 8 years and there are roughly 500,000 trips per year, even after removing ones outside of our target area. Each trip has a multitude of fields, including pickup and drop off coordinates, trip cost, driver number, and a host of location data. We were working on standard laptops. Thus we decided to work with a small subset, January 2012, and focus solely on the pickup coordinates. The coordinates were in latitude and longitude, however the geographical area we were focusing on was relatively small, thus we scaled the coordinates to mean zero and a deviation of one. This allows us to use a standard Euclidean distances instead of a great circle distance.

Clustering
===========

A good clustering is one that allowed us to detect anomalous behavior, but we had neither a list of anomalies or any way of detecting them in a graph. So, as a first pass approximation, we decided to eyeball it. 

We didn't know how many final clusters there would be we decided to go with a density based approach. We first went with DBSCAN. In a nutshell, the user gives DBSCAN the points, and the density required for a cluster. The density is specified by an $$\epsilon$$ which specifies how far away to look for other points, and a minPts parameter which specifies the number of points required to be a cluster. It finds all points with minPts within $$\epsilon$$ from it, and merges points into clusters if they have common minPts. 

 ![first-pass]
 
 This was our first approach, where the blue circles show cluster centers. However, we were dissatisfied with the results. No matter how much we fiddled with hyper-parameters, either Manhattan was a single cluster or all the other points where set as noise.

We decided to move to a more sophisticated method, OPTICS. OPTICS, in essence, runs DBSCAN for all $$\epsilon$$ values, after specifying minPTS. This creates a hierarchy structure which allows for clusters of different densities to be produces. This is done by specifying a cut off value, and flattening the hierarchy where each point is labeled the as the most specific. 
![optics1] 

This is an example, see how the blue cluster is spread throughout the entire area.
So while it allowed for differing densities, the edges of each boundary are less defined.
We left it as it was but it ended up not being an issue. 

Both methods run in $$O(n\log n)$$ time, so running OPTICS on the entire month was too much for our laptops to handle. our laptops could handle just about a single days worth of trips. However, this gave us 31 different clustering, and no way to tell which was best.

Comparing and Combining Clusters
=================================

The first step to comparing clusters is finding an appropriate distance metric. The first I tried was known as Variation of Information. Given two clustering $$X$$ and $$Y$$  of some data set, it is defined as $$VI(X;Y) = H(X) + H(Y) - 2I(X,Y)$$ where $$H(X)$$ is the entropy of X and $I(X,Y)$$ is the mutual information between X and Y. In English it measures the overlap of the clustering, regardless of what label the points are given. Thus a lower variation of information is better. However, we have a bunch of clustering on different data sets. To fix this is created a sampled data set, and had each clustering by day predict on it. Then I compared thees directly. Using MDS to plot gives

![visamp]

Included is the clustering on the sampled cluster. Interestingly the right group consists of weekends,while the right are the week days. The ideal cluster would minimize be a clustering that minimizes average distance between itself and all other clusters. Finding this cluster is hard, so I resorted to basic optimization techniques. For simplicity, I used a measure called Normalized Mutual Information. NMI measures the same thing as VI, but is normalized so its easier to compare and combine measurements.  I looked at using a basic greed approach, one that changes the cluster of a single point and if the NMI increases we keep it. I also tried simulated annealing. 

As a proof of concept I created some 2 slightly different data sets, and clustered them using OPTICS, as shown below.

![sepT] ![connT]

The ideal pre-labeled looks like this:

![labeled](/images/taxiCapStone/combineTests/ideal.png){: width="50%"}

Below are four runs of the optimization algorithm.

![combined1] ![combined2]

![combined3] ![combined4]

Though combining clusters worked, and looked good. The process took a long time with all 31 days. So we just eyeballed it, and using the graph above chose day 24 as our clustering, as it was fairly centered.

Results {#Results}
=============

The final clustering looked like this:

![finalClust](/images/taxiCapStone/NewYork.png)

Note that for this plot only each node is a cluster, not color. There are thousands of nodes. If we zoom in at known location we can see the nodes make sense. Looking at Grand Central Station there are three nodes, one for each entrance.

![gcs](/images/taxiCapStone/gcs.png)

The same goes for Madison Square Garden.

![msq](/images/taxiCapStone/msg.png)

We were also able to capture a less dense clusters like those found around John F. Kennedy Airport.

![jfk](/images/taxiCapStone/jfk.png)

Anomaly detection was then accomplished by running a linear regression over each nodes traffic. Thus when traffic outside the predicted amount we could identify anomalies. Our post can be found [here](https://docs.google.com/presentation/d/1KfCfBBXUFwhRZ5SGga4HfMZiBlg5LI5hUQcjfMm8UQA/edit?usp=sharing).


[spence-gal]: https://www.linkedin.com/in/galbraithsn/
[jon-choi]: https://www.linkedin.com/in/jonghyun-choi/
[nyc-data]: http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml

[first-pass]:/images/taxiCapStone/firstPass.png


[optics1]: /images/taxiCapStone/jfkoptics1.png
[visamp]: /images/taxiCapStone/visamp.png

[sepT]: /images/taxiCapStone/combineTests/Seperated.png
{: width="45%"}
[connT]: /images/taxiCapStone/combineTests/Connected.png
{: width="45%"}

[combined1]: /images/taxiCapStone/combineTests/Combined_2.png
{: width="45%"}
[combined2]: /images/taxiCapStone/combineTests/start_with_ideal.png
{: width="45%"}
[combined3]: /images/taxiCapStone/combineTests/four_clusters.png
{: width="45%"}
[combined4]: /images/taxiCapStone/combineTests/six_clusters.png
{: width="45%"}