---
layout: post
title: "Robot Localization Through Color Blob Detection"
date: 2017-06-13
categories: robotics localization sphero colorblob ros python
thumbnail: /images/colorBlob/7.png
---

Problem
=============

As part of the Mother Duck demo I needed a better way to localize the Spheros. The only sensor the Spheros have is an imu based odometer, which has two issues. First, its not very accurate. Second, each sphero acts in its own coordinate system, that is the odometer reads only its displacement from its original location, this data isn't enough to project the swarm into a common plane.

The old system of using AprilTags (a VR taglike system) worked well, except it was difficult to attach flat tags to rolling robots, and the cup method is prone to failure, as seen in my AI [final project video](https://www.youtube.com/watch?v=WF05LLP_99U&t=69s). Also, it was computationally expensive. 

Solution
========

There is a technique in image recognition called color blog detection. It simply finds continuous pixels all of the same or similar color or intensity. This method runs into issues when the scene is varied, that is many none sphero color blobs are detected when brightly lit. Luckily the Spheros are a light source, thus if we set the camera exposure low all but the Spheros become black. The process looks like this:

{% highlight python %}
image = cam.read()
hsv_image = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
#creates a black and white image makes it easier to find blobs.
mask = cv2.inRange(hsv_image, lb, up)
_, blob_list, _ = cv2.findContours(mask, ...)
for blob in blob_list:
    #find color of blob
    color = cv2.mean(hsv_image, mask)

    #find center location of blob
    moment = cv2.moments(blob)
    x = moment["m10"] / moment["m00"]
    y = moment["m01"] / moment["m00"]

{% endhighlight %}

 Finally, each Sphero needs to be individually identifiable. This is done by assigning each sphero a color at startup, spreading the colors evenly across the spectrum. To adjust for camera variation, the Spheros set themselves so they appear the correct color to the camera. The follow code is the startup process, stripped of boilerplate and a simpler adjustment algorithm.
 
 {% highlight python %}
 hue_max = 179.0
 goal_hue = 0
 hue_step = hue_max / len(sphero_list)

 for sphero in sphero_list:
    sphero.turn_off_light()

for sphero in sphero_list:
    sphero.set_color(goal_hue)
    seen_hue = blob_detector.get_color()
    while seen_hue != goal_hue:
        error = goal_hue - seen_hue
        sphero.set_color(goal_hue - error )
        seen_hue = blob_detector.get_color()

{% endhighlight %}

The startup process from the point of view of the camera is shown here.

<iframe width="560" height="315" src="https://www.youtube.com/embed/fuwC1P1KDzo" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

The color blob tracker is a ros package that runs the web cam, finds all blobs in each frame, and publishes there locations and colors. Other nodes deal with adjusting the colors, and translating colors to spheros. In testing this system could uniquely identify about 11 spheros with no mistakes. It worked with more but occasionally would mix them up.

The final ros packages can be downloaded [here](/data/color_blob.zip)
