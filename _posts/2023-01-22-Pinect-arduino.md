---
title: Chapter 4. Arduino firmware
date: 2023-01-20
categories: [Pinect]
tags: [Pinect, electronics, arduino, firmware]     # TAG names should always be lowercase
image: http://ser-mk.github.io/assets/Pinect/ch3/preview.png
description: Controlling everything from the camera's power supply to cooling the Pinect game console
---


In my previous post, we took an in-depth look at the Pinect control board and all of its features. There's still so much more to discover. We're going to examine the firmware that powers it all. This software is responsible for controlling everything from the camera's power supply to cooling the board, and understanding it can help you modify the software to suit your specific needs.

A component that plays a central role in board operation: Arduino. With Arduino at the center of the Pi board, we can control all sorts of devices and systems.

1. Camera power
2. Turn off/on fan
3. Monitoring input signals from the sensors and tablet
4. Turn on/off LED
5. Signaling information through the use of the LEDs and the buzzer.

Download code from **[github](https://github.com/ser-mk/pirduino).**

Just like a typical Arduino program, our program is designed to continuously perform the operations above in a loop. In each iteration of the loop, the program first polls the motion sensors to gather information about the presence of humans in the area.

![motion_reg](/assets/Pinect/ch3/motion_reg.webp)

If at least one of the sensors detects movement, the program waits for half a second before polling the sensor again. This is to ensure that the sensor reading is accurate and to prevent false positives. If the sensor still detects movement after the second poll, the program begins to prepare Pinect for operation. This involves activating the camera to capture video, turning on the LED to highlight the target, starting the fan for device cooling and signaling the tablet to run the game.

Once all of the gadgets are ready, the Arduino program begins to monitor a reconnect signal from the tablet.

The signal from the tablet is used to reconnect the camera if it becomes disconnected or fails to connect properly the first time. Sometimes USB cameras can freeze while connecting, which can be frustrating and disrupt the functionality of the system. After the reconnection process is complete, the camera will typically function normally again.

![reconnect](/assets/Pinect/ch3/reconnect.webp)

The Arduino program is designed to constantly monitor the motion and temperature sensors in order to gather information about the presence of a gamer and the temperature of Pinect.

If the temperature of the devices rises above a predetermined threshold (40 degrees Celsius), the program might turn on a cooling fan to bring it down to a safe level. However, if the temperature continues to increase beyond this point, the program may decide to power off all systems and trigger an alarm.

![off](/assets/Pinect/ch3/off.webp)

If the gamer leaves the controller and neither motion sensor detects any movement for half a second, the program won't immediately power off the devices. Instead, it will wait for a minute to see if a new gamer enters the room. This way, you can get back to playing as quickly as possible without having to go through the hassle of restarting the systems.