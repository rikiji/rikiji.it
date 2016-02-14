---
layout: post
title: rediswrap
---
I'll dump here some info about a [library](https://github.com/rikiji/rediswrap/) i started writing some time ago or i'll end up forgetting it. This is a straightforward c++ wrapper of libhiredis, I know that there are already dozens of c++ Redis libraries out there but i needed something natively compatible with the standard c++ vectors and strings, therefore this library.

I already implemented Redis operations for `strings`, `keys`, `lists` and `sets`, so it's already quite usable. `hashes` and server management calls are not there yet. The usage is designed to be as simple as possible, the following is a snippet taken from the regression tests:

    rediswrap::Redis r;
    r.set("foo","100");
    assert(r.decr("foo") == 99);
    assert(r.decrby("foo",6) == 93);
    
    vector<string> vals;
    vals.push_back("aaa");
    vals.push_back("bbb");
    assert(r.lpushm("foo",vals) == 3);


The source code is up on [github](https://github.com/rikiji/rediswrap/).
