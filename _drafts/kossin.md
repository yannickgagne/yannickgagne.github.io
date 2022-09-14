---
layout: post
title: ESPHome Bathroom Monitor
subtitle: Home automation to help with bathroom congestion
tags: [esp8266, esphome, homeassistant]
comments: true
thumbnail-img: /assets/img/blog/xxx-thumb.jpg
cover-img: /assets/img/blog/xxx-header.jpg
---

In this simple tutorial I will show you how I made a little desk clock based on the esp8266 MCU with MicroPython. The project includes include the following features :
- ESPHome framework
- Magnetic door sensor (Reed switch)
- Piezo buzzer
- Neopixel LED bar

## **Parts**
- ESP8266 (I use an [Adafruit HUZZAH ESP8266](https://www.adafruit.com/product/2471))
- Active piezo buzzer
- Neopixel 10 LED bar
- Magnetic door sensor
- 3D printed enclosure (optional)

## **Connections**

![Fritzing](/assets/img/blog/xxx-fritzing.png){: .mx-auto.d-block :}

![Real thing](/assets/img/blog/xxx-connections-thumb.jpg){: .mx-auto.d-block :}

## **Context**

First of all, let's talk about [ESPHome](https://esphome.io/), this awesome project is basically a framework to build firmware for different ESP MCUs. You just have to provide a config file written in YAML and you are good to go without writing C code. It was my first time using it for this simple project but it was my fastest idea to working prototype ever.

The basic idea is the following, we are always between 6 to 8 people in my home and we only have one bathroom. So it happen quite often that you need to go but someone is already there so you have to wait behind the door. So I wanted to be able to see if the door was opened or closed remotely and for how much time. I also wanted to be able to send an alarm if the person in the bathroom was there for a suspicious amount of time.

So the solution was to use [ESPHome](https://esphome.io/) to make the bathroom door "smart" and combine that with [Home Assistant](https://www.home-assistant.io/) to make it easy to see the data from any computer or cellphone.

The [ESPHome](https://esphome.io/) can work on it's own but combining with [Home Assistant](https://www.home-assistant.io/) makes it far more user-friendly.

## **Code**

You'll see that the code is very basic with a couple of more complex snippets but ESPHome makes it really simple and the documentation on their site is really well made. Everything is declared in the YAML config file like little modules that can interact with each other.



{% highlight python linenos %}

{% endhighlight %}


I printed a small enclosure and power the device with a simple 5V wall adapter.

I hope this can help someone, feel free to ask questions or comments down below!

Thanks for reading ;-)