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

Thread t1;
Thread t2;

void lcdRestDisplay()
{
    lcdReset = true;
}

void lcdResetThread()
{
    while(1)
    {
        if(lcdReset)
        {
            lcd_mutex.lock();
            uLCD.cls();
            uLCD.locate(4,6);
            uLCD.color(WHITE);
            uLCD.printf("Welcome!!");
            uLCD.locate(2,8);
            uLCD.printf("Smart Garage");
            lcd_mutex.unlock();
            lcdReset = false;
        }
        Thread::wait(100);
    }

}

void speakerToggle()
{
    if(speaker != 0.0)
    {
        speaker = 0.0;
    }
    else 
    {
        speaker = 0.5;
    }
}

void speakerTimer()
{
    speaker = 0.0;
    speakerSound.detach();
}

void light_trigger()
{
    relay = 0;
}

void access_light()
{
    ledgreen = 0;
    ledred = 0;
}


void bluetoothRead()

{
    while(1) {
        if(blue.readable())
        {
            lcd_mutex.lock();
            pc.putc('s');
            if(blue.getc() == '!')
            {
                pc.putc('!');
                if(blue.getc() == 'B')
                {
                    pc.putc('B');
                    char button = blue.getc();
                    pc.putc(button);
                    if(logged_in)
                    {
                        switch(button)
                        {
                            case '2':
                            {
                                if(gate == 1.0)
                                {
                                    blue.putc('1');
                                    pc.putc('1');
                                }
                                else
                                {
                                    blue.putc('0');
                                    pc.putc('0');
                                }
                            }break;

                            case '3':
                            {
                                gate = 1.0;
                                light_switch.detach();
                                relay = 1;
                            }break;

                            case '4':
                            {
                                gate = 0.0;
                                light_switch.attach(&light_trigger,garageLedTimeout);

                            }break;

                            case '5':
                            {
                                char rx = blue.getc();
                                pc.putc(rx);
                                int i = 0;
                                while(rx != '\0')
                                {
                                    required_password[i] = rx;
                                    i++;
                                    rx = blue.getc();
                                    pc.putc(rx);
                                }
                                required_password[i] = rx;
                            }break;
                        }
                    }
                    if(button == '1')
                    {
                        char passChar = blue.getc();
                        pc.putc(passChar);
                        int i= 0;
                        while(passChar != '\0')
                        {
                            if(required_password[i] != passChar)
                                break;
                            i++;
                            passChar = blue.getc();
                            pc.putc(passChar);
                        }
                        if(i == strlen(required_password))
                        {
                            logged_in = true;
                            blue.putc('1');
                            pc.putc('1');
                            ledred = 0;
                            //LCD display says logged in
                            uLCD.cls();
                            uLCD.color(GREEN);
                            uLCD.locate(4, 6);
                            uLCD.printf("Logged In");
                            ledgreen = 1;
                            login_led.attach(&access_light,20.0);
                            //alarm = false;
                            speaker = 0.0;
                            speakerSound.detach();
                            alert.detach();
                            tries = 0;
                        }
                        else 
                        {
                            blue.putc('0');
                            pc.putc('0');
                            tries++;
                            //LCD display wromg password and tries remaining
                            uLCD.cls();
                            uLCD.color(RED);
                            uLCD.locate(4, 4);
                            uLCD.printf("Wrong Password!!");
                            uLCD.locate(2, 6);
                            uLCD.printf("Number of tries remaining: %d",3-tries);
                            if(tries == 3)
                            {
                                tries = 0;
                                ledred = 1;
                                login_led.attach(&access_light,20.0);
                                speaker = 0.5;
                                alert.attach(&speakerTimer,20.0);
                                speakerSound.attach(&speakerToggle,0.5);
                                //alarm =1;
                            }   
                        }
                    }
                    else if(button == '6')
                    {
                        logged_in = false;
                        ledgreen = 0;
                        ledred = 0;
                        light_switch.attach(&light_trigger,garageLedTimeout);
                        uLCD.cls();
                        uLCD.color(RED);
                        uLCD.locate(2, 6);
                        uLCD.printf("Disconnected!!");
                        lcdDisplay.attach(&lcdRestDisplay, garageLedTimeout);
                        if(gate == 1.0)
                        {
                            speaker = 0.5;
                            alert.detach();
                            alert.attach(&speakerTimer,10.0);
                            speakerSound.attach(&speakerToggle,0.5);
                            //display on LCD gate open
                            uLCD.locate(4, 8);
                            uLCD.printf("Gate Open!!");
                            uLCD.locate(2, 10);
                            uLCD.printf("Connect again");
                            uLCD.locate(4, 12);
                            uLCD.printf("and Close");
                        }
                        lcdDisplay.attach(&lcdRestDisplay, garageLedTimeout);
                    }
                }
            }
            lcd_mutex.unlock();
        }      
        //Thread::wait(1);
    }
}

int main()
{
    speaker = 0.0;
    gate = 0.0;
    relay = 0;
    uLCD.locate(4,6);
    uLCD.color(WHITE);
    //uLCD.text_width(1);
    //uLCD.text_height(2);
    uLCD.printf("Welcome!!");
    uLCD.locate(2,8);
    uLCD.printf("Smart Garage");

    t1.start(bluetoothRead);
    t2.start(lcdResetThread);
    
    while(1)
    {
        Thread::wait(1000);
    }
    
}



