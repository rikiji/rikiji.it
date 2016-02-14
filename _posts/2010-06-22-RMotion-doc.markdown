---
layout: post
title: RMotion doc
---
RMotion provides a simple interface to build motion detection software in ruby.
At the moment it's still under test, i need to try it with different webcams before releasing it. If you have a webcam that works on linux and would like to try it now just drop me an email!

[Last time](http://www.rikiji.it/post/14) i showed up with a demo of car motion detection with some noise and green cells to mark movements. Now RMotion is much more configurable, both noise selection and output marking are customizable. This is how RMotion runs with default options: http://www.youtube.com/watch?v=HNL-2pNkuSo

Now let's see in __irb__ some configuration options. Require the lib, instantiate the main object and print default options:

    irb(main):001:0> require 'rmotion'
    => true
    irb(main):002:0> m=Motion.new
    => #<Motion:0xb74d14a0>
    irb(main):003:0> m.show?
    => true
    irb(main):004:0> m.write?
    => false
    irb(main):005:0> m.fft?
    => true

`show`, `write` and `fft` are the most important options. When `show` is true the video analyzed is also displayed in a window. You can set `write` to a string (a filename) to save the video analyzed to a file! Both displayed and saved video will have motion detection markings on it, like the video i just showed to you.

`fft` manages how the video frames are processed (fast fourier transform or direct): i recommend to set it always true, except in case your cpu isn't fast enough to follow the stream in realtime. Soon i'll post an in-depth analysis of what's the difference between `fft=true` and `fft=false`.

    irb(main):006:0> m.fill?
    => true
    irb(main):007:0> m.rect?
    => false
    irb(main):008:0> m.point?
    => false

Next group of options is about output frames: `fill` default true behaves like the previous video, `rect` draws rectangles around moving objects and `point` draws instead a small circle centered on the object. Let's see them! Note that i can swap them live!
http://www.youtube.com/watch?v=0LQIt0XiWqE

    irb(main):012:0> m.threshold_fft?
    => 1.0
    irb(main):013:0> m.threshold_direct?
    => 8.0
    irb(main):014:0> m.threshold_distance?
    => 9.0
    irb(main):015:0> m.threshold_group?
    => 20

`threshold_fft` and `threshold_direct` values influence what is recognized as noise and what as an object. Note that they are used in a mutually exclusive way: if `fft` is true then `threshold_direct` won't affect result. Same for `fft` as false and `threshold_fft`. Play with those value to fit your enviroment (light, object speed..).

`threshold_distance` and `threshold_group` also have a role on recognizing objects: an object is a group of al least __group__ cells distant each other no more than __distance__. So you can choose min size of objects and spread of cells. Cells are a group of pixel determined dynamically on camera resolution, so don't worry about it.

Next time i'll talk about capture loop and Entities.
