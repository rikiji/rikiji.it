---
layout: post
title: RMotion with additional neural networks
---
Last year I developed RMotion, a library with __ruby__ bindings that can be used to detect moving objects in a video/cam stream in real time.  It is quite convenient to solve simple detections problems because you can just program the logic in __ruby__ inside the main loop and you get most of the detection issues solved for free by the library.

I subsequentely added classifying features: as you can see in the [previous video](http://www.youtube.com/watch?v=1wxg4nUQ_DA&feature=g-upl) or in this one below, it is now possible to train a classifier with a desired shape and then receive in __ruby__ the type of the moving entity, according to an accuracy threshold specified.

The improvement from my last post is the classification of multiple classes of objects instead of a single one, and the removal of the edge effect that occurred when the object was not fully in view. The video is slowed down at approx. 30% to make the labels visible for humans!

{% youtube IsNLzhu_BKM %}

Unfortunately the manual work for the user increases linearly with the number of classes to detect. Currently i'm using multiple neural networks ( __supervised learning__ ) as a classifier, and human intervention is necessary to decide which shapes belong to a class and which are noise before training the net. After that the shapes are generated, the user places them in appropriately labeled directories and then a training script does everything automatically:

    rkj@harpoon:~/code/rmotion/v3$ ruby net/car.net sets/car/
    Training
    count: 00000, terror: 1.26979
    count: 00010, terror: 0.02167
    count: 00020, terror: 0.00835
    ...
    count: 02530, terror: 0.00004
    count: 02540, terror: 0.00004
    Testing
    EXPECTED: true
    0.941322135603077 true
    EXPECTED: false
    0.02227733085452 false
    rkj@harpoon:~/code/rmotion/v3$ ruby net/bike.net sets/bike/
    Training
    count: 00000, terror: 1.46554
    count: 00010, terror: 0.01378
    ...


The idea to mitigate this effort is to use __unsupervised learning__ to classify the different classes, so that if the classifier learns how a high number of objects are shaped it will be eventually able to partition them in groups, and the job of the user will be just to decide within how many groups the objects are going to be split. I didn't decide yet the nature of the unsupervised classifier, I'll run some tests as soon as i can get some good quality traffic footage.

The second thing that i'd like to add is a stronger object tracking algorithm, so that if the object stops or its path overlaps with another one it would be possible to track it anyway. I can implement this using some kind of __feature detection__ other than movement or i can record the position and the full path on screen of the object and trace back the origin of disappearing/reappearing objects. The first approach sound more appealing to me, so the next step will be in that direction.
