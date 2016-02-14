---
layout: post
title: Vehicle classification
---
I improved my ruby motion recognition library, [RMotion](https://github.com/rikiji/rmotion]), to provide object classification features.
Currently it has been tested on vehicles, expecially on cars shapes. In the following demo video it is configured to recognize cars (red label) and filter out everything else (blue label). Cars which are within the camera only by an half are not classified as matching, and that's the intended behaviour since their shape isn't looking like a car one.

{% youtube 1wxg4nUQ_DA %}

Video is deliberately slowed down, since here in front of my window people drive insanely fast! As usual i'll write some documentation and usage examples and post them here.
