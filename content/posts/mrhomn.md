---
title: "MrHomn"
date: 2021-06-27T15:09:54-07:00
description: "My new home automation / data intelligence plan. Honing my skills and having fun."
images:
    - /images/mrhomn/diagram.png
tags: [ "Summer of 42", "home-assistant", "appdaemon", "sysadmin", "mrhomn" ]
draft: false
---

# My new home automation plan

{{< figure src="/images/mrhomn/mrhomn1.jpg" width="70%" >}}{{< /figure >}}

When I decided to leave my job of over 15 years I knew that I wanted a bit of a career rewind. The last several years have seen me in full-time management positions and my programming, sysadmin, and SRE skills have gotten rusty. Since I learn by doing I wanted a project that would get me up-to-date and which I'd have fun implementing.

[MrHomn](https://github.com/Bishma/MrHomn) is the 3rd generation of my home automation setup[^explain]. Which I plan to expand out to utilize common open source tools

## Stack Elements

{{< figure src="/images/mrhomn/diagram.png" width="70%" >}}
The Planned Tech Stack... vaguely
{{< /figure >}}

### Synology

I'll be using my [Diskstation](https://www.synology.com/en-us) as my storage pool, docker host, and backup agent.

### hass

[Home Assistant](https://www.home-assistant.io/) is the brain of the setup. It provides the UI's, device integrations, data recording, state tracking, events, and more.

### sql_db

I've gone with [MariaDB](https://mariadb.org/) for now. Home Assistant makes use of [SQLAcadamy](https://www.sqlalchemy.org/) which opens up a lot of DB options. At some point I may have more specific needs but for now the versatility of MariaDB seems like a good choice.

### appdaemon

[AppDaemon](https://appdaemon.readthedocs.io/en/latest/) is a great way to incorporate loosely coupled Python apps with home assistant. Most critically this is how I'll be creating my APIs to Home Assistant. My existing alexa skill connects to an [appdaemon api](https://appdaemon.readthedocs.io/en/latest/AD_API_REFERENCE.html#alexa) which makes use of `sql_db`.

### nginx

Initially, at least, I'm going to need a public API to support my Alexa skill. [Nginx](https://www.nginx.com/) is how I'll handing internal and external routing within my stack.

## Near Future Goals

#### Monitoring

I'm currently weighing [Prometheus](https://prometheus.io/) vs [Zabbix](https://www.zabbix.com/). I'd learn more implementing Prometheus but it's overkill for this project.

#### Log Management

TBD, but I'm leaning toward [Greylog](https://hub.docker.com/r/graylog/graylog).

#### RUM and Synthetics

This is running on my local network. I'm not going to have enough traffic for these to be of any use but it is one of the things I'd like to grain some practical experience with. ¯\\\_(ツ)_/¯ 

### Data Analysis

#### timeseries_db

SQL is well and good, but if you want to evaluate trends you want to be able to query for timeseries data. I am planning to use [InfluxDB](https://www.influxdata.com/).

I will also make use of InfluxDB as the storage backend for Prometheus.

#### Grafana

[Grafana](https://grafana.com/) is a solid visualization platform that is common use. Understanding it at more than basic level will pay dividends.

### Purpose built sensors

It has been decades since I did anything with hobbyist electronics and I intend to make the time to relearn it. I intend to use ESP modules and simple sensors anywhere I can figure out how to power them. I'd like to sample the temp and humidity in each space of my house. I'd like to add CO<sub>2</sub>, Lux, dB, and in key areas. And I also want some manner particulate sensor(s) in my attic due to local wildfires. For added fun I can 3D Print each sensors' housing.

{{< figure src="/images/mrhomn/arduino-example.png" width="60%" >}}
[Photo from](https://www.circuitbasics.com/how-to-set-up-the-dht11-humidity-sensor-on-an-arduino/)
{{< /figure >}}

Additionally I'm planning to build an RPi based outdoor weather station.

#### ESPHome

Those purpose built sensors have to tell someone what they sense. That someone is [ESPHome](https://github.com/esphome/esphome) which I plan to deploy in the [docker](https://hub.docker.com/r/esphome/esphome) stack.

### Telegram Chatbot

My goal is to be able to duplicate a natural language interface for both voice assistants and telegram. Telegram has the extra benefit of being able to present options as buttons.

### No hardware requiring the cloud

I've got a number of devices in my current home automation that require a cloud provider. I want to limit, if not eliminate this weakness. First it means that if the internet is out my house is degraded. It also means that I'm reliant on these companies keeping their services online and open enough for home assistant integrations to work. Also, I want my data and I want to stop giving so much of it away. Frankly I don't think any of these companies properly compensate us for the wealth of data we expose.

#### Voice Assistant

The little bit of research that I've done so far suggests that the weak point in the realm of open/local voice assistants is wake work detection. Every time an OSS project starts to get close it gets snapped up by companies looking to compete with Amazon and Google.

All of that is to say that I want to replace Alexa. I'll be doing various experiments to see how viable it is.

#### Doorbell

I'm hearing that the [Amcrest wifi doorbell](https://amcrest.com/smarthome-2-megapixel-wireless-doorbell-security-camera-1920-x-1080p-wifi-doorbell-camera-ip55-weatherproof-two-way-audio-ad110.html) is decent hardware with a terrible app to back it up. But it has the but plus of not being cloud based. I can mimic alerting with Home Assistant + Telegram Bot but I don't want to loose the intercom functionality. There's enough chance that we won't be happy with the end result that I'm not willing to spend the money on one yet.

We have a couple Amcrest PTZ cameras connected to home assistant via [Synology Surveillance Station](https://www.synology.com/en-us/surveillance).

### PyTorch

#### Nvidia Jetson

This is a very affordable device that should be capable of keeping up with the tiny amount of learning and inference I'll "need" for my house.

#### Face detection

Nothing creepy, but if we have non-cloud door cams I can have MrHome only send notifications if it identifies a human in the camera's view when a motion detection triggers. And if I can get it to recognize ~~my many enemies~~ friends then I can ~~launch countermeasures~~ play harp music... or something.

#### Routine Detection

This is a half-baked idea at this point, but I'd like to have MrHomn be able to do something like:

* Detect a recurring change to my day-to-day routine
* Generate an automation change proposal based on the new behavior
* Send a Telegram message to suggest the change
* Implement the change if approved

E.g. "I've noticed you're going to bed later recently. Would you like me to delay your pre-bedtime automation by 30 minutes?"

----

{{< figure src="/images/mrhomn/mrhomn2.jpg" width="60%" >}}{{< /figure >}}

[^explain]: OK, yes, I promised, after naming my second home automation repo "[HomeAssistant TNG](https://github.com/Bishma/home-assistant-tng)," that I'd name my next one "Deep Space Nine." But [MrHomn](https://memory-alpha.fandom.com/wiki/Homn) was too good to pass up.

    He was [Lwaxana Troy's](https://memory-alpha.fandom.com/wiki/Lwaxana_Troi) very diligent valet in Star Trek TNG, three  syllable wake words tend to work best for VUIs, [Carel Struycken](https://en.wikipedia.org/wiki/Carel_Struycken) is awesome, and Homn is a homonym of home (at least the way Lwaxana pronounced it).
