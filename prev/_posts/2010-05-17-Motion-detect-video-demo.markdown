---
layout: post
title: Motion detect video demo
---
I'm working on a basic although rather complete framework to recognize moving objects on webcam video streams. You can enjoy a demo on this [youtube video](http://www.youtube.com/watch?v=_MVFwVuGeyg).

Today it's windy so some green squares appear also above trees and brushes! Cars and walking people are recognized and tracked from one side to the other of the viewport. 
Many kind of informations about moving entities will be available to applications, and the algorithm is heavily customizable since different cameras record different levels of noise.. At the moment I'm focusing on speed to reach acceptable fps even on slow machines, then i just have to package it as a library, provide some documentation and post it here.	
