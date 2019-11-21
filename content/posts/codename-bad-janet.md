---
title: "Codename: Bad Janet"
date: 2019-11-03T11:11:53-08:00
draft: true
tags: ["home-assistant", "alexa", "appdaemon"]
---

# A Better Alexa Skill

In 2016 I was given a 1st gen Echo Dot and I immediately sought to integrate it with [Home Assistant](https://www.home-assistant.io/). The quick way to so at the time was via the Philips Hue Bridge which allowed simple on/off control of devices and that was plenty. Over time more of my home was being controlled by Home Assistant and just turning things on and off wasn't enough.

I could achieve most of what I wanted using custom intent scripts. This allowed me to have nicer grammar when interacting with things like my media center, gave me the ability to pass variables to Home Assistant (like "turn up the volume _two times_"), and allowed me to get [jinja templated](https://www.home-assistant.io/docs/configuration/templating/) responses. At this point I think you can do most, if not all, of this with [Nabu Casa](https://www.nabucasa.com/) + template sensors + Alexa routines but at the time it was the most direct means to get what I wanted.

Now I want more. I want to be able to do fallbacks, have multi-step intents, do slot confirmations, and just generally have a good VUI. The most personally interesting way to approach this is to learn python and make use of [AppDaemon](https://appdaemon.readthedocs.io/en/latest/) to build a full featured Alexa API. I'll take a stepwise approach to achieving this, starting with trying to feature match (plus a few upgrades) my existing system.

## Minimum Viable Product

### On / Off

This was an interesting and entirely unnecessary problem to solve. We can do this outside of my skill because we also use Nabu Casa and the home assistant skill. I choose to duplicate the functionality as a personal challenge and because we get into the habit of invoking the skill and forget that the basics aren't part of it.

This presented the problem of needing to get a device registry to Alexa. I didn't want to have to hard-code any devices into my intent slots and [Catalog Management](https://developer.amazon.com/docs/smapi/catalog-url-reference.html) was more involved than I wanted to get for something only a couple people will use. I understand there are additional options if you make a skill that's the "home automation" type but I didn't investigate that very far because those need to be hosted in AWS Lambda, and I want to keep things local.

In the end I used a Search Query slot and then have my AppDaemon app search Home Assistant for the unbounded phrase. This will be the first place that I want to add validation and/or confirmation. Here is what is is able to search and control.

#### Example Phrases

- turn/switch {device} on
- turn/switch {device} off

Where {device} can be:

- All smart switches
- All smart lights
- All groups
  - Fixtures with multiple bulbs.
- The living room media center
  - Turns on by starting the Plex activity in Harmony.
- The living room stand fan
  - Toggle
- The Bedroom AC
  - Toggle

### Dynamic Up/Down

Turn things up or down by a dynamic amount. This will default to 1 if no value is specified.

#### Example Phrases

- turn {device} up
- turn {device} down twice
- turn {device} up by {increment}.

Where {device} can be:

- The TV
  - A.K.A. The Bose Soundbar
  - Synonyms: The Television, The Volume, The Television Volume
- The Fan
  - This is the living room stand fan (if out).
  - Speed only goes up, then loops to the beginning after the highest speed.
  - Synonyms: The Stand Fan, The Living Room Fan
- The Thermostat
  - Synonyms: The Heat, Ecobee
- The Bedroom AC
  - This will only work if the AC is already turned on.
  - Synonyms: The Bedroom Air Conditioner, The Bedroom AC Temperature, The Bedroom Air Conditioned Temperature

Increment can be any integer value (though I will probably add bounds to this)

### Media control

Control the base functions of the media center. These can all be triggered ne of more time. These won't work for the record player or over the air.

#### Example phrases:

These will all work followed by "the tv," ",the television", or "the living room tv / television."

- {action} the television
- {action} the tv twice.
- {action} the tv {increment} times

Where {action} can be:

- Confirm
  - Synonyms: Okay, Yes, Go
- Fast forward (10 seconds)
  - Synonyms: Go Forward, Skip Ahead
- Rewind (also 10 seconds)
  - Synonyms: Go Backward, Go Backwards
- Play
  - Synonym: Unpause
- Pause
- Stop
  - Synonyms: Back, Cancel
- Skip Back (1 minute)
  - ToDo: This will work whether followed by "tv" or "television" or not.

### Media Switch

Switching between Harmony activities.

#### Example

- Switch to {activity}
- Change to {activity}
- Make {activity} go now

Where {activity} can be:

- Amazon
  - Synonym: Amazon Prime
- Hulu
- Kodi
- Over the Air
  - Synonyms: OTA, Antenna
- Plex
- Record Player
  - Synonyms: Records, Bluetooth, 
- Spotify
  - Synonyms: Music
- Switch
  - Synonyms: Nintendo, Nintendo Switch, Console
- Sling
- Netflix

### Command Phrases

There are certain things we want to be able to trigger without a slot. These have analogs that as Alexa Routines.

#### Phrases

- skip back
  - Is also a media command, so "skip back the tv" will also work.
  - Performs a 60 second rewind in all video based Harmony activities.