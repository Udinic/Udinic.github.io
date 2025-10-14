---
author: udinic
date: 2021-01-13 09:30:00
layout: post
slug: automating-balcony-shade-with-nfc
title: Automating my balcony shade... with NFC!
categories:
- Home Automation
- Electronics
tags:
- electronics
- NFC
- PN532
- Arduino
- ESP32
- Smart Shade
- Shade Controller
- Motorized shade
- Smart blinds
- Google Assistant
- Alexa
- IFTTT
- shade presets
- Servo motor
- high torque
- crank automation
- MEGA 2560
- LiPo battery
- buck converter
- Battery Eliminator Circuit
- NFC Tags
- I2C
- IRQ
- Adafruit PN532
- Waterproof buttons
- EasyEDA
- PCB
- Voice command
- WiFi
- Web Service
- MQTT
- Mosquitto
- Blynk
- Adafruit IO
- Step Drill
- ServoCity
yt_demo: f4yYC6kvG9Y
yt_motor_test: FIFcKEm9gtY
yt_prototype_test: rQhZ0Q_558M
yt_nfc_devenv: 5fkpSk1L73I
yt_nfc_test: pWtwcLzs28E
yt_voice_command: bzwZ1PaiDjU 
yt_demo_full: lLbN8Sc2E9Q
---

These past few months I spent mostly at home, as everyone else, and decided to take on a project to improve my time there. With three kids at our cozy apartment, it’s no wonder I loved spending a lot of time on our balcony. While I like the breeze and the view, I hated lowering and raising the sun shade. It’s a heavy shade, so it took more effort and time to operate than a regular bedroom window shade/blinds. I challenged myself to automate this shade by any means necessary!

I found a strong motor, designed a circuit and programmed a microcontroller to run the show. I mounted a switch box with up/down buttons and used NFC to support "shade presets". Finally, I got everything connected to the internet and respond to voice commands from Google Assistant. The result:

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/intro-circuit.jpg">
		<img src="/assets/images/{{ page.slug }}/intro-circuit.jpg"/> 
	</a>
</div>

<br />

<div class="media-center">
{% include youtubePlayer_v.html id=page.yt_demo %}
</div>
<div class="image-caption">
	Demo of the smart shade controller
</div>

That was quite a journey.

I dived into the world of Arduino, where I learned how to design and program circuits. I also gained a better grasp on mechanical engineering concepts and got some experience applying them in real life. At the end, I had a polished Shade Controller that works reliably and safe for everyone to use, including my kids.

In this post I wanted to share my experience __building a smart shade controller from scratch__. Here's an overview of the steps I'll cover:
- Finding a suitable motor and figuring out how to attach it to the shade.
- Building the circuit and testing it in my balcony.
- Taking automation to the next level by using NFC (Near-Field Communication).
- Improving the appearance of the shade controller.
- Supporting voice commands using Google Assistant.
- Tips and lessons learned.

I think the creation process was a cool experience that’s worth the read on it’s own, but I also hope that some of my methods, research and learnings will help others undertake similar projects.

<br />

## Setting up the Motor

Building a custom smart shade/blinds is a pretty common project, but as I researched this subject I didn’t find a solution that fits the type of shade I have. In most of these examples, the motor was moving the blinds by pulling a chain or rotating a small twist shaft.

<div class="outer-grid">
	<div class="inner-grid">
  		<a href="http://www.roberttrescott.com/shadesblinds.html">
  			<img src="/assets/images/{{ page.slug }}/other-shaft.jpg"/> 
  		</a>
	</div>
  	<div class="inner-grid">
		<a href="https://github.com/vietquocnguyen/NodeMCU-ESP8266-Servo-Smart-Blinds">
  			<img src="/assets/images/{{ page.slug }}/other-ballchain.jpg"/>
  		</a>
  		<a href="http://energydefender.blogspot.com/2010/02/arduino-controlled-electric-blinds.html">
  			<img src="/assets/images/{{ page.slug }}/other-twist.jpg" style="padding-top: 8px;"/>  	
  		</a>
  	</div>
</div>
<div class="image-caption">
	Examples for other smart shade projects. Click on each image to see the relevant project.
</div>


The shade I installed in my balcony is much bigger and heavier than most of these shades. It’s 8 feet wide, weighs 8 pounds and uses a crank and a wand to operate.

<div class="media-center">
	<a href="https://www.amazon.com/gp/product/B00IWMLEZS/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1">
		<img src="/assets/images/{{ page.slug }}/shade-amazon.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The shade I have on my balcony
</div>

To start, I need to find a strong motor that can actually move this shade. There are ways to measure the force needed to move the shade (like using a [luggage scale](https://www.engineeringtoolbox.com/torque-wrench-luggage-scale-d_1909.html)), but I ended up consulting my mechanical engineer brother, who helped me pick a motor that should be strong enough for the job. I found a [Servo motor](https://www.servocity.com/sg12-series-servo-gearbox-2-1-ratio-continuous-rotation-700-oz-in-31-3-rpm/) that's strong by itself, but it also comes inside a gearbox that increases its torque (and I could use all the torque I can get!). The gearbox has 2 gears in a 2:1 ratio, which produces half the speed of the original servo motor, but with twice the torque. To close or open the shade we need to rotate the crank 60 times, so with the 31 RPM the motor can do - it’ll take about 2 minutes. That’s reasonable enough.

Now that I have a motor, I needed to find a way to mount it on the roller and get it to rotate the crank. I took measurements and started researching…

<div class="outer-grid">
	<div class="inner-grid">
		<img src="/assets/images/{{ page.slug }}/measure1.jpg"/> 
	</div>
	<div class="inner-grid">
		<img src="/assets/images/{{ page.slug }}/measure2.jpg"/> 
	</div>
	<div class="inner-grid">
		<img src="/assets/images/{{ page.slug }}/measure3.jpg"/> 
	</div>
</div>
<div class="image-caption">
	Taking measurements
</div>


Connecting the motor to the crank and rotating it was also not trivial. I took some measurements, but I don't have a 3D printer so I can't just print a perfect adapter. Luckily, after some online searching I found a [bracket](https://www.servocity.com/lightweight-linear-actuator-mounting-bracket/) that fits perfectly! To celebrate, I did an initial test to verify the motor can actually move the shade in both directions:

<div class="media-center">
{% include youtubePlayer_v.html id=page.yt_motor_test %}
</div>
<div class="image-caption">
	blabla
</div>

A successful mount of the motor needs to produce a smooth crank rotation while the motor spins, any misalignment between the rotation planes of the motor and the crank could cause friction that damages the shade's mechanism or the motor. Getting this alignment right was found to be quite challenging.

Since I don't have a 3D printer, I had to improvise a setup with existing structural components. I found [ServoCity](https://www.servocity.com/), a well-organized website that sells parts for projects like this. On the site, I found and ordered a bunch of different structural components that might help me. I used various metal plates and brackets to mount the motor in different ways, testing the crank's rotation and making adjustments. After some trial and error, and a bunch of new holes in my wall, I eventually found an arrangement that worked.

<div class="outer-grid">
	<div class="inner-grid">
		<a href="/assets/images/{{ page.slug }}/mount1.jpg">
			<img src="/assets/images/{{ page.slug }}/mount1.jpg"/> 
		</a>
	</div>
	<div class="inner-grid">
		<a href="/assets/images/{{ page.slug }}/mount2.jpg">
			<img src="/assets/images/{{ page.slug }}/mount2.jpg"/> 
		</a>
	</div>
</div>
<div class="image-caption">
	Mounting the motor. First attempt on the left, final position on the right.
</div>

Cool, now that I have the motor in place - I need to build the circuit that controls it. 

<br />

## Building the Shade Controller Circuit

My initial requirement for the shade controller was simple: have 2 buttons that can spin the motor in both directions. That was a great excuse to start tinkering with Arduino, something I wanted to do for a while.  So I dived right in.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/arduino-simple.jpg">
		<img src="/assets/images/{{ page.slug }}/arduino-simple.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Simple circuit to spin the motor in both directions.
</div>

The 2 buttons are connected to the Arduino through the breadboard, each one is connected to a different pin on the Arduino board. The code constantly reads the value of these pins to detect when a button was pressed. By default, each pin returns the value HIGH but during the time a button is pressed the pin's value will be LOW. The code ignores long presses and acts only when a button changes state, which is why we compare the current pin's value to the previous one.

{% highlight cpp %}
#include <Servo.h> 

// Pin definitions
#define PIN_SERVO (13)
#define PIN_UP_BTN (5)
#define PIN_DOWN_BTN (15)

Servo myServo;

// ...

void setup() {
  // Setting up the button with the pins they're wired to
  pinMode(PIN_UP_BTN, INPUT_PULLUP);
  pinMode(PIN_DOWN_BTN, INPUT_PULLUP);
}

// Main function that loops as long as the controller is powered on
void loop() {
  // Reading the value in the pins connected to the buttons.
  // The default read will be HIGH, while pressing a button it's changed to LOW
  btnUpCurr = digitalRead(PIN_UP_BTN);
  btnDownCurr = digitalRead(PIN_DOWN_BTN);

  // The UP button is pressed 
  if (btnUpCurr == LOW && btnUpPrev == HIGH) {
    // If the motor is already spinning - stop it
    if (raisingShade || loweringShade) {
      stopShade();
    } else {
      raiseShade();
    }
  }  

  // The DOWN button is pressed  
  if (btnDownCurr == LOW && btnDownPrev == HIGH) {
    if (raisingShade || loweringShade) {
      stopShade();
    } else {
      lowerShade();
    }
  }

  btnUpPrev = btnUpCurr;
  btnDownPrev = btnDownCurr;
}

{% endhighlight %}


The Servo motor is also connected to one of the Arduino's pins, using the yellow/white wire. Controlling the motor is done using an API that's part of a Servo library.

{% highlight cpp %}

void lowerShade() {
  myServo.attach(PIN_SERVO);
  // Value smaller than 1500 will spin the motor counter-clockwise
  myServo.writeMicroseconds(850);  
  loweringShade = true;
  raisingShade = false;
}

void raiseShade() {
  myServo.attach(PIN_SERVO);
  // Value larger than 1500 will spin the motor clockwise
  myServo.writeMicroseconds(2100);  
  raisingShade = true;
  loweringShade = false;
}

void stopShade() {
  raisingShade = false;
  loweringShade = false;
  myServo.detach();
}

{% endhighlight %}

Internally, the Servo library is controlling the motor by sending it signals in "different formats", which contain the position and direction the motor should be spinning. For more on that, check out [how Pulse-Width Modulation works](https://en.wikipedia.org/wiki/Servo_control).

After getting this simple circuit to work, I decided to replace the Arduino board with a smaller microcontroller - the ESP32.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/esp32-comparison.jpg">
		<img src="/assets/images/{{ page.slug }}/esp32-comparison.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Arduino vs. ESP32
</div>

Arduino is an open source hardware and software for electronics hobbyists. The board I have, the MEGA 2560, is Arduino-compatible. It was not manufactured by the Arduino company but it did use the same open source Arduino hardware design. This means I can use the Arduino IDE to program my board and utilize the great variety of Arduino libraries to integrate with external sensors and peripherals. The ESP32 hardware was not built by the same Arduino specification, but its software bridges the gap between the 2 hardware designs. This means we can program the ESP32 almost exactly the same as an Arduino board or an Arduino-compatible board, even though they are very different.

Why choose ESP32 over an Arduino board? Well, Arduino is great for beginners because it’s simpler to use and easier to get started. However, the ESP32 is more powerful, smaller, cheaper and supports Bluetooth and WiFI, which the Arduino doesn’t. To make it easier to work with, the ESP32 chip comes integrated in a small dev-board that has features like micro-USB connector and voltage regulator, as well as easier access to all its pins (like an Arduino board has).

I wanted my circuit to have a small footprint and I also had plans for using the WiFI feature, so I bought a couple of [ESP32 dev boards](https://www.amazon.com/gp/product/B07Q576VWZ/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1). I migrated my code to be compatible with ESP32, which basically was just updating to an ESP32-flavored Servo library.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/esp32-breadboard.jpg">
		<img src="/assets/images/{{ page.slug }}/esp32-breadboard.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Using an ESP32 to control the motor.
</div>

I also updated the circuit to power the motor and the ESP32 from a single battery, but more on that later.

Now that I have the basic circuit for my shade controller, I can start testing it at my balcony. So far I tested my circuit on a breadboard, but I want something more reliable for my first prototype. I took out my soldering iron and built the circuit on a perfboard.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/soldering.jpg">
		<img src="/assets/images/{{ page.slug }}/soldering.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Soldering the components for the shade controller prototype.
</div>

To test it, I hung the shade controller circuit next to the motor with zip-ties and connected the motor to it. The battery is also there at the back, inside a ziplock bag. Nothing too fancy for a first prototype.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/prototype-combina-big.jpg">
		<img src="/assets/images/{{ page.slug }}/prototype-combina-big.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The first shade controller prototype.
</div>

I also added a 2-button remote, for a better accessibility when trying to operate the shade from my balcony chair.

Now that everything is in place, I can test the first prototype:

<div class="media-center">
{% include youtubePlayer_v.html id=page.yt_prototype_test %}
</div>
<div class="image-caption">
	Testing the prototype
</div>


<br />

## The First Prototype
It was a real treat using my prototype for a couple of weeks! The shade controller worked well - it was responsive and the motor raised that shade like a champ! There was only one problem I found, but before diving into that - let’s first get to know the prototype a little better.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/diagram-prototype.jpg">
		<img src="/assets/images/{{ page.slug }}/diagram-prototype.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Diagram of the shade controller prototype circuit.
</div>

In the diagram on the right there’s the remote, with 2 buttons - up and down. Like we saw earlier in the basic circuit, each button is responsible to spin the motor in a different direction (i.e. clockwise/counter-clockwise). The motor receives the commands from the ESP32 using the cyan/yellow wire.

The circuit is powered by a single 7.4v battery. The motor can run with 5v, but it’ll run faster and provide more torque with a higher voltage. According to its [spec](https://www.servocity.com/sg12-series-servo-gearbox-2-1-ratio-continuous-rotation-700-oz-in-31-3-rpm/), the motor's safest maximum voltage is 7.4v, which is why I picked a 7.4v battery for the job. The motor is connected directly to the battery, but the ESP32 is more sensitive and could be damaged by voltage too high. To be on the safe side, I added a [DC step-down convertor](https://www.adafruit.com/product/1385) (a.k.a Buck Converter or BEC - Battery Eliminator Circuit) that will convert the battery’s 7.4v to a steady 5v that will power the ESP32.

The ESP32 chip itself is powered using 3.3v, but the dev-board it comes with has a built-in voltage regulator that can take a higher voltage and step it down to the needed 3.3v. That’s why it’s possible to power it via micro-USB, which provides 5v. When using an ESP32 dev-board, I can either connect a 3.3v power supply to the 3v3 pin, or I can connect 5v to the VIN pin which will first go through the built-in regulator. In the diagram, the DC step-down converter is connected to the battery on one side, where it receives 7.4v, and outputs 5v on the other side, which is connected to the VIN pin using the red wire.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/prototype-mounted-big.jpg">
		<img src="/assets/images/{{ page.slug }}/prototype-mounted-big.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The prototype with the battery powering it.
</div>

After using the prototype for some time, I noticed a problem - the __battery life isn't great__, a full battery lasted only 5 days.

I got a pretty strong battery, a 6200mAh LiPo 7.4v battery, which is intended for Remote Control race cars. The motor draws between 200mA-3000mA while running, which the battery can handle as it's designed for operating strong race car motors, but the majority of time the circuit is on standby. While on standby, the ESP32 is powered on and waiting for input from one of the buttons, which consumes about 50mA. The battery's capacity is 6200mAh, meaning it can power a 50mA device for 124 hours (6200 / 50 = 124), which is a little over 5 days. This aligns with what I saw in practice and explains why the battery doesn't last that long.

I decided that the best solution is to simply ditch the battery. Why? I can't see myself optimizing the battery consumption in a significant way. In addition, even if the battery will last a few weeks or a month, I still need to recharge it from time to time and risk having a dead battery when I need the shade. I don’t want to think about it, I want it to __Just Work__.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/circuit-barreljack.jpg">
		<img src="/assets/images/{{ page.slug }}/circuit-barreljack.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Connecting the circuit to a power adapter.
</div>

To support a power adapter, I added a [barrel-jack port](https://www.adafruit.com/product/373?gclid=Cj0KCQiArvX_BRCyARIsAKsnTxOEqPoVS5H8MBUYNwGpgpWzCSj7SRv3-nlrttM9-tQ5JrCaKR-fR9EaAu2mEALw_wcB) to my circuit and got a power adapter that supplies 7.5v. A pretty simple modification that means I don't need to worry about sitting in the sun because I forgot to charge the battery in time.

That was the first modification I made to the shade controller’s circuit, but it was just the beginning. In the next section, I’ll go over a new feature for my shade controller that would require bigger changes, including communicating with an external chip.

<br />

## Supporting Automatic Stops Using NFC
At this point of the project I got the minimal features for a fully working motorized shade, but I wanted to take this a step further. It takes a single button press to start lowering/raising the shade, but I still need to wait and press the button again in order to stop it.

In most cases, I want to fully lower the shade or fully open it, so it makes sense the controller should know when to stop instead of waiting for me to do it manually. In addition, the shade I have installed in my balcony is larger than the area it needs to cover, so I need to set clear boundaries and prevent lowering it too low or raising it higher than necessary. Otherwise, the motor or the shade’s roller could be damaged, or worse - my downstairs neighbor will come over to complain about the shade dangling above his head. By having a reliable way to know the shade’s position, I can __program the shade controller to stop automatically when reaching a specific position__.

There are several ways this can be accomplished. One way could be calibrating the controller with the time it takes to fully lower / raise the shade, then use this time to know how long the controller should spin the motor  before stopping it. This method is simple to implement but also unreliable, since the duration of lowering/raising the shade could skew over time and require frequent calibrations (e.g. strong winds could pull the shade and create resistance to the motor, which slows it down). Another option is to stick 2 magnets on the shade itself, one at the top and the other at the bottom, then use a [Hall effect sensor](https://www.electronics-tutorials.ws/electromagnetism/hall-effect.html) to detect these magnets and stop the motor from rolling the shade past these points. While this could work for simple use cases, it won’t prevent moving the shade past the boundaries the magnets created. For example, pressing “down” will lower the shade until the magnet is reached, but afterwards I can press “down” again to lower the shade further, past the lowest position allowed. 

I decided to try an interesting approach - using NFC to read position markers on the shade. The problem with the magnets approach is the fact they aren’t distinguishable, so the shade controller can't know which magnet is caught by the Hall effect sensor. Using an NFC reader, the shade controller can read NFC tags placed on the shade in different places and decide whether to stop or continue raising/lowering the shade. The NFC tags are distinguishable by their ID, but they also allow writing data in them, which can be used for storing in each NFC tag the position it represents (e.g. "100% open", "50% open").

The PN532 is the most popular NFC/RFID chip. Pretty much all the mobile phones who support NFC use this chip, or other chips from the same company, to do things like wireless payments or sharing data between nearby devices. When I started tinkering with NFC I got the [Adafruit PN532 breakout board](https://www.adafruit.com/product/364), but I later got me a [much smaller PN532 module](https://www.amazon.com/gp/product/B01I1J17LC/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1), which works the same and will be easier to mount next to the shade controller.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/pn532.jpg">
		<img src="/assets/images/{{ page.slug }}/pn532.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The PN532 NFC Module.
</div>

There are several ways to connect peripherals to microcontrollers such as the ESP32 or Arduino. A peripheral can be a temperature sensor, LCD display or an external NFC module such as the PN532. Both Arduino and ESP32 support 3 common communication protocols - UART, SPI and I2C. Each protocol has its own benefits/drawbacks and picking which one to use really depends on the use case. SPI for example is much faster than I2C, which means it’s probably better for communicating with LCD displays or sensors that are sampled frequently, but with I2C we can connect a lot more external devices at once. 

I decided to connect the PN532 NFC module using I2C ( = inter-integrated circuit), because it’s a simpler connection but also for a reason I’ll cover a little later. Here’s a diagram showing how everything is connected:

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/diagram-final.jpg">
		<img src="/assets/images/{{ page.slug }}/diagram-final.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Diagram of the main circuit connected to the PN532 NFC module.
</div>

There are 4 main wires necessary to connect the PN532 to the ESP32. First the power pins, VCC and GND, are connected using the blue and green wires to the 3v3 and GND pins on the ESP32. These will give the PN532 the 3v it needs to be powered. Next are the communication pins - for the I2C protocol we need to connect the SCL (clock line) and SDA (data line) pins to the equivalent pins on the microcontroller, which for the ESP32 are the D22 and D24 pins. In the diagram, these are represented by the white and purple wires. The orange wire is connected to the RESET pin, it’s optional and used to reset the device if needed. The yellow wire is connected to the IRQ pin, which will be discussed a bit later.

There’s another connection we need to make for the I2C interface to work properly - the SCL and SDA pins also need to be connected to the power line (i.e. the 3v3 pin). This is necessary due to the open drain design of I2C - during data transfer the SDA/SCL lines are pulled low, which is a problem if other devices also try to communicate. To make I2C work, we need to keep these lines high by allowing power from the 3v3 pin to come through. But, the power can be too strong, which is why I had to connect 1.5kΩ resistors per line (I didn’t have a single 1.5kΩ resistor, so I just connected 3 that totaled in 1.5kΩ). There’s more discussion on this subject [here](https://forums.adafruit.com/viewtopic.php?f=19&t=88591).

Writing the code to interact with the NFC chip was pretty straightforward using the [Adafruit PN532 library](https://github.com/adafruit/Adafruit-PN532) - there's an API method that starts listening for cards/tags, waits for a card to come close to the reader and reads its data when that happens. It looks simple, but there’s one major problem for my use-case - the command that starts listening for NFC cards __is a blocking call__. This means my microcontroller, the ESP32, is essentially stuck waiting for NFC tags and can’t respond to other inputs, such as button presses. Here’s how this looks in the code:

{% highlight cpp %}
// Pin definitions
#define PN532_IRQ   (2)
#define PN532_RESET (4)

// ...

void loop() {

  // ...

  // The DOWN button is pressed  
  if (btnDownCurr == LOW && btnDownPrev == HIGH) {
    if (raisingShade || loweringShade) {
      stopShade();
    } else {
      lowerShade();

      // Wait for the NFC module to detect a card or a tag and read it.
      // This is a BLOCKING call.
      success = nfc.readPassiveTargetID(PN532_MIFARE_ISO14443A, uid, &uidLength);

      if (success) {
        // Get card ID and check if we need to stop the motor
      }
    }
  }

  // ...
}
{% endhighlight %}

For example, say I press the “down” button to lower the shade all the way down. While the motor is running, the ESP32 will activate the NFC module and wait for the NFC tag that signals the end of the shade. But, during this time the buttons aren't responsive, which means there's nothing I can do to stop lowering the shade before it reaches the end.

I needed to find a way to make the microcontroller wait for NFC tags and __still be responsive__ to new commands, but there were no API methods to help with that. I saw others use a second ESP32 with the sole purpose of waiting for NFC cards, then communicating that to the first ESP32. That's probably an overkill.. I decided to dig a little deeper into the NFC module's spec sheet and find a better solution.

According to the [PN532 user manual](https://www.nxp.com/docs/en/user-guide/141520.pdf), we can use the chip’s IRQ pin in order to be notified asynchronously when a card is detected. The IRQ line is an optional connection we can make between the host controller (i.e. The ESP32) and the PN532 NFC module. When everything is connected and working, the PN532 will continuously send high voltage in the IRQ line, which essentially means we'll get a binary 1 value whenever we read from that pin. When the PN532 has some response it wants to notify the host controller about (e.g. new NFC card/tag was detected) - it momentarily drives this line low ( = sends a binary 0 instead of 1 for a short amount of time). 

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/pn532-sequence.jpg">
		<img src="/assets/images/{{ page.slug }}/pn532-sequence.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Sequence diagram showing the changes in the IRQ line when messages are sent and received between the ESP32 and PN532.
</div>

I connected the IRQ pin using the yellow wire in the diagram above, then updated my code to keep track of its value. The ability to use the IRQ line is the reason I chose the I2C protocol over SPI and UART - the Adafruit PN532 library supports the IRQ pin only when connected using I2C. However, this library didn't include a non-blocking API to put the NFC module in a card detection state. So.. I just added new API methods myself! Here's the updated code:

{% highlight cpp %}
// API for the NFC module (PN532 chip)
Adafruit_PN532 nfc(PN532_IRQ, PN532_RESET);

// Supported NFC tags for each shade position (0%, 100% etc.)
const uint32_t CARDID_100_PERCENT = 3952824665;
const uint32_t CARDID_0_PERCENT = 913822226;

// ..

void setup() {
  pinMode(PN532_IRQ, INPUT_PULLUP);

  // ...
}

// Main function that loops as long as the controller is powered on
void loop() {
  irqCurr = digitalRead(PN532_IRQ);

  // ...

  // The DOWN button is pressed
  if (btnDownCurr == LOW && btnDownPrev == HIGH) {
    if (raisingShade || loweringShade) {
      stopShade();
    } else {
      lowerShade();

      // Puts the NFC module on a listening mode, a NON-BLOCKING call
      nfc.startPassiveTargetIDDetection(PN532_MIFARE_ISO14443A);
      listeningToNFC = true;
    }
  }

  // If the NFC module is currently listening and we’re notified via the
  // IRQ line that a card was detected
  if (listeningToNFC && irqCurr == LOW && irqPrev == HIGH) {
    // Read the detected NFC tag's info
    success = nfc.readDetectedPassiveTargetID(uid, &uidLength);

    if (success) {
      // If the tag is valid, we'll read its ID
      uint32_t cardId = getCardId(uid, uidLength);
      if (cardId == CARDID_0_PERCENT) {
        if (loweringShade) {
          // Found the "0% open" position tag while lowering the shade, 
          // we can stop because the shade is now fully closed
          stopShade();
          listeningToNFC = false;
        } else if (raisingShade) {
          // Ignoring.. 
        }
      } else if (cardId == CARDID_100_PERCENT) {
        // Similar implementation as above.
        // Stop raising the shade when the CARDID_100_PERCENT tag is detected, 
        // since the shade is now fully open.
      }
    }

    // If we didn't stop the motor and still need to listen to NFC tags
    if (listeningToNFC) {
      delay(500);

      // Puts the NFC module on a listening mode again
      nfc.startPassiveTargetIDDetection(PN532_MIFARE_ISO14443A);
    }
  }
    // ...
}
{% endhighlight %}

Using the new API, I put the NFC module in a detection mode without blocking. During this time, if the IRQ value changes from HIGH to LOW - we know a card/tag was detected and it's time to try reading it. Each NFC tag represents a shade position (e.g. 0% opened), which will help determine whether the motor needs to be stopped. We can attach more NFC tags to the shade and support more "shade presets", like 50% or 25% open.

The new API methods I added were merged into the PN532 library, now everyone can use them! There’s more info in the  [pull-request I made on Github](https://github.com/adafruit/Adafruit-PN532/pull/82).

With these changes, the up/down buttons are always responsive and the motor can be stopped either by pressing any of the buttons or when the NFC module is reading the appropriate NFC tag. The next video demonstrate this in my development environment, note how I use a different NFC tag to stop each spin direction:

<div class="media-center">
{% include youtubePlayer_h.html id=page.yt_nfc_devenv %}
</div>
<div class="image-caption">
	Testing the new NFC-based auto-stop mechanism
</div>

By the way, I could’ve used another Arduino feature called [Interrupts](https://learn.sparkfun.com/tutorials/processor-interrupts-with-arduino/all) to listen and react to changes in the IRQ line. While interrupts might seem as a more elegant implementation, they also cause disruptions to the execution flow which made the shade controller unreliable in some situations. With more effort, I might be able to get interrupts to work for me, but for now I chose the approach above for its flexibility and consistency.

To test this NFC-based positioning system on my balcony shade, I need to put it close to the shade. For that, I got a plastic sheet [from Amazon](https://www.amazon.com/gp/product/B07MG8KTBX/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&th=1) to hold the NFC module on one side and supports the shade on the other. The shade rests on this plastic sheet as it moves up or down, allowing the NFC module to read the NFC tag that’s on the shade. In my first test, I attached an NFC tag on the shade using 2 of my daughter’s hair clips (I couldn’t find a stapler ¯\\\_(ツ)\_/¯ ), and verified the shade controller stops the motor after detecting the NFC tag.

<div class="media-center">
{% include youtubePlayer_h.html id=page.yt_nfc_test %}
</div>
<div class="image-caption">
	Testing the shade controller auto-stops when reaching the end.
</div>

The NFC module can read tags that are up to 2 inches away, which is more than enough for my setup, as long as the reader is aligned with the NFC tag attached to the shade. 

Supporting automatic stops is a big step up for my shade controller. This new feature is making the shade controller more user friendly, robust and reliable. It’s also the foundation for the next major feature - voice command.

<br />

## Hardware facelift
My shade controller is more automatic now, but how about making it look more professional? It’s a little too fragile (especially with children around), not weather resistant and looks kinda hacky. Before adding more software features, I decided to spend some time polishing the hardware side, making it more user friendly and sustainable.

As a first step, I created a switch box to replace the remote I created earlier.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/improve-btns.jpg">
		<img src="/assets/images/{{ page.slug }}/improve-btns.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The old remote alongside the new waterproof switch box.
</div>

I drilled a couple of holes into a waterproof plastic box, where I installed the 2 waterproof buttons. I organized all the wires inside cable channels to protect them from the rain, and more importantly - my kids. Besides the robustness of the new switch box, it's also a lot nicer than that hacky-looking remote.

The main circuit used to sit inside its own box next to the motor, but that was before I added the NFC module. For a cleaner look, I put both of them in the same box and mounted it on that plastic sheet the shade is resting on. The NFC module is still positioned close enough to read NFC tags from the shade.

While I was doing that, I figured I could reduce the footprint even further by trimming the plastic sheet, since I don't really need it to be this big. I took a hacksaw and removed a significant chunk of it.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/improve-main-before.jpg">
		<img src="/assets/images/{{ page.slug }}/improve-main-before.jpg"/> 
	</a>
</div>
<div class="image-spacer"></div>
<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/improve-main-after.jpg">
		<img src="/assets/images/{{ page.slug }}/improve-main-after.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Before and after the changes to the main circuit housing.
</div>

Now I have the same shade controller, just better looking and with a much smaller footprint. This will be useful when I’ll need to explain all this to my wife

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/improve-wall.jpg">
		<img src="/assets/images/{{ page.slug }}/improve-wall.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Now that looks a lot nicer!.
</div>

As one last improvement, I wanted to polish my circuit and create a PCB (Printed Circuit Board) to replace the perfboard I was using this far. Perfboards are a great way to quickly create and iterate on a prototype; they are less flexible than breadboards, but more sustainable and reliable. Now that I’m done with hardware changes, at least for the near future, a PCB will look a lot nicer and reduces the amount of loose wires that could be accidentally damaged.

I never designed PCBs before, but it was simpler than I thought. I found [EasyEDA](https://easyeda.com/), which is an online tool (and a Mac app) for designing PCBs and also ordering them directly from the tool. I started by creating the schematic design of my circuit:

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/easyeda-schematic.jpg">
		<img src="/assets/images/{{ page.slug }}/easyeda-schematic.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The schematic design of the shade controller circuit.
</div>

The schematic is then used by EasyEDA to generate a PCB design. The design has all the components connected to each other, so I just moved components around to arrange them a little better, added labels and other small tweaks. Here’s what I came up with:

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/easyeda-pcb.jpg">
		<img src="/assets/images/{{ page.slug }}/easyeda-pcb.jpg"/> 
	</a>
</div>
<div class="image-spacer"></div>
<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/pcb-empty.jpg">
		<img src="/assets/images/{{ page.slug }}/pcb-empty.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The PCB design in EasyEDA and the PCB I received.
</div>


The red and blue lines, in the PCB design, are the wires that connect the different components on the PCB and are generated based on the circuit's schematic design. The reds are on the top layer of the PCB and the blues are on the bottom layer. This layer separation is necessary because the wires cannot cross each other and there isn’t always a way around to get the wire where it needs to go. Complex circuits might require moving components around in order to allow all the wires to be connected. There’s also an option to add images and text to the PCB, so I added a small logo and versioning information.

EasyEDA has the option to directly order the PCB from [JLCPCB](https://jlcpcb.com/), a Chinese company that manufactures PCBs. I picked the color I wanted (there are 6 options!) and paid the $2 the PCBs cost (+ $16 shipping to the US), then waited about a week until they arrived (min. of 5 PCBs per order). After some quality time with my soldering iron, I got me a slick new shade controller circuit:

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/pcb-migration-front.jpg">
		<img src="/assets/images/{{ page.slug }}/pcb-migration-front.jpg"/> 
	</a>
</div>
<div class="image-spacer"></div>
<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/pcb-migration-back.jpg">
		<img src="/assets/images/{{ page.slug }}/pcb-migration-back.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The old circuit compared to the PCB I designed.
</div>

Improving the look and feel of a product is not just a cosmetic change best saved for last, it can be an important step in the testing phase. With these improvements, I made the shade controller more accessible and user friendly. I feel more confident letting my kids use it now, which might expose usage problems I didn’t see before. Plus, it’s nice to clean up a bit before moving forward, it’s more fun to use a polished product and it helps seeing potential problems more clearly. 

<br />

## Supporting Voice Commands via Google Assistant
One of the reasons I switched very early on from an Arduino board to ESP32, is the WiFi support. This opens up new possibilities for my shade controller, the main one I was after is supporting Google Assistant and responding to voice commands. This is the change that will upgrade the shade controller to be a “smart shade controller” 😎.

There are a couple of ways I could use the WiFi support for controlling my circuit remotely. With a web server running on the ESP32, I can connect clients directly to it and communicate over HTTP requests/responses. The web server can be accessible on the internet or by a direct connection ([example project](https://randomnerdtutorials.com/esp32-client-server-wi-fi/) that connects two ESP32s to each other without internet). While creating a web server on the ESP32 is a powerful feature, it has a few disadvantages such as a large overhead per message, speed and battery performance.

Another way to communicate with an ESP32 via WiFi is by using MQTT, a lightweight messaging protocol. MQTT is more performant and lightweight than HTTP, it also provides a simpler two-way communication mechanism between the ESP32 and its clients. For these reasons, MQTT is the preferred method for communicating with IoT devices. To use MQTT we first need an MQTT broker, which is a fancy name for an MQTT server. Any message sent to the broker needs to include a “topic”, then only clients subscribed to that topic will receive that message.

To set up MQTT for my shade controller, I can create an MQTT server in my local network (using [Mosquitto](https://mosquitto.org/)) or use a cloud service such as [Blynk](https://blynk.io/) or [Adafruit IO](https://learn.adafruit.com/welcome-to-adafruit-io). My needs are very simple and the free tier of a cloud service should suffice. I picked Adafruit IO, set up my account and added a new feed (i.e. MQTT Topic) for my project, called "shade-open". In the web console, I can create a dashboard and see the latest value sent to that feed.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/adafruitio-dashboard.jpg">
		<img src="/assets/images/{{ page.slug }}/adafruitio-dashboard.jpg"/> 
	</a>
</div>
<div class="image-caption">
	My shade controller dashboard on Adafruit IO.
</div>

To integrate and use Adafruit IO in the shade controller, I added their Arduino library into my project and made some changes:

{% highlight cpp %}
#define IO_USERNAME  "< AIO Username >"
#define IO_KEY       "< AIO KEY >"

#define WIFI_SSID "< Wifi SSID >"
#define WIFI_PASS "< Wifi password >"

#include "AdafruitIO_WiFi.h"

AdafruitIO_WiFi io(IO_USERNAME, IO_KEY, WIFI_SSID, WIFI_PASS);

// Adafruit IO feed showing how much the shade should be open (either 0% or 100%)
AdafruitIO_Feed *balconyShade = io.feed("shade-open");

boolean connectingInProgress = false;

// …

void loop() {
  // Keep the connection alive, if already connected.
  // Times-out after 400ms to keep the microcontroller responsive.
  aio_status_t ioStatus = io.run(400, true);

  // Connect to Adafruit IO, if connected then subscribe to the feed
  if (ioStatus < AIO_CONNECTED && !connectingInProgress) {
    connectingInProgress = true;
    io.connect();       
  } else if (ioStatus >= AIO_CONNECTED && connectingInProgress) {
    connectingInProgress = false;
	
    // Subscribe to the "shade-open" feed
    balconyShade->onMessage(handleShadeLevelMessage);
  }
  // ...
}

// This function is called when a new message is sent to the “shade-open” feed.
void handleShadeLevelMessage(AdafruitIO_Data *data) {
  // Got a new message with the percent the shade should be open 
  int percentOpen = atoi(data->value());
  
  // If we're asked to open the shade to 100%, then raise it. Open to 0% means closing/lowering the shade.
  if (percentOpen == 100) {
    raiseShade();
  } else if (percentOpen == 0) {
    lowerShade();
  }
  // ...
}
{% endhighlight %}

The code connects to Adafruit IO cloud service and keeps the connection alive by calling the *io.run()* method in the main loop function. When a new message is posted to the “shade-open” feed, the callback function is called and sends the appropriate command to the motor.

I implemented the Adafruit IO integration a little bit differently than the examples they provide. In their [examples](https://github.com/adafruit/Adafruit_IO_Arduino/blob/master/examples/adafruitio_00_publish/adafruitio_00_publish.ino), the microcontroller is blocked from doing anything until it’s connected to the cloud. I didn’t like the fact I won’t be able to use the physical buttons to control the shade while the controller is trying to connect to the cloud (do I really need to sit in the sun if my network is down or very slow?!). The code I wrote is more responsive and allows the controller to work offline, while still trying to regain connectivity. I tested the integration worked by sending MQTT messages from the Adafruit IO web console and verified they were received by the ESP32.

Now let’s move to the final part - controlling the shade from Google Assistant. This was probably the easiest part of this entire project. The shade controller can already take commands from the web using MQTT, so if we want Google Assistant to control it - we need to connect the two.

[IFTTT](https://ifttt.com/adafruit) (If-this-then-that) is a great tool for connecting different web services and products to each other. It supports integrations with Google Assistance and  Adafruit IO, so all I needed to do is authenticate to both services and create rules (called “Applets”) to communicate between them. For my use case, I created an applet that reacts to a Google Assistant command to open or close the shade and sends data to a specific feed on my Adafruit IO account.

<div class="outer-grid">
	<div class="inner-grid">
		<a href="/assets/images/{{ page.slug }}/ifttt1.png">
			<img src="/assets/images/{{ page.slug }}/ifttt1.png"/> 
		</a>
	</div>
	<div class="inner-grid">
		<a href="/assets/images/{{ page.slug }}/ifttt2.png">
			<img src="/assets/images/{{ page.slug }}/ifttt2.png"/> 
		</a>
	</div>
	<div class="inner-grid">
		<a href="/assets/images/{{ page.slug }}/ifttt3.png">
			<img src="/assets/images/{{ page.slug }}/ifttt3.png"/> 
		</a>
	</div>
</div>
<div class="image-caption">
	My IFTTT configuration for opening the shade from Google Assistant.
</div>

When I say “OK Google, Open the balcony shade”, Google Assistant will send the value “100” to the “shade-open” feed. The shade controller receives this message through Adafruit IO and raises the shade until it reaches the NFC tag that signals the shade is 100% open. 

Here’s everything in action:

<div class="media-center">
{% include youtubePlayer_v.html id=page.yt_demo_full %}
</div>
<div class="image-caption">
	Testing voice command from Google Assistant.
</div>

Nice!

<br />

## Final thoughts and tips
I spent weeks working on this project, I love how it turned out and what I gained from it. Operating my balcony shade is more convenient for sure, but that’s not even the most useful outcome of this project. For years, I’ve used software to improve different aspects of my life, but now I’ve gained the experience and confidence to build custom electrical products and unlocked a new type of solutions I can improve my life with. The potential is huge and I’m excited to see what I will do with it next.
 
I learned a lot of new things in this project, not all about electronics, and I think the best way to learn is to simply get your hands dirty and dig deeper as problems arise. I figured it’s worth mentioning a few quick tips from my experience working on this project, some I learned the hard way:

While I knew I wanted to add NFC support from the beginning, I went with an __iterative approach__ - started with a simple circuit and made changes in steps. This might take a bit longer than completing the vision in one go, but it helps seeing and fixing problems early, before they become bigger or harder to pinpoint. The battery life problem is a good example for something I didn’t put too much thought into, but I was able to fix it before getting into the more complex NFC issues that came later.

I also recommend having a simple way to use the circuit in a __development environment__. After I mounted the motor and the switch box, I was still able to take the main circuit to my desk and debug problems using a simpler motor and remote. I also added a flag in the code for disabling the NFC, so I won’t have to connect an NFC module in my development environment unless I need to. This was really convenient and greatly improved my development process.

__Breadboards can be unreliable__, take that into account when testing your circuits. I spent hours on a problem with the NFC module, which turned out to be a result of jumper cables that got momentarily disconnected and caused weird state issues. If the circuit is not expected to change much, consider doing it on a perfboard.

Soldering is an important skill and having the right tools can make a big difference. My circuits looked a lot more professional and less frustrating to make after I got me a [helping hand](https://www.amazon.com/gp/product/B07LGLWGZ5/ref=ppx_yo_dt_b_search_asin_image?ie=UTF8&psc=1) - a __soldering tool__ with arms for holding the circuit and other components while soldering. It also includes a strong light and magnifying glass.

One more soldering tip - some components are just hard to solder straight (I'm looking at you headers!). I learned I can use [sticky mounting tabs](https://www.amazon.com/dp/B074R75LH1/ref=cm_sw_em_r_mt_dp_Ogp7FbP40B436?_encoding=UTF8&psc=1) to __steadily position the components while soldering them__. Really useful and made my circuits look more professional.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/tips-mounting-tabs.jpg">
		<img src="/assets/images/{{ page.slug }}/tips-mounting-tabs.jpg"/> 
	</a>
</div>
<div class="image-caption">
	Soldering headers with mounting tabs.
</div>

There was a lot of drilling in this project, and I learned a lot about types of surfaces and drills, but I think the highlight is getting to know the [Step Drill](https://www.amazon.com/dp/B07NKXLTCB/ref=cm_sw_em_r_mt_dp_R6xWFbNZJNW16) for __drilling perfect round holes__ in plastic boxes. I found out about this awesome drill bit after spending way too much time drilling holes in the switch box, for inserting the 2 up/down buttons. The Step drill can make the same holes in seconds, and perfectly round too. I think it's a must for projects like this.

<div class="media-center">
	<a href="/assets/images/{{ page.slug }}/step-drill.jpg">
		<img src="/assets/images/{{ page.slug }}/step-drill.jpg"/> 
	</a>
</div>
<div class="image-caption">
	The Step drill bit and the clean hole I made with it.
</div>

That’s all! The source code and designs are available in the Resources section below, in addition to the list of components I used and other useful resources for this and similar projects.

<br />

## Resources
The source files are available on Github, including the Fritzing diagrams, PCB schematic and Gerber files. 

[https://github.com/Udinic/smartShadeController](https://github.com/Udinic/smartShadeController)

The full parts list:
- ESP32 Dev-board [[Amazon](https://www.amazon.com/gp/product/B07Q576VWZ/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1&fpw=alm)]
- PN532 NFC module [[Amazon](https://www.amazon.com/HiLetgo-Communication-Arduino-Raspberry-Android/dp/B01I1J17LC/ref=sxts_sxwds-bia-wc-drs1_0?cv_ct_cx=pn532&dchild=1&keywords=pn532&pd_rd_i=B01I1J17LC&pd_rd_r=625c7ef6-462a-4aa4-9587-8b7ff990436d&pd_rd_w=O7fj7&pd_rd_wg=jacP8&pf_rd_p=c33e4373-edb9-47f9-a7e6-5d3d6a7a4ad0&pf_rd_r=AMD8MZN7FS1WPQTJGH86&psc=1&qid=1607317072&sr=1-1-5e875a02-02b1-4426-9916-8a5c26cd5a14)]
- Servo motor [[ServoCity](https://www.servocity.com/sg12-series-servo-gearbox-2-1-ratio-continuous-rotation-700-oz-in-31-3-rpm/)]
- Waterproof plastic box [[Amazon](https://www.amazon.com/gp/product/B07C6W5MN4/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)]
- Bracket for connecting the motor to the crank [[ServoCity](https://www.servocity.com/lightweight-linear-actuator-mounting-bracket/)] 
- Structural components for mounting [[ServoCity](https://www.servocity.com/structure/)]
- Servo extension cords [[Amazon](https://www.amazon.com/dp/B07BQMS1JV/ref=cm_sw_r_tw_dp_h5BZFbQ86XA19?_x_encoding=UTF8&psc=1)]
- Barrel jack connector [[Adafruit](https://www.adafruit.com/product/373?gclid=CjwKCAiAn7L-BRBbEiwAl9UtkJhH8XqXyK7Sm-5B5ml_mp9IU7h-2Blk8scO0Ig0RNnKCLC5b1yfmxoCOhMQAvD_BwE)]
- 2x 330Ω, 220Ω and1kΩ resistors [[Amazon](https://www.amazon.com/Elegoo-Values-Resistor-Assortment-Compliant/dp/B072BL2VX1/ref=sr_1_3?crid=3CIQCTPZLFIGO&dchild=1&keywords=resistors&qid=1607317189&s=industrial&sprefix=resis%2Cindustrial%2C154&sr=1-3)]
- Buck Converter [[Adafruit](https://www.adafruit.com/product/1385)]
- Headers [[Amazon](https://www.amazon.com/Header-Lystaii-Pin-Connector-Electronic/dp/B06ZZN8L9S/ref=sr_1_2?crid=2YEXP032IUD8O&dchild=1&keywords=headers+electronics&qid=1607317221&sprefix=headers+elec%2Caps%2C165&sr=8-2)]
- Terminal block [[Amazon](https://www.amazon.com/dp/B01MT4LC0F/ref=cm_sw_em_r_mt_dp_xEBZFbGJ0RPX3)]
- Waterproof buttons [[Amazon](https://www.amazon.com/gp/product/B079HR5Q4R/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1&fpw=alm)]
- Switch box [[Amazon](https://www.amazon.com/gp/product/B07C6S4Y7D/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1&fpw=alm)]
- Acrylic sheet [[Amazon](https://www.amazon.com/gp/product/B07MG8KTBX/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1&fpw=alm)]
- Power adapter [[Amazon](https://www.amazon.com/gp/product/B01MT5WVCG/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1&fpw=alm)]
- Perfboard [[Amazon](https://www.amazon.com/dp/B01KJMN7OY/ref=cm_sw_r_tw_dp_x_AQBZFb7THRB8W)]

A few useful places that helped me along the way:

[ServoCity](https://www.servocity.com/) - Where I bought the motor and a lot of other structural components that helped me mount the motor. Great site to look for interesting parts for different types of projects.

[Adafruit](https://www.adafruit.com/) - I got many of my components from Adafruit, they open-sourced great Arduino libraries and operate helpful forums. A must have for any hobbyist.

[SparkFun](https://www.sparkfun.com/) - I liked their [“where do I start” tutorials](https://learn.sparkfun.com/tutorials/where-do-i-start#starter-tutorials), which are a great refresher for the basics. SparkFun also makes and sells components, much like Adafruit.





