# Introduction #

This is a overview of how to program a MX-28 servo ID and/or Baud rate

## Items ##
  * <p> Ardunio, for this wiki we will use a Ardunio UNO<br>
<br>
Unknown end tag for </P><br>
<br>
<br>
<ul><li><p> MX-28 servo<br>
<br>
Unknown end tag for </P><br>
<br>
<br>
</li><li><p> Adapter cable, this is to connect the MX-28 servo to the Ardunio<br>
<br>
Unknown end tag for </P><br>
<br>
<br>
</li><li><p> 12 volt(direct current) power supply <i>(item not shown in image</i> below) <br>
<br>
Unknown end tag for </P><br>
<br>
<br>
<img src='http://slide-33.googlecode.com/files/prog items.jpg' />
<br>
<br>
<P><br>
<br>
-----------------------------------------------------------------------------------<br>
<br>
</P><br>
<br>
</li></ul>


<h1>Details</h1>

<ul><li><b>Task 1 :</b> <p> Download the <br>
<br>
<A href ="http://slide-33.googlecode.com/files/Dynamixel_Serial%20-%20V2.0.zip"><br>
<br>
 Dynamixel_serial library <br>
<br>
</A><br>
<br>
 which is found in the download section <br>
<br>
Unknown end tag for </P><br>
<br>
</li></ul>

<ul><li><b>Task 2 :</b> <p> Connect the MX-28 "VDD" pin to the Arduino "Vin" pin</p> <p> connect the MX-28 "GND" pin to the Ardunio "Gnd" pin </p> <p>connect the MX-28 "data" pin to the Arduino "TX" pin (Also know as pin 1)</p>
<img src='http://slide-33.googlecode.com/files/prog connection.jpg' /></li></ul>

<ul><li><b>Task 3 :</b> <p> program the Ardunio with the following sketch</li></ul>

<pre><code><br>
/*<br>
This is a setup sketch for only one MX-28 connected and it is used to set ID and Baudrate of the Dynamixal<br>
<br>
Connection of Dynamixel to Arduino<br>
==================================<br>
You do not need a half to full duplex circuit if you do not wish to receive ANY data FROM the Dynamixal servo, as we are only setting up the Dynamixal servo we will leave this circuit out and connect directly to Arduino to make things simple.<br>
<br>
MX-28 (Pin)            Arduino (Pin)<br>
====================================<br>
GND (1) -------------- GND (Power GND)<br>
VDD (2) -------------- VIN (Power VIN)<br>
DATA(3) -------------- TX (Pin 1)<br>
<br>
With the 3 wires connected as above and the Arduino programmed with this sketch connect a 12Vdc to the DC in of the Arduino.<br>
(CAUTION! This power supply must not be greater then +14.8Vdc as this is the supply that powers the Dynamixal).<br>
Wait about a ONE minute and if successfully the Dynamixal should start to move with its LED turning ON and OFF.<br>
<br>
Robotis e-Manual ( http://support.robotis.com )<br>
<br>
*/<br>
<br>
<br>
#include &lt;Dynamixel_Serial.h&gt;       // Library needed to control Dynamixal servo<br>
<br>
#define SERVO_ID 0x01               // ID of which we will set Dynamixel too<br>
#define SERVO_ControlPin 0x02       // Control pin of buffer chip, NOTE: this does not matter becasue we are not using a half to full contorl buffer.<br>
#define SERVO_SET_Baudrate 100000  // Baud rate speed which the Dynamixel will be set too (1Mbps)<br>
#define LED13 0x0D                  // Pin of Visual indication for runing "heart beat" using onboard LED<br>
<br>
<br>
void setup(){<br>
pinMode(LED13, OUTPUT);            // Pin setup for Visual indication of runing (heart beat) program using onboard LED<br>
digitalWrite(LED13, HIGH);<br>
<br>
delay(1000);                                                 // Give time for Dynamixel to start on power-up<br>
<br>
for (int b=1; b&lt;0xFF; b++){                                  // This "for" loop will take about 20 Sec to compelet and is used to loop though all speeds that Dynamixel can be and send reset instuction<br>
long Baudrate_BPS = 0;<br>
Baudrate_BPS  = 2000000 / (b + 1);                        // Calculate Baudrate as ber "Robotis e-manual"<br>
Dynamixel.begin(Baudrate_BPS ,SERVO_ControlPin);   // Set Ardiuno Serial speed and control pin<br>
Dynamixel.reset(0xFE);                                // Broadcast to all Dynamixel IDs(0xFE is the ID for all Dynamixel to responed) and Reset Dynamixel to factory default<br>
delay(5);<br>
<br>
}<br>
digitalWrite(LED13, LOW);<br>
<br>
delay(3000);                                                 // Give time for Dynamixel to reset<br>
<br>
// Now that the Dynamixel is reset to factory setting we will program its Baudrate and ID<br>
Dynamixel.begin(57600,SERVO_ControlPin);                 // Set Ardiuno Serial speed to factory default speed of 57600<br>
Dynamixel.setID(0xFE,SERVO_ID);                               // Broadcast to all Dynamixel IDs(0xFE) and set with new ID<br>
delay(10);                                                     // Time needed for Dynamixel to set it's new ID before next instruction can be sent<br>
Dynamixel.setStatusPaket(SERVO_ID,READ);                      // Tell Dynamixel to only return status packets when a "read" instruction is sent e.g. Dynamixel.readVoltage();<br>
Dynamixel.setBaudRate(SERVO_ID,SERVO_SET_Baudrate);           // Set Dynamixel to new serial speed<br>
delay(30);                                                    // Time needed for Dynamixel to set it's new Baudrate<br>
<br>
<br>
Dynamixel.begin(SERVO_SET_Baudrate,SERVO_ControlPin);    // We now need to set Ardiuno to the new Baudrate speed<br>
Dynamixel.ledState(SERVO_ID, ON);                            // Turn Dynamixel LED on<br>
delay(5);<br>
Dynamixel.setMode(SERVO_ID, SERVO,0x000,0xFFF);              // Turn mode to SERVO, must be WHEEL if using wheel mode<br>
delay(5);<br>
Dynamixel.setMaxTorque(SERVO_ID, 0x2FF);                     // Set Dynamixel to max torque limit<br>
}<br>
<br>
<br>
<br>
// Flash Dynamixel LED and move Dynamixel to check that all setting have been writen<br>
void loop(){<br>
digitalWrite(LED13, HIGH);                  // Turn Arduino onboard LED on<br>
Dynamixel.ledState(SERVO_ID, ON);           // Turn Dynamixel LED on<br>
delayMicroseconds(1);<br>
//  Dynamixel.wheel(SERVO_ID,LEFT,0x3FF);              // Comman for Wheel mode, Move left at max speed<br>
Dynamixel.servo(SERVO_ID,0x001,0x100);   // Comman for servo mode, Move servo to angle 1(0.088 degree) at speed 100<br>
delay(4000);<br>
<br>
digitalWrite(LED13, LOW);                  // Turn Arduino onboard LED off<br>
Dynamixel.ledState(SERVO_ID, OFF);         //Turn Dynamixel LED off<br>
delayMicroseconds(1);<br>
//  Dynamixel.wheel(SERVO_ID,RIGHT,0x3FF);          // Comman for Wheel mode, Move right at max speed<br>
Dynamixel.servo(SERVO_ID,0xFFF,0x3FF);  // Comman for servo mode, Move servo to max angle at max speed (angle<br>
delay(4000);<br>
<br>
}<br>
<br>
<br>
</code></pre>

<ul><li><b>Task 4 :</b> <p>Once the code has been programed to the Ardunio disconnect the USB cable and connect the 12 volt power supply.<br>
<br>
Unknown end tag for </P><br>
<br>
</li></ul>

<ul><li><b>Task 5 :</b> <p>Press the reset button on the Ardunio</p></li></ul>

<ul><li><b>Task 6 :</b> <p>Wait about 1 minute for the program to run on the Ardunio</p></li></ul>

<ul><li><b>Task 7 :</b> <p>If every thing was done correctly the MX-28 servo should start to move, at the same time the LED on the servo and Ardiuno pin 13 will flash<br>
<br>
Unknown end tag for </P><br>
<br>
