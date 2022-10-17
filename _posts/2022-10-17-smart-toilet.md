---
layout: post
title: Bathroom Door Monitor with ESPHome
subtitle: Home automation to help with bathroom congestion
tags: [esp8266, esphome, homeassistant]
comments: true
thumbnail-img: /assets/img/blog/smart-toilet-thumb.jpg
cover-img: /assets/img/blog/smart-toilet-header.jpg
---

In this simple tutorial I will show you how I transformed my normal bathroom door into a smart bathroom door to help with congestion.

## **Parts**
- [ESPHome framework](https://esphome.io/)
- [ESP8266](https://amzn.to/3T9qgra) (I use an [Adafruit HUZZAH ESP8266](https://www.adafruit.com/product/2471))
- [Active piezo buzzer](https://amzn.to/3S7Rb5k)
- [Neopixel LED bar](https://amzn.to/3yLRCLY)
- [Magnetic door sensor](https://amzn.to/3D4bRXD)
- 3D printed enclosure (optional)

## **Pictures**

![1st installation picture](/assets/img/blog/smart-toilet-install.jpg){: .mx-auto.d-block :}

## **Context**

First of all, let's talk about [ESPHome](https://esphome.io/), this awesome project is basically a framework to build firmware for different ESP MCUs. You just have to provide a config file written in YAML and you are good to go without writing C code. It was my first time using it for this simple project but it was my fastest idea to working prototype ever.

The basic idea is the following, we are always between 6 to 8 people in my home and we only have one bathroom. So it happen quite often that you need to go but someone is already there so you have to wait behind the door. So I wanted to be able to see if the door was opened or closed remotely and for how much time. I also wanted to be able to send an alarm if the person in the bathroom was there for a suspicious amount of time.

So the solution was to use [ESPHome](https://esphome.io/) to make the bathroom door "smart" and combine that with [Home Assistant](https://www.home-assistant.io/) to make it easy to see the data from any computer or cellphone.

The MCU with [ESPHome](https://esphome.io/) can work on its own but combining it with [Home Assistant](https://www.home-assistant.io/) makes it far more user-friendly.

## **Code**

You'll see that the code is very basic with a couple of more complex snippets but ESPHome makes it really simple and the documentation on their site is really well made. Everything is declared in the YAML config file like little modules that can interact with each other.

That first block in the YAML config is very simple, you just have to set the right MCU you are using and your wifi informations. It is also strongly suggested to have an OTA update password, if not anybody that has access to your local network can push a firmware update from a web browser. NTP time syncing is also strongly suggested since I want to see the real date and time for each door events.

{% highlight yaml linenos %}
esphome:
  name: wifitoilet
  on_boot:
    - light.turn_on: status_l

esp8266:
  board: huzzah

# Enable logging
logger:

# Enable Home Assistant API
api:
  password: ""

ota:
  password: "****************"

wifi:
  ssid: "CanIGetA_OhYeah"
  password: "**************************"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Wifitoilet Fallback Hotspot"
    password: "UMG9vSbYhkfu"
    
web_server:
  port: 80
  
  time:
  - platform: sntp
    id: sntp_time
    timezone: America/Toronto
{% endhighlight %}

So let's get back to the main idea, making that bathroom door smart, I wanted to have the following features ;
- View the door state remotely
- View since when the door was closed
- Warn the person in the bathroom that someone is waiting

First I needed a way to tell the ESP if the door is opened or closed, for that I used a simple magnetic door sensor and connected it to a GPIO pin. As you can see below, I defined a binary sensor and I set the pin to 12. I also added a delay of 250ms to change between states to avoid reporting too fast to Home Assistant. I also named the sensor and set its device class.

{% highlight yaml linenos %}
binary_sensor:
  - platform: gpio
    pin:
      number: 12
      mode:
        input: true
        pullup: true
    filters:
      - delayed_on_off: 250ms
    id: toilet_door_sensor
    name: Toilet Door
    device_class: door
{% endhighlight %}

Now the only thing remaining is being able to warn a bathroom user that is taking a bit too much time. For this I used an active piezo buzzer and some addressable RGB LEDs, with this I have an audio and visual warning.

First thing I needed is a PWM output pin to play a tone with the piezo buzzer. I selected pin 14, defined it as 'esp8266_pwm' and set its id as 'rtttl_out'. Then I defined a RTTTL object that use the previously configured 'rtttl_out'.

Right after this, I defined the warning LEDs. I selected 'neopixelbus' because this is what I had on hand, there's a lot of options in esphome so select what you prefer. I also need to tell esphome a couple details about the LEDs strip used, mine had 8 WS2812 LEDs on it and it's controlled by pin 5 of the ESP. You can also see that I defined a simple custom light effect in the 'effects' section.

Now all I need is a software button to activate the buzzer and light. The button definition itself is pretty simple, all the magic happens in the 'on_press'event. You can see that I first call 'rtttl.play' with a strange looking string, that string is actually the different tones and timings to play instead of just playing a single tone. You can find a lot of those on the web. Then using lambda calls to turn on the LEDs and perform the effect I defined earlier, do that effect for a 10 seconds delay and turn off the LEDs.

{% highlight yaml linenos %}
output:
  - platform: esp8266_pwm
    pin: 14
    id: rtttl_out

rtttl:
  output: rtttl_out
  on_finished_playback:
    - logger.log: 'Song ended.'
    
light:
  - platform: neopixelbus
    id: sos_light
    type: GRB
    variant: WS2812
    pin: 5
    num_leds: 8
    name: "SOS Light"
    internal: true
    effects:
      - strobe:
          name: "Poopoo-Alert"
          colors:
            - state: true
              brightness: 100%
              red: 100%
              green: 100%
              blue: 0%
              duration: 1000ms
            - state: false
              duration: 350ms
            - state: true
              brightness: 100%
              red: 100%
              green: 0%
              blue: 0%
              duration: 1000ms

button:
  - platform: template
    id: button_warning
    name: I need to go!
    on_press:
      - rtttl.play: "Looney:d=4,o=5,b=140:32p,c6,8f6,8e6,8d6,8c6,a.,8c6,8f6,8e6,8d6,8d#6,e.6,8e6,8e6,8c6,8d6,8c6,8e6,8c6,8d6,8a,8c6,8g,8a#,8a,8f"
      - lambda:
          auto call = id(sos_light).turn_on();
          call.set_effect("Poopoo-Alert");
          call.perform();
      - delay: 10s
      - lambda:
          auto call = id(sos_light).turn_off();
          call.perform();
{% endhighlight %}

There's also a great little feature in esphome called 'status_led', you can basically use any LED through a simple GPIO to have a visual indicator of the health of the system. Super simple to configure and voila!

{% highlight yaml linenos %}
light:
  - platform: status_led
    name: "Status LED"
    pin:
      number: 0
      inverted: true
    id: status_l
    internal: true
{% endhighlight %}

I printed a small enclosure and power the device with a simple 5V wall adapter.

I hope this can help someone, feel free to ask questions or comments down below!

Thanks for reading ;-)
