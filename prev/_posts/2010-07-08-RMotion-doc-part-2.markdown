---
layout: post
title: RMotion doc part 2
---
Second part of RMotion documentation.
Motion objects methods `vid` and `cam` yield information about moving objects to the passed block.
    m=Motion.new
    m.cam do |e| 
      ...
    end 

Those informations are organized as an array of `Entity` instances. So far up to 8 different objects can be recognized. Entities array entries are ordered by size, so the bigger-faster moving one will be first. Entities methods are `center` and `corners`

    m=Motion.new
    m.cam do |e| 
      p e.first.center
      p e.first.corners
    end 

and that's the output produced, in the form

    [:x,:y]
    [:topleftx,:toplefty,:bottomrightx,:bottomrighty]

    nil
    [nil, nil, nil, nil]
    [416, 68]
    [320, 0, 512, 136]
    [356, 36]
    [320, 0, 392, 72]
    ...

To track current entities could be useful to keep each entity at the same array index even when others disappear, so that's the meaning of class method `reorder`.
`Entity.status` returns an initialized empty array, so it can be used to store last frame entity information.
`Entity.reorder` takes the reference array and the new generated one and places everything where it should be.

    m=Motion.new
    status=Entity.status
    m.cam do |e| 
      Entity.reorder status, e
      status.each_with_index do |s,n|
        puts "Entity #{n} is in position #{s.center.inspect}" unless s.center.nil?
      end
    end 

`vid` can read a video file and acts like cam. To quite the loop you can execute `m.quit` or hit b on you keyboard, unless you previously set `m.show=false`.

    m=Motion.new
    i=0
    m.vid("capture.avi") do |e|
      m.quit if i > 100
      i+=1
    end 

I'm posting the gem to github and rubygems as soon as possible.
