+++
hasMath = false
draft = false
summary = "How to turn a display on and off based on an infrared motion detector (PIR) using Python."
date = "2017-05-21T15:09:23+02:00"
title = "Controlling a display based on a PIR motion detector"
tags = [
  "tech"
]
showLogo = true
logo = "/imgs/logos/raspi_pir.jpg"
+++
I am working on a [Magic Mirror](https://www.raspberrypi.org/blog/magic-mirror/): that's an old LCD display behind a two-way mirror. The mirror shows your reflection when you look at it, but all light coming from behind it passes through - so stuff displayed on the LCD appears on the mirror. A Raspberry Pi 3 controls what is shown using [this open source modular smart mirror platform](https://magicmirror.builders/).

Of course I don't want the display to be always on - that'd be bad for my electricity bill. Instead I want to detect if a person is standing in front of the mirror and only then turn it on.
For this I'm using a [passive infrared (PIR) sensor](https://en.wikipedia.org/wiki/Passive_infrared_sensor) which connects to three GPIO pins on the raspi.

Some Python then controls the logic:

```python
from gpiozero import MotionSensor
import time
from time import sleep
import subprocess

SENSOR_PIN = 23
stayAliveTime = 30
pir = MotionSensor(SENSOR_PIN)

try:
    lastTimeActivated = time.time()
    subprocess.check_call(['/home/pi/bin/turn-screen', '1'])
    currentlyOn = True
    while True:
	if currentlyOn and not pir.motion_detected and time.time() - lastTimeActivated > stayAliveTime:
            print "Movement stopped on " + time.strftime('%d. %b %X')
            subprocess.check_call(['/home/pi/bin/turn-screen', '0'])
	    currentlyOn = False
	if pir.motion_detected:
            print "Movement detected on " + time.strftime('%d. %b %X')
            lastTimeActivated = time.time()
	    if not currentlyOn:
		subprocess.check_call(['/home/pi/bin/turn-screen', '1'])
		currentlyOn = True
	sleep(0.1)
except KeyboardInterrupt:
    print "Stopping."
```

In a nutshell: the first `if` checks whether more than `stayAliveTime` many seconds have passed since the last movement was registered, and turns off the display if enough time has passed && and the display is on && there's no motion right now.    
The second `if` covers the other case and checks for motion. Any motion updates the point in time `lastTimeActivated` where the last motion was detected: this way the display stays on if a person keeps moving in front of it. If it's currently off, then it'll also turn on.    

You'll want to install the `gpiozero` package and possibly some dependencies of it.

The `/home/pi/bin/turn-screen` script is quite simple:

```bash
#!/bin/bash
if (( $# < 1));
then
	echo "Pass '0' for 'off' and '1' for 'on'!"
	exit
fi
vcgencmd display_power $1
```

and that'll do it!
