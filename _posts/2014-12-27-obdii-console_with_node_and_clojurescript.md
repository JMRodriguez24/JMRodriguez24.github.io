---
layout: post
title: "obdii-console with nodejs and clojurescript"
categories: clojurescript
---

I recently read David Nolen's post on how he uses a [browserless clojurescript] [bcljs]
workflow.  It inspired me to give clojurescript/nodejs a try.

Check out [obdii-console!] [obdiicon]  It's a command line interface that let's you
query the [on-board computer] [obdiiwiki] in your car through a serial port.

It leverages a few node packages:

* [serialport] [serialport] (read/write to a serial port)
* [prompt] [prompt] (prompt the user for input)

Requiring node packages is simple with [cljs.nodejs] [cljs_nodejs] and the clojurescript needed to interop is straight forward.  In this code I use prompt to get some information from the user.

{% highlight clojure %}
(def prompt (nodejs/require "prompt"))

(def prompt-props
  #js {:properties 
    #js {:baudrate #js { :description "baud rate" }
         :port #js { :description "serial port name" :required true } } })

(defn -main []
  (.start prompt)
  (.get prompt prompt-props (fn [err result]
                              (if (not (nil? err))
                                (on-error err)
                                (run result)))))
{% endhighlight %}

This is what it looks like.

{% highlight sh %}

$ node run.js
prompt: serial port name:   COM2
prompt: baud rate:    9600
> 

{% endhighlight %}

Once it's running and you have specified a serial port and baud rate you can send any one of these commands.

Commands
=================

* __open__: new connecton to serial port
* __close__: end connection to serial port
* __exit__ or __quit__:  exit application
* Any PID from [PIDs] [pids]

Code
=================

The code is pretty straight forward.
Any text from `stdin` get's `put!` into `in-chan` and an action is taken depending on the command.
There are two other channels, `cmd-chan` and `data-chan`.  `cmd-chan` writes commands to the device and `data-chan` prints results coming back from the device.

{% highlight clojure %}
(defn run [p]
  (let [port (new serial-port (.-port p) #js { :baudrate (or (.-baudrate p) 9600) } false)
        stdin (.-stdin js/process)
        in-chan (chan 1)
        cmd-chan (chan 1)
        data-chan (chan 1)]
 
    ;; setup go loops 
    (print-result data-chan)
    (write-cmd cmd-chan port)
    (process-input in-chan cmd-chan data-chan port)

    (.resume stdin)
    (.setEncoding stdin "utf8")
    (print "> ")
  
    ;; Put input into core.async chan
    (.on stdin "data" (fn [text] (put! in-chan text)))))

(defn- print-result 
  [c]
  (go 
    (loop []
      (let [data (<! c)]
        (when data
          (do
            (print (str data))
            (recur)))))))

(defn- write-cmd 
  [c port]
  (go 
    (loop []
      (let [cmd (<! c)]
        (when cmd
        (do 
          (.write 
            port 
            cmd 
            (fn [err result] 
              (when (not (nil? err)) (on-error err))))
          (recur)))))))

(defn- process-input 
  [in-chan cmd-chan data-chan port]
  (go 
    (loop []
      (let [in (<! in-chan)]
        (when in
          (do  
            (cond

              (or (= (str "quit" eol) in) (= (str "exit" eol) in))
	      (do (print "Bye!") (close! in-chan))

              (= (str "close" eol) in) 
	      (.close port)

              (= (str "open" eol) in) 
	      (.open port 
		     (fn [err] 
		       (if (not (nil? err)) 
			 (on-error err)
			 (do 
			   (print "> ") 
			   (.on 
			     port 
			     "data" 
			     (fn [data] (put! data-chan data)))))))

              :else 
	      (put! cmd-chan in))

            (recur)))))
    (close! data-chan)
    (close! cmd-chan)
    (.exit js/process)))

{% endhighlight %}

Here is an example query.

{% highlight sh %}

# 010C is the PID for RPM
> 010C
010C

41 0C 31 BA

# ((A*256)+B)/4 = 3183

{% endhighlight %}

So the RPM at that instant was 3183.

Playing with node through clojurescript is a lot of fun!


   [bcljs]: http://swannodette.github.io/2014/12/21/browserless-clojurescript/   
   [obdiicon]: https://github.com/JMRodriguez24/obdii-console
   [obdiiwiki]: http://en.wikipedia.org/wiki/On-board_diagnostics
   [serialport]: https://www.npmjs.com/package/serialport
   [prompt]: https://www.npmjs.com/package/prompt
   [cljs_nodejs]: https://github.com/clojure/clojurescript/blob/master/src/cljs/cljs/nodejs.cljs
   [pids]: [http://en.wikipedia.org/wiki/OBD-II_PIDs]
