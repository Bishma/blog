---
title: "Mouse shortcuts with xbindkeys & xdotool"
date: 2020-02-02T10:54:44-08:00
draft: task
tags: [ "automations", "linux" ]
---

# Context Aware Mouse Button Shortcuts in Gnome

The thing I've missed the most, since switching to full time linux, is AutoHotKey. It was my go to for customizing controls and making macros. A lot of the things I used are different enough now that the AHK script wouldn't be applicable anyway, but my context aware mouse buttons have been a huge hit to the way I work and play. I use a 9 button Logitech Vertical Mouse. It's a form factor I prefer and it has helped so much with my RSI that I have one at home and one at work. The only bad thing I have to say about it concerns the lack of support it has in linux.

 Thankfully I've [found an approach](https://github.com/Bishma/.xbindkeys) that replicates the mouse button shortcuts I used the most via a combination of [xbindkeys](https://linux.die.net/man/1/xbindkeys), [xdotool](http://manpages.ubuntu.com/manpages/trusty/man1/xdotool.1.html), and a bash script.

 ## Problem 1: Determine the Active Window

The way I tend to work I can get away with simply knowing the process name and window title of the window that is currently focused.

    # The window title is available straight from xdotool
    ACTIVE_WINDOW_TITLE=$(xdotool getactivewindow getwindowname)
    # Getting the process name requires back tracking through the active windows pid
    ACTIVE_WINDOW_PROCESS=$(ps -p "$(xdotool getactivewindow getwindowpid)" -o comm=)

Here `xdotool getactivewindow getwindowname` will give me the title of the the active window. I can get the the PID of that window using `xdotool getactivewindow getwindowpid` and use `ps` to get the process name from that.

## Problem 2: Sending Keystrokes to the Active Window

## Problem 3: When in Firefox, What Site Am I On

I like to use mouse buttons 8 and 9 as next/previous keys on my most frequently visited websites. The shortcut is different on various sites so I needed a way to distinguish what side I'm on. While I haven't been able to find a way to get the url of the active tab in Firefox (without hijacking my clipboard) but I was able to gleen what I needed from the window title.

    function get_firefox_site() {
        if [[ $ACTIVE_WINDOW_TITLE =~ " / Twitter" ]]; then
            SITE="twitter"
        elif [[ $ACTIVE_WINDOW_TITLE =~ "All Personal Feeds" ]]; then
            SITE="feedly"
        else
            SITE="other"
        fi

        echo $SITE
    }

The sites I use next/prev on the most are Twitter, Feedly, and Reddit. Unfortunately when reddit opens thelightbox of a post it makes the title match the post title. So my detection amounts to Twitter, Feedly, and "Other."