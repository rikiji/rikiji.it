---
layout: post
title: Japanese learning tool with HTML5 interface
---
I always find useful those language learning tools that are aimed to train a single specific skill among those needed to master a foreign language (one tool one task).
I was looking for something that could help me to build some knowledge about daily sentences, both in meaning and in pronunciation, showing kanjis while providing also them as kana. Like i did with __playkanji.com__ i decided to build an user interface in JavaScript to fulfill the need: [teacher.playkanji.com](http://teacher.playkanji.com).

Japanese suits very well to software development since it isn't really necessary to have your program translate a sentence (very hard) to help the user with additional info. [Chasen](http://chasen.naist.jp/hiki/ChaSen/) provides some help in parsing sentence structure while [Edict](http://ftp.monash.edu.au/pub/nihongo/00INDEX.html) is the most famous digital japanese dictionary.

My web application is really __HTML5__ dependent, it makes use of some __local storage__ functions as well as __audio__ tags. You are supposed to read and listen the sentence clicking on the `play ♪` button, the if necessary some info (meaning and pronunciation) of displayed kanjis is provided clicking on `kanji help` (`help 漢字`). While typing your translation in the grey box it will be parsed and evaluated in real time. It will be compared to the reference translation, and matching word will be highlighted in blue. For those interested in the internals of it I'm using the [Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance]) to highlight your words in a blue shade getting darker while they are getting closer to the correct word.
One limitation of my batch of data is that dictionary references that you will find clicking on `help` won't exatcly match the reference translation, since it would require large amount of manual work to fix. Consider them as a hint and help yourself with some synonyms while messing with Levenshtein output.

__HTML5__ local storage capabilities let me save the study history within your browser, so there's no need for login, password and so on: just fire up [teacher.playkanji.com](http://teacher.playkanji.com) and enjoy.
Clicking `clear this result` and `clear all history` in the right upper side of the page will lead you to reset the current result and the whole history respectively.

Drop me a line if you spot a bug or a bad translation, the app has not been tested thoroughly and i'm still drilling through sentences. Currently supports only __Firefox__ and __Chrome__, don't even bother trying on IE. Opera and safari untested. 
