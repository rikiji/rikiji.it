---
layout: post
title: EAP authentication scan
---
I've been working on a [NSE script](http://nmap.org/nsedoc/) which enumerates the authentication methods available on an eap authenticator, especially useful when those methods that support 2-step authentication with an anonymous identity in the first step are offered.

The principle is straightforward, for a group of auth methods of interest the script will follow the eap handshake sequence multiple times keeping track of the methods offered:

1. sending eap start packet to the broadcast mac address `01:80:c2:00:00:03`
2. responding to the identity request issued by the ap
3. parsing the auth request to find the auth method offered, then responding with a nak packet that requests another method
4. If auth request: `5`, if instead failure packet is received: `6`
5. respond with another nak and another auth method, then again `4`
6. auth method not available. restart from state `1` unless already tested every protocol of interest.

A simple session is given below (relying on my memory, hope to remember everything correctly):

    Eap start
    -- -- -- --
    01 01 00 00
    
    Eap identity request
    -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
    01 00 00 0A 01 EF 00 0A 01 68 65 6C 6C 6F
    
    Eap identity response
    -- -- -- -- -- -- -- -- -- -- -- -- --  
    01 00 00 09 02 EF 00 09 01 75 75 65 72 
    
    EAP PEAP authentication request
    -- -- -- -- -- -- -- -- --  
    01 00 LL LL 01 CD LL LL 19  ... more stuff (LL = length)
    
    EAP NAK response, TTLS requested
    -- -- -- -- -- -- -- -- -- --
    01 00 00 05 02 CD 00 05 03 15

The script currently enumerates successfully the auth methods when tested with hostapd v0.6.10, it's on my [github](https://github.com/rikiji/nmap-scripts) as well as in the nmap main trunk.

    nmap -e eth2 -sn --script=eap-info --datadir=. localhost
    Pre-scan script results:
    | eap-info:
    | Available authentication methods with identity="anonymous":
    |   true     PEAP
    |   true     EAP-TTLS
    |   false    EAP-TLS
    |_  false    EAP-MSCHAP-V2
