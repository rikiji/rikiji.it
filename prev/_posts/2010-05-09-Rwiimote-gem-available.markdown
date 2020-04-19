---
layout: post
title: Rwiimote gem available
---
I just pushed to [rubygems](http://rubygems.org]) my first gem!! It's called [rwiimote](http://rubygems.org/gems/rwiimote), it's a wrapper around __libcwiimote3__, which provides some bindings to control wiimotes over bluetooth. To install:

    gem install rwiimote
Included within the gem there's a `README` that provides some documentation, along with some usage examples. Let's analyze them here:

    w=Wiimote.new
    
    # w.connect requires mac address of device to connect to. find it by running "hcitool scan" just after you hit 1&2 on your wiimote
    
    puts "press both 1 and 2 keys to connect"
    exit unless w.connect "00:11:22:33:44:55"
    puts "connected!"
    
    # you can turn those fancy blue leds on or off by calling on/off methods on them 
    w.led[1].on
    w.led[2].off
    
    puts "click home to disconnect"
    while w.open?
      # w.update has to be called every time you want to send/receive new data to/from the device. usually it's at the beginning of a loop like this 
      w.disconnect unless w.update
      
      # check if button "a" was pressed
      puts "a!" if w.a?
    
      # disconnect if button "home" was pressed
      w.disconnect if w.home?
    end
    
The second one shows how to access the force data:

    w=Wiimote.new
    exit unless w.connect "00:11:22:33:44:55"
    
    # turn on accelerometer
    w.acc= true
    
    puts "shake to disconnect!!"
    while w.open?
      w.disconnect unless w.update
    
      #	print information about each axis of the accelerometer
      puts "force x: " + w.force_x.to_s
      puts "force y: " + w.force_y.to_s
      puts "force z: " + w.force_z.to_s
    
      # disconnect if you were shaking too hard :)
      w.disconnect if w.force > 2
    end
    
There are some other methods available, check them in the `README` file. Drop me a line if something does not work or if you just have some questions about it.
