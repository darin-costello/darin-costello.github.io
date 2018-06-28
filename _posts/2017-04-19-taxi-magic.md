---
layout: post
title: "Natural Nodes: Finding Patterns in NYC Taxi trips"
date: 2017-04-19
categories: taxi datamining clustering unsupervised
math: true
thumbnail: /images/taxiCapStone/thumbnail.png
---

Introduction
=============

For my undergraduate big data capstone project [Spencer Galbraith][spence-gal], [Jonghyun Choi][jon-choi], and I worked on a project for Lawrence Livermore National Laboratory. Given a cleaned version of the [New York City taxi trip data][nyc-data] our goal was to find "natural" nodes, such that anomalous behavior could be detected by observing the flow of traffic between nodes. I focused on created the nodes, while Spencer and Jon focused on detecting the anomalies.

Feel free to skip to the [results](#Results).

Data
=======

The data spanned over 8 years and with roughly 500,000 trips per day after removing ones outside of our target area. Each trip has a multitude of fields, including pickup and drop off coordinates, trip cost, driver number, and a host of location data. We were working on standard laptops, so we decided to use only a small subset of the data, pickup coordinates during January 2012. The coordinates were in latitude and longitude, however the geographical area we were focusing on was relatively small so the curvature of the earth wasn't a problem. All distances where measured in Euclidean, after  we scaled the coordinates to mean zero and a deviation of one.

Clustering
===========

A good clustering is one that allowed us to detect anomalous behavior, but we had neither a list of anomalies or any way of detecting them in a graph. So, as a first pass approximation we eyeballed it. Looking for clusters that made 'sense'.

We didn't know how many final clusters there should be, and a node should be a place with a high concentration traffic. Thus we went with a density based clustering algorithms. We used the algorithm DBSCAN as our first approach. In a nutshell, the user gives DBSCAN the points, and the density required for a cluster. DBSCAN then gives back areas all with approximately that density of points. In our case it gave back this map:

 ![first-pass]
 
The blue circles show cluster centers, the bigger the circle the more points contained in that cluster. However, we were dissatisfied with the results. No matter how much we fiddled with hyper-parameters, either Manhattan was a single cluster or all the other points where set as noise.

We decided to move to a more sophisticated method, OPTICS. OPTICS, in essence, runs DBSCAN for a bunch of different density values. The different clusterings can then be layered, to create a hierarchy, resulting in clusters of different densities.

![optics1] 

This is an example showing OPTICS run on JFK airport. Se how the blue cluster is spread throughout the entire area.
So while it allowed for differing densities, the edges of each boundary are less defined.
It still worked better then anything else, so we kept it as is, deciding to deal with that problem later.

Both methods run in $$O(n\log n)$$ time, so running OPTICS on the entire month was too much for our laptops to handle. So instead we ran OPTICS for each day of our target month, giving us 31 different clusterings. We then had to combine them somehow, or choose one that was best.  

Comparing and Combining Clusters
=================================

The first step to comparing clusters is finding an appropriate distance metric.The most natural is a measure called Variation of Information. Given two clusterings $$X$$ and $$Y$$ of some common data set, variation of information is defined as $$VI(X;Y) = H(X) + H(Y) - 2I(X,Y)$$ where $$H(X)$$ is the entropy of $$X$$ and $$I(X,Y)$$ is the mutual information between $$X$$ and $$Y$$. In English it measures the unique information of each clusterings, or how much they don't overlap. Thus, the lower the variation of information the more similar $$X$$ and $$Y$$ are. However, we have a bunch of clustering on different data sets. To fix this is created a sampled data set, and had each clustering on it, which where compared directly. With a similarity metric, we can then plot all the clusterings, giving us an idea of how different they are.

![visamp]

Interestingly the right group consists of weekends, while the left are the weekdays. The ideal cluster would be a clustering that minimizes average distance between itself and all other clusters. Finding this cluster is hard, so I resorted to basic optimization techniques. For simplicity, I used a measure called Normalized Mutual Information. NMI measures the same thing as VI, but is normalized so its easier to compare and combine measurements.  I looked at using a basic greedy approach, one that changes the cluster of a single point and if the NMI improved the change was kept, as well as simulated annealing. 

As a proof of concept I created some 2 slightly different data sets, and clustered them using OPTICS, as shown below.

![sepT] ![connT]

The ideal pre-labeled looks like this:

![labeled](/images/taxiCapStone/combineTests/ideal.png){: width="50%"}

Below are four runs of the optimization algorithm.

![combined1] ![combined2]

![combined3] ![combined4]

Though combining clusters worked, and looked good. The process took a long time with all 31 days. To move on we chose to use day 24's clusterings, as it was pretty a pretty typical day.

Results {#Results}
=============

The final clustering looked like this:

![finalClust](/images/taxiCapStone/NewYork.png)

Note that for this plot each point is a cluster, not color. If we zoom in at known locations we can see the nodes make sense. Looking at Grand Central Station there are three nodes, one for each entrance.

![gcs](/images/taxiCapStone/gcs.png)

The same goes for Madison Square Garden.

![msq](/images/taxiCapStone/msg.png)

We were also able to capture a less dense clusters like those found around John F. Kennedy Airport.

![jfk](/images/taxiCapStone/jfk.png)

Anomaly detection was then accomplished by running a linear regression over each nodes traffic. Thus when traffic surged or dropped outside the predicted amount we could identify anomalies. Our poster can be found [here](https://docs.google.com/presentation/d/1KfCfBBXUFwhRZ5SGga4HfMZiBlg5LI5hUQcjfMm8UQA/edit?usp=sharing).


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