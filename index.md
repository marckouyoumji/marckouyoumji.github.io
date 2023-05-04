---
layout: page
title: "Smart Garage Project"
---
**Team Members: Bhavya Lokasani, Pari Goyal, Marc Kouyoumji, Yalaj Goyal**  
**Georgia Institute of Technology** 
Watch our presentation:  
Presentation(5 min):

![IMG_2382](/assets/IMG_2382.jpg)

## Project Idea

The idea is to have an automated and secure smart parking garage-system.
It will have the ability to grant or deny access to vehicles trying to enter the parking space through a password. We also included automated lighting
that turns on if a presence is detected and turns off after fixed period of time. In addition to that, it will be possible for the owner to customize 
the application by changing the password and being able to modify the timer value


## Components

1. Mbed
2. Bluetooth module
3. LEDs
4. Relay
5. LCD
6. Servo
7. Speaker
8. Class D Amplifier
9. Lidar


## Schematic 

![block_diagram](/assets/block_diagram.jpg)


## Software

Here is the summary of how our software will work:

if(bluetooth readable)
	Button1: capture entered password;
		if(entered == required) -> logged_in = true ( green led = 1)
		Else -> tries++, red led =1 if tries > 3, alarm = 1
	if(logged_in)
		Button2: Send status of gate to BT
		Button3: Open gate, turn on garage lights
		Button4: Close gate, turn off garage lights after set time
		Button5: Change password
		Button6: Disconnect Bluetooth, if gate open -> alarm = 1
    
As seen in the pictures below, here is part of the user interface that will be displayed to our user:

![photo1](/assets/photo1.jpg)

![photo2](/assets/photo2.jpg)


## Source code

main.cpp for mbed:

```cpp
#include "Mutex.h"
#include "Servo.h"
#include "mbed.h"
#include "rtos.h"
#include "SDFileSystem.h"
#include "wave_player.h"
#include "uLCD_4DGL.h"
#include "XNucleo53L0A1.h"
#include "XNucleo53L0A1.h"

Serial blue(p13,p14); //tx,rx
SDFileSystem sd(p5, p6, p7, p8, "sd"); //SD card
Serial pc(USBTX, USBRX);
static bool logged_in = false;
static bool lcdReset = false;
static char required_password[9] ="00000000";
static int tries = 0;
Servo gate(p24);
DigitalOut ledgreen(p16);
DigitalOut ledred(p17);
DigitalOut relay(p30);
PwmOut speaker(p25);
Timeout login_led;
Timeout alert;
Timeout light_switch;
Timeout lcdDisplay;
Ticker speakerSound;
float garageLedTimeout = 3.0;
Mutex lcd_mutex;
uLCD_4DGL uLCD(p9, p10, p7);
