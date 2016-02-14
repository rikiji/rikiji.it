---
layout: post
title: ruby-memscan 0.01
---
First version of ruby-memscan (my ruby memory scanning API) completed and [pushed to github](https://github.com/rikiji/ruby-memscan)!
Features heap, stack, bss and data dumping into arrays of ulong values and  basic (long) search at the moment.

Sample usage is found inside __test__ subdirectory:`make` and run `./test` then
    ruby test.rb $(pidof test)

Here follows [test.rb](https://github.com/rikiji/ruby-memscan/blob/master/test/test.rb):

    require 'memscan'
    require 'pp'
    
    m= Memscan.new
    
    pid= ARGV[0].to_i
    m.attach pid
    
    puts m.dump_stack.size
    puts  m.dump_heap.size
    
    pp m.search_long 3735928559
    pp m.search_long 3405691582

When run against test.c it will print at least two positive matches.
