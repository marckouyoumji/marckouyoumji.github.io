---
layout: page
title: "Smart Garage Project"
---
**Team Members: Bhavya Lokasani, Pari Goyal, Marc Kouyoumji, Yalaj Goyal**  
**Georgia Institute of Technology** 
 

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

{% include_relative main.cpp %}

