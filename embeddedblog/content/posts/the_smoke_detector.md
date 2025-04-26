+++
title = 'The smoke detector'
date = 2025-04-26T11:26:37-03:00
draft = false
+++

So I had the idea to make a smoke detector (gas too but that's just a bonus) that does not require a fire central
(I really don't know how those things are called outside Brazil) to log information in a network.

Would be simple in hardware but require software much more complex than anything I've done here.

<!--more-->

### The idea

So first let's see what we want to do:

1) A smoke detector capable of logging data
2) Some way to connect it to a network
3) Some way to get the data
4) A way to reset it

Seems simple but let's see how it is not:

The first problem is: Using ethernet to connect is bad since no one wants to pass load of cat5 cable around the house for that.

The second one is: Domestic Wi-fi networks have passwords (at least I hope they do in every house)

This is a huge problem but Ericsson can save us with Bluetooth.

### Bluetooth

We can use bluetooth to connect to the controller and send a config text stream to setup it's connection with a temporary public connection that we can disable later.

For that we can set two modes of operation: log and config. I got a switch for that.

### How can we do it then

We need to make the esp32 controller to operate in two modes, in the first it will receive bt messages and setup it's configs and in the second it will start an http server to log data, nad maybe use part of the internal storage for a time window.

With the server running, a mobile app will be able to scan the network to find installed devices catalog them and request their data.

In theory this would work for many sensors, but one big this is that if you have a VPN (like in a company) you can see your reading from any place and know if a fire started or if gas is leaking, and even use another server to monitor that from another place for you.

### The setup

The hardware setup is very simples as I'll show in this immage:

![Circuit](/images/diagram_01.jpg)

#### The real problem is the software setup.

Of course we can use a simple web server, but what I want to do is: Run the server in the ESP and use a mobile app to display the data. I'm not a beginner dev but I'm hopping that a Kotlin app would be less of a mess than what my graduate experience with Flutter was.

### Conclusion

In the end what's left for me is just elaborate and do some tests before diving into it.

I'll try to setup the dual operation mode in the ESP on another post eventually.

— Talkys