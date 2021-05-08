# Cycle Statistic Tool

This code is for my 2021 senior project, the Cycle Statistic Tool (CST). The CST is composed of two microcontrollers that collect and relay information about a cycling trip via Bluetooth Low Energy (BLE). This information is displayed on an e-ink display that is mounted on the handlebars, but can also be displayed and recorded by a separate mobile app. My [senior project partner](https://github.com/Jacob-Hoff-man/) wrote a [mobile app](https://github.com/Jacob-Hoff-man/Companion) in Xamarin with a database in SQLite.


Supporting documents for the Cycle Statistic Tool can be found in [this](https://github.com/CodyKlingler/CycleStatisticToolBinder) repository.


<video controls>
  <source src="../master/assets/riding.mp4" type="video/mp4">
</video>





The layout of the Cycle Statistic Tool (wind sensing has not yet been implemented)
<p align="center">
  <img src="../master/assets/schematic.jpg" width=66% height=66%>
</p>


The housing for the primary microcontroller.
<p align="center">
  <img src="../master/assets/handlebars.jpg" width=66% height=66%>
</p>




The code needs a bit of polishing and commenting because it was rushed to meet a deadline, but it works and is maintainable. My next step in the project will be to clean this up as you can see in the TODO section.



Metrics that are currently either displayed, transmitted, or otherwise calculated include:
1. Speed
2. Average Speed
3. Distance
4. Acceleration
5. Incline Angle
6. Longitude
7. Latitude
8. Elevation
9. Cadence (pedals per minute)
10. Angular Velocity of Pedal
11. Power
12. Calories Burned
13. Torque
14. Gear Ratio
15. Average Efficiency (distance per unit energy)
16. Instantaneous Efficiency (speed per unit power)



Metrics that are planned but not yet implemented include:
1. Wind Direction
2. Wind Speed
3. Compass 




## Issues

1. The gear ratio metric can be inaccurate because the pedals do not spin while coasting. This could be fixed by only calculating gear ratio when a threshold for power is met. Measurements should be recorded at various gear ratio's and inclines to find a proper threshold value.



2. The incline angle, which is calculated from acceleration vectors, is inaccurate when accelerating. It may be possible to compensate by subtracting acceleration found by the IR sensor from the IMU's acceleration vectors. If this is still inaccurate, a 9 axis IMU may be necessary.



3. The GPS infrequently causes the primary microcontroller to crash when garbage data is read from it. This issue is difficult to reproduce. Using another IO method than i2c or using a different module altogether may fix the issue.




## TODO

1. Clean up codebase. (see comments in main, updateEvent.h, and Stat.h)
2. Fix gear ratio and incline angle inaccuracies.
3. Add charging ports to housing
4. Add a voltage regulator and implement lithium batteries to secondary microcontroller.
5. Design housing for secondary microcontroller, potentially compact the current design.
6. Modify display code so that tables are indexed by row and column number rather than variables.





## Hardware

The block diagram below details each component and its connections.


![](../master/assets/block_diagram.jpg)



MICROCONTROLLERS

![](../master/assets/primary_microcontroller.jpg)


![](../master/assets/secondary_microcontroller.jpg)

>The primary and secondary microcontrollers are both Arduino Nano 33 IoT's. These boards have a built in IMU for measuring 3-axis acceleration and 3-axis angular velocity, a BLE module for transmitting data wirelessly with low energy usage, and a Wi-Fi module. These features combined with their small form factor make them an ideal candidate for this project. The microcontrollers are programmed in C++ using the Arduino IDE.




DISPLAY AND DISPLAY BREAKOUT

![](../master/assets/display.jpg)

>The display and display breakout are connected to the primary microcontroller inside of the encasement on the handlebars. the display breakout acts as a data buffer and controller for the display, which connects to it directly. The e-ink display that was selected has a very low current draw and is easy to see in daylight, but can only be updated once every 3 minutes. The display shows rolling averages of values recorded since the display update in a table format. This is not a functionality included in the libraries for the display.





BATTERIES
>The primary microcontroller is powered by a 3.7V lithium-ion battery. The logic level of both microcontrollers is 3.3V and can have an input voltage of 21V. The secondary microcontroller was also intended to be powered with a 3.7V lithium ion battery but would need a 5V voltage regulator because our load cell amplifier uses 5V logic. Our intended temporary solution to this issue was to use a USB battery bank, but all of the ones that were tested shut off due to their circuitry which disables low current draw. For the final presentation, three 1.5V AA batteries were used in series for a total output of 4.5V. This was done because of the lack of a voltage regulator and the issues with our USB battery banks. The secondary microcontroller needed to have its VUSB jumpers located on the back of the board shorted, and also to be powered via its USB port rather than VIN and GND pins in order to enable 5V output.



IR SENSOR

![](../master/assets/ir_sensor.jpg)

>The IR sensor mounted on the frame of the bicycle faces the path of an aluminum disk which is mounted on the axle. This allows us to effectively count the rotations of the tire, which is then used to calculate speed, distance, and acceleration. The signal produced by the IR sensor is a digital square wave where LOW corresponds to the aluminum disk reflecting the infrared light emitted by the sensor. This signal must be debounced to increase accuracy.




IMU
>An IMU is used on both the frame of the bicycle and the crankarm. The microcontrollers selected have an IMU installed on the board, internally connected via i2c, eliminating the need to purchase additional modules. On the frame, the internal IMU measures a ratio of accelerations to calculate the angle of incline of the bicycle. On the crankarm, the IMU is used to measure angular velocity, which is then used to calculate cadence (pedals per minute), power, and calories burned.




STRAIN GAUGES AND LOADCELL AMPLIFIER

![](../master/assets/strain_gauges.jpg)

>The strain gauges are variable resistors which change value based on the deflection of the crank arm. This deflection is caused by the torque applied to the pedal by the cyclist. The strain gauges are extremely sensitive and volatile so they must be handled with care. Proper application procedures are vital to mounting these sensors on the crank arm. The strain gauges are assembled in a Wheatstone bridge where both the input and output voltages are measured by a load cell amplifier (HX711). This load cell amplifier outputs a digital signal corresponding to the strain on the crank arm. The strain values measured must be tared and a calibration factor applied in order to get measurements of torque or mass. The calibration factor can be found by placing known masses on the crank arm. Change in the strain value should have a linear relationship with mass. Measurements from the strain gauges are used in calculations for power, calories burned, and torque.

The strain gauge model that was used is the Micro-Measurements EA-13-250BF-350.







GPS
>The GPS is connected to the primary microcontroller through i2c and is mounted in the casement on the handlebars. The GPS is used to measure latitude, longitude, and altitude but could also be used to measure speed and cardinal direction. The i2c connection to the primary microcontroller is volatile, and occasionally (maybe once in 24 hours of usage) garbage data may be received. Both of these issues can cause the device to crash. The device can also take a while to find a fix which takes a connection to at least 4 satellites. This is due to the weak antenna of the GPS module. All of these problems could potentially be rectified by using a different module or using smart phone to relay GPS information.





## Instructions


You will need the following devices from the Arduino Device Manager:

	Arduino Nano IoT 33 Core



You will need the following libraries from the Arduino Library Manager:

	ArduinoBLE

	Adafruit_GPS

	Adafruit_EPD

	Adafruit_GFX

	SparkFunLSM6DS3


>NOTE: Configuration inside the code that you may need to do includes: calibrating your strain gauges to find the strain factor for the crank arm that you are using, defining the secondary microcontroller's UUID in `CSTDefinitions.h`, changing pin definitions in both `CSTDefinitions.h` and `final_secondary_microcontroller_code.ino`.



1. Delete `main.cpp` from both the final and secondary microcontroller folders. Those files were placed there only for readability on GitHub. The files named `final_primary_microcontroller_code.ino` and `secondary_primary_microcontroller_code.ino` are equivalent to their respective `main.cpp`.
2. Place both the `/final_primary_microcontroller_code/` folder and `/secondary_primary_microcontroller_code/` folder in `/Documents/Arduino/`
3. Open the `.ino` corresponding to the microcontroller that you want to program
4. Compile and upload code.





Steps to Attach to Bike


Assuming that the crankarm comes preassembled with microcontroller, battery, and strain gauges already mounted and that the user has removed any existing crank arm from the left side, the steps are as follows:



1. Place CST crank arm on bicycle.
2. Fasten crank arm bolt.
3. Attach aluminum disc to rear tire axle using adhesive.
4. Secure casement on handlebars with zip tie.
5. Secure IR Sensor to frame with solid core wire.
6. Secure loose wires with zip ties or tape.



Setup Steps Before Use
1. Power on primary microcontroller via power switch
2. Turn left crank arm so that it is parallel with the ground.
3. Power on the secondary microcontroller.
4. If desired, connect the mobile app via Bluetooth.


Video of the steps taken to start the Cycle Statistic Tool:
![](../master/assets/starting.mp4)

