---
title: "NWS Alerts"
date: 2018-11-18T19:47:48-08:00
draft: false
tags: ["home-assistant", "automations"]
---

# 3.1 Weather Alerts sent to Telegram via Home Assistant

**Updated 2018-11-20**

User _finity_ on the Home Assistant Forums [posted a some REST sensors](https://community.home-assistant.io/t/severe-weather-alerts-from-the-us-national-weather-service/71853/11) for getting National Weather Service alerts from home assistant. I took this idea (and copy and pasted their [REST sensor](https://www.home-assistant.io/components/sensor.rest/)) to use as an automation that will send me alerts via Telegram. As _finity_ points out, this is a nice way to insulate you from the possible shutdown the of the Weather Underground API. It's also a fun use of a valuable public API. **Note:** The National Weather Service APIs are only available for US locations.

_finity's_ implementation is much more involved than mine and includes their Echos vocalizing the the weather alert. You should [check it out](https://community.home-assistant.io/t/severe-weather-alerts-from-the-us-national-weather-service/71853/11).

![Example of the NWS Alert Text sent through Telegram.](/images/NWS-Alerts_1-telegram-example.png)

## Prerequisites

At this point I have already set up a Telegram bot named Janet (same as the invocation for my alexa skill)  as `notification.general`. That was set up using the [notification component's docs](https://www.home-assistant.io/components/notify.telegram/).

## Sensors

Two REST sensors are used to pull from Nation Weather Service APIs.

The first looks for the total number of alerts in my area.

```YAML
- platform: rest
    resource: https://api.weather.gov/alerts/active/count
    name: NWS Alert Count
    value_template: >
      {% if value_json.zones.ORZ604 is defined %}
        {{ value_json.zones.ORZ604 | int }}
      {% elif value_json.zones.ORC039 is defined %}
        {{ value_json.zones.ORC039 | int }}
      {% else %}
        0
      {% endif %}
    headers:
      User-Agent: Homeassistant
      Accept: application/ld+json
    scan_interval: 300
```

This looks for alerts in my zone `ORZ604`, failing that it looks for alerts in my county `ORC039`, failing that it sets a value of 0 (zero). I have this set to update every 300 seconds (5 minutes). You can find your zone and county IDs on [this page](https://alerts.weather.gov/).

The second sensor requests the actual alert data from my zone and county and feeds any existing value into its sensor attributes under `features`. If there is no data, then the attribute is set to None. This has the limitation of only setting the first alert in the area to the sensor's state, but at the moment I'm not using the state anyway. The full features array is stored to the sensor's attributes, so I can loop through all events in my automation.

```YAML
- platform: rest
    resource: https://api.weather.gov/alerts/active?zone=ORZ604,ORC039
    name: NWS Alert Event
    value_template: >
      {% if value_json.features[0] is defined %}
        {{ value_json['features'][0]['properties'].event }}
      {% else %}
        None
      {% endif %}
    json_attributes:
      - features
    headers:
      User-Agent: Homeassistant
      Accept: application/geo+json
    scan_interval: 300
```

## Automation

The automation uses a template condition to handle any change that results in a count from the first sensor that is greater than 0 (zero). If true and if this automation has not beed triggered in the last 6 hours then we send a notification via the `notify.general` service. The first NWS sensors gives a handy integer to use in for range. My message template starts with a fun header (chipper voice of doom) and then concatinates all alerts together.

```YAML
- id: weather_alert_doooooom
  alias: NWS Notification Weather Alert
  trigger:
    platform: state
    entity_id: sensor.nws_alert_count
  condition:
    - condition: template
      value_template: '{{states.sensor.nws_alert_count.state | int > 0}}'
    - condition: template
      value_template: '{{ as_timestamp(now()) | int - as_timestamp(states.automation.nws_notification_weather_alert.attributes.last_triggered) | int > 21600 }}'
  action:
  - data_template:
      message: >
        Hi There,

        I've detected a change in national weather service advisories for Eugene.
        {% for i in range(states('sensor.nws_alert_count') | int) %}
          {%- if i > 0 -%}
            ---------
          {%- endif %}
          {{ state_attr('sensor.nws_alert_event', 'features')[i].properties.headline }}
          {{ state_attr('sensor.nws_alert_event', 'features')[i].properties.description }}
        {% endfor %}
    service: notify.general
```

## Still to come

I don't love that I'm just blocking alerts for 6 hours. I get alert fatigue quickly, but I want to know if conditions are deteriorating. But what seems to happen most commonly is that an alert is retracted, then a new alert is issued with minimally updated information.

I want to add a Lovelace condition card to my outdoor climate tab that will pop up, highlighted, showing that there's an alert. When I set up HTML notifications I may also want to push to that. I'm also considering changing the detailed sensor. Right now I like the flexibility of the full data from the API, but I don't think I'll want more than the headline and description.