---
layout: post
title: OpenCV gem patch
---
I had to fix some lines of the gem source code in order to build it correctly on my debian machine.

    iplimage.cpp: In function ‘VALUE mOpenCV::cIplImage::rb_initialize(int, VALUE*, VALUE)’:
    iplimage.cpp:91: error: ‘cvCvToIplDepth’ was not declared in this scope
    iplimage.cpp: In function ‘VALUE mOpenCV::cIplImage::new_object(int, int, int)’:
    iplimage.cpp:228: error: ‘cvCvToIplDepth’ was not declared in this scope
    iplimage.cpp: In function ‘VALUE mOpenCV::cIplImage::new_object(CvSize, int)’:
    iplimage.cpp:234: error: ‘cvCvToIplDepth’ was not declared in this scope
    make: *** [iplimage.o] Error 1

Here's the fix

    --- iplimage.cpp	1970-01-01 01:00:00.000000000 +0100 
    +++ _iplimage.cpp	2010-04-16 09:59:23.000000000 +0200 
    @@ -88,7 +88,7 @@ 
    rb_scan_args(argc, argv, "22", &width, &height, &depth, &channel); 
    int _depth = argc < 3 ? CV_8U : FIX2INT(depth); 
    int _channel = argc < 4 ? 3 : FIX2INT(channel); 
    - DATA_PTR(self) = cvCreateImage(cvSize(FIX2INT(width), FIX2INT(height)), cvCvToIplDepth(_depth), _channel); 
    + DATA_PTR(self) = cvCreateImage(cvSize(FIX2INT(width), FIX2INT(height)), cvIplDepth(_depth), _channel); 
    return self; 
    } 
    
    @@ -225,13 +225,13 @@ 
    VALUE 
    new_object(int width, int height, int type) 
    { 
    - return OPENCV_OBJECT(rb_klass, cvCreateImage(cvSize(width, height), cvCvToIplDepth(type), CV_MAT_CN(type))); 
    + return OPENCV_OBJECT(rb_klass, cvCreateImage(cvSize(width, height), cvIplDepth(type), CV_MAT_CN(type))); 
    } 
    
    VALUE 
    new_object(CvSize size, int type) 
    { 
    - return OPENCV_OBJECT(rb_klass, cvCreateImage(size, cvCvToIplDepth(type), CV_MAT_CN(type))); 
    + return OPENCV_OBJECT(rb_klass, cvCreateImage(size, cvIplDepth(type), CV_MAT_CN(type))); 
    } 
    
    __NAMESPACE_END_IPLIMAGE 

then just run as usual

    ruby extconf.rb 
    make
    
and place opencv.so in your ruby lib path. You will need libffcall1-dev to build the GUI module, otherwise opencv will build cleanly but without gui support.
