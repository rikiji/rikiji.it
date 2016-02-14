---
layout: post
title: Tor consensus nse parser
---
Today I quickly translated a python script i had floating around into a more useful [NSE script](http://nmap.org/nsedoc/).

It pulls the consensus data from one of the 9 Tor directory servers ([documentation here](https://gitweb.torproject.org/tor.git/tree/HEAD:/doc)) and runs a regular expression to extract the ip addresses of the nodes until it finds a matching one.

    nmap -p0 -dd -Pn --datadir=. --script=tor-consensus-checker 86.59.11.2
    ...
    NSE: Starting 'tor-consensus-checker' (thread: 0x9a87568) against 86.59.11.2.
    Initiating NSE at 23:36
    NSE: checking if 86.59.11.2 is a tor relay
    NSE: Final http cache size (674972 bytes) of max size of 1000000
    NSE: consensus retrieved from 128.31.0.39
    NSE: Finished 'tor-consensus-checker' (thread: 0x9a87568) against 86.59.11.2.
    PORT  STATE  SERVICE REASON
    0/tcp closed unknown conn-refused
    
    Host script results:
    | tor-consensus-checker: 
    |_  86.59.11.2 is a tor node
    
    Nmap done: 1 IP address (1 host up) scanned in 1.42 seconds

The script can currently be found on my [github repository](https://github.com/rikiji/nmap-scripts) of nmap scripts.
