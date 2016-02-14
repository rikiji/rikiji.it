---
layout: post
title: Redis Android NDK port
---
I just finished the first step of porting [Redis](http://redis.io), the server component, to the Android ndk. I didn't test anything besides basic features, value store/load, db save and db restore but the few things i tested worked perfectly. To port it i created a stub app and compiled the source tree as a __JNI__ library. I brought in __lua__ too since it is a dependency needed by Redis for server-side scripting.

I added in the source code  `#ifdef __ANDROID__` macros where I introduced some fixes to compile successfully, mostly related to compatibility with the bionic c library on Android. The entry point in `redis.c` has been substituted by a __JNI__ call and the log function hooked  to __Logcat__ for convenient debugging. The db is stored on the sdcard in `/sdcard/redis/`.

The stub app has a button to start/stop Redis as an Android service and another one to test if it answers correctly to set/get calls. It is up [here on github](https://github.com/rikiji/redis-android). I use [Jedis](https://github.com/xetorthio/jedis) as a client library, the next step will be importing the Jedis test suite to fix what doesn't behave in the right way and add an option to read the server configuration file from the sdcard. 
>  [Fork it on github](https://github.com/rikiji/redis-android)
