---
title: "Ubuntu on Raspberry Pi - Logitech keyboard not working"
date: 2022-04-22
draft: false
tags: ['raspberry-pi','ubuntu']
---

When running a fresh install of Ubuntu 21.XX Desktop x64 on a Raspberry Pi, I always have issues with using my Logitech Keyboard (K400r).

I can use the keyboard to log in and type text within the search bar. But if I try and type in the terminal or Firefox search bar, nothing works.

I have found this  [workaround](https://forums.raspberrypi.com/viewtopic.php?t=310293#:~:text=Re%3A%20ubuntu%2021.04%20%2D%2D%20keyboard%20not%20typing%20in%20apps,-Tue%20Jul%2013&text=For%20those%20who%20are%20wondering,ALT%20%2B%20F3%20to%20get%20terminal.) provided by user matzelchen on the Raspberry Pi forum, and adding here so I can easily find it again.

1. Remove the hid_logitech module from the kernel

    ```bash
    sudo rmmod hid_logitech_dj && sudo rm /lib/modules/$(uname -r)/kernel/drivers/hid/hid-logitech-dj.ko
    ```

2. Remove usb bluetooth receiver
3. Reconnect usb bluetooth receiver
