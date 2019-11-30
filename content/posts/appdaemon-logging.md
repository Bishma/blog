---
title: "Appdaemon Logging"
date: 2019-07-22T18:33:18-08:00
draft: false
tags: ["home-assistant", "appdaemon"]
---

# Combined logging between Home Assistant and AppDaemon

I've begun using [AppDaemon](https://appdaemon.readthedocs.io/en/latest/) apps to extend [Home Assistant](https://www.home-assistant.io/). It's a way to write sandboxed [Python](https://www.python.org/) apps that have access to [home-assistant](https://www.home-assistant.io/) events, devices, services, and presence. It's intended to be replacement for automations that can leverage all the power of python. You can also use it as a way to create sensors in Home Assistant.

My [first foray](https://github.com/Bishma/home-assistant-tng/blob/master/appdaemon/apps/lunch-schedule.py) into AppDaemon (and the Python Language) makes use of an API we have at work to read our next three lunch menus and turn them into a Home Assistant sensor. If lunch is being served the sensor will contain what's for lunch. There is also an attribute on the sensor containing the next three days' menus. I then wrote a [crude custom lovelace card](https://github.com/Bishma/custom-lovelace-cards/tree/master/list-item-card) to display that on my frontend.

![Custom card displaying attributes from an Appdaemon Sensor](/images/appdaemon-logging_1-custom-card.png)

In working through this I found that needing to keep track of multiple logs was tedious. And since I'm using [Hass.io](https://www.home-assistant.io/hassio/) with the [LogViewer](https://github.com/hassio-addons/addon-log-viewer) addon, I'd like my AppDaemon app logs to be visible there. After a lucky find in the [Home Assistant Forums](https://community.home-assistant.io/t/adding-logs-from-appdaemon-to-the-main-home-assistant-log/105722) I was able to piece together what I wanted.

#### Step 1:

I need to use the home assistant [python script component](https://www.home-assistant.io/integrations/python_script/) to expose a generic logging service. I can use the resulting service in AppDaemon to pass logs into Home Assistant for logging.

```python
message = data.get('message')
if not message:
  logger.error('No message provided')

# Send to the appropriate log level
received_level = str(data.get('level')).lower()
if received_level == 'debug':
  logger.debug(message)
elif received_level == 'warning':
  logger.warning(message)
elif received_level == 'error':
  logger.error(message)
else:
  logger.info(message)
```

This accepts a message and an optional level. If no level is passed it logs as INFO. This is saved in `<config>/python_scripts/log.py`. Once this is added (and Home Assistant restarted with the python script component enabled) I have a service called `python_script.log`.

#### Step 2

Fine grained [logger](https://www.home-assistant.io/components/logger/) configs allow me to keep my Home Assistant logs outputting at my desired level (warning) while allowing me to turn things that come through my new `python_script.log` service all the way to debug when I'm developing a new AppDaemon app.

```yaml
# "The logger component lets you define the level of logging activities in Home Assistant."
# https://www.home-assistant.io/components/logger/
logger:
  default: warning
  logs:
    homeassistant.components.python_script.log.py: debug
```

Each call to the `python_script.log` service will generate a log at the info level. Having a default log level of warning here has the benefit of not showing my all my AppDaemon logs twice.

#### Step 3

AppDaemon has the means to register a listener on all calls to self.log in all running AppDaemon apps. To do this you want to set up a stand-alone log handling AppDaemon app.

```python
import appdaemon.plugins.hass.hassapi as hass

class LogBridge(hass.Hass):

    def initialize(self):
        self.listen_log(self.cb)
    
    def cb(self, name, ts, level, message):
        msg = "[AppDaemon] {}: {}".format(name, message)
        self.call_service("python_script/log", level = level, message = msg)
```

The callback (`cb`) formats a message for readability and then passes that and the level to the Home Assistant `python_script.log` service created above. Save this in the apps directory as `log_bridge.yaml`.

#### Step 4

Register this script as an AppDaemon app in `apps.yaml`.

```yaml
log_bridge:
  module: log_bridge
  class: LogBridge
```

After a reboot every log line from AppDaemon pipes into home assistant.