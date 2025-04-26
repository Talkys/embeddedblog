+++
title = 'The smoke detector'
date = 2025-04-26T11:26:37-03:00
draft = false
+++

So, I had the idea to make a smoke detector (gas too, but that's just a bonus) that doesn't require a fire central  
(I really don't know what those things are called outside Brazil) to log information to a network.

The hardware would be simple, but it would require software that's much more complex than anything I've done here before.

<!--more-->

### The Idea

First, let's see what we want to achieve:

1. A smoke detector capable of logging data  
2. Some way to connect it to a network  
3. Some way to retrieve the data  
4. A way to reset it

Seems simple, but let's see why it's not:

The first problem: using Ethernet to connect is bad, since no one wants to run a bunch of Cat5 cables around the house for that.

The second problem: domestic Wi-Fi networks have passwords (at least, I hope they do in every house).

This is a huge problem, but Ericsson can save us with Bluetooth.

### Bluetooth

We can use Bluetooth to connect to the controller and send a config text stream to set up its connection with a temporary public network that we can disable later.

For that, we can set two modes of operation: **log** and **config**. I have a switch for that.

### How We Can Do It

We need to make the ESP32 controller operate in two modes:  
- In the first mode, it will receive Bluetooth messages and set up its configuration.  
- In the second mode, it will start an HTTP server to log data, and maybe use part of the internal storage for a time window.

With the server running, a mobile app will be able to scan the network to find installed devices, catalog them, and request their data.

In theory, this would work for many sensors. One big plus is that if you have a VPN (like in a company), you could check your readings from anywhere to see if a fire has started or if gas is leaking. You could even have another server monitoring it remotely for you.

### The Setup

The hardware setup is very simple, as I'll show in this image:


![Circuit](/images/diagram_01.png)

#### The real problem is the software setup.

Of course, we could use a simple web server, but what I want to do is run the server on the ESP and use a mobile app to display the data. I'm not a beginner dev, but I'm hoping that a Kotlin app will be less of a mess than my graduate experience with Flutter was.

### Conclusion

In the end, what's left for me is to elaborate and run some tests before diving into it.

I'll try to set up the dual operation mode on the ESP in another post eventually.

— Talkys
