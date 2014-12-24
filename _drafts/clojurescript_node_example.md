---
layout: post
title: "Playing with nodejs through clojurescript"
categories: clojurescript
---

I recently read David Nolen's post on how he uses a [browserless clojurescript] [bcljs]
workflow.  I used clojurescript to write a small tool to see how much
node support has improved lately.

Check out [OBDII-console!] [obdiicon]  It's a command line interface that let's you
query the [on-board computer] [obdiiwiki] in your car through a serial port.

It leverages a few node packages:

* [serialport] [serialport] (read/write to a serial port)
* [prompt] [prompt] (prompt the user for input)

Requiring node packages is simple with [cljs.nodejs] [cljs_nodejs].

TODO: code for requiring packages

Interop with node packages is straight forward.  In this code I use prompt to get some information from the user.

TODO:  prompt code

I used core.async channels to avoid callback hell.

Give it a try! 

TODO:  Pic of the console commands to run it.


First you have to open a connection, then you can start querying information from the onboard computer.  The quit command will exit the session. 

Indeed playing with node through clojurescript is a lot of fun!


   [bcljs]: http://swannodette.github.io/2014/12/21/browserless-clojurescript/   
   [obdiicon]: https://github.com/JMRodriguez24/obdii-console
   [obdiiwiki]: http://en.wikipedia.org/wiki/On-board_diagnostics
   [serialport]: https://www.npmjs.com/package/serialport
   [prompt]: https://www.npmjs.com/package/prompt
   [cljs_nodejs]: https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/nodejs.cljs
