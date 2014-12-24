---
layout: post
title: "Targeting nodejs with clojurescript"
categories: clojurescript
---

I recently read David Nolen's post on how he uses a [browserless clojurescript] [bcljs]
workflow.  I decided use clojurescript to write a small tool to see how much
node support has improved lately.

Check out [OBDII-console!] [obdiicon]  It's a command line interface that let's you
query the [on-board computer] [obdiiwiki] in your car through a serial port.

It leverages a few node packages:

* [serialport] [serialport] (read/write to a serial port)
* [prompt] [prompt] (prompt the user for input)

   [bcljs]: http://swannodette.github.io/2014/12/21/browserless-clojurescript/   
   [obdiicon]: https://github.com/JMRodriguez24/obdii-console
   [obdiiwiki]: http://en.wikipedia.org/wiki/On-board_diagnostics
   [serialport]: https://www.npmjs.com/package/serialport
   [prompt]: https://www.npmjs.com/package/prompt
