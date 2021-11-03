# Advanced Tutorial 2 - Communication Protocol (CAN) and RoboMaster motor features
Authors: Dennis, Amber
Modified by: Chalex

## Table of Content

[TOC]

## Communication Protocol(CAN)
### What is CAN
CAN, Controller area network, is an electronic communication bus defined by the ISO 11898 standards. Those standards define how communication happens, how wiring is configured and how messages are constructed, among other things. Collectively, this system is referred to as a CAN bus.
![](https://i.imgur.com/16oIj5m.png)
### Principle of CAN
Differential Signal
![ISO11898-2.jpg](https://i.loli.net/2020/03/30/rmacW45kE8Y13T2.jpg)
From the above graph, we can observe that, CNA is different from normal signal which using  5V and 0V to represent logic 1 and logic 0. It use differential signal, when there is a difference in voltage, it mean 0 and otherwise meaning 1.
ADV of Differential signal:
Noise tolerance
Accurate timing positioning

#### Data Frame
Stanard CAN message (11-bit identifer)
![](https://i.imgur.com/xi9v8vW.png)

Extended CAN message(29-bit identifier)
![](https://i.imgur.com/Ok3YeQc.png)
A CAN data frame can be divided into 8 parts:
1. SOF:Start of frame
2. CAN ID
3. RTR
4. Control
5. Data
6. CRC
7. ACK
8. End of frame

Normally, we will only focusing on three parts: CAN ID, Control and Data.
CAN ID: indicating where the data frame should be sent to and who should receiving it.
Control: Inform the length of data in bytes (0-8).
Data: Containing Actual data value, which need to be scaled or converted to be readable, e.g. current for motor.


### STM32 CAN config
**CAN1(master)** and
![](https://i.imgur.com/lKbY7qi.png) 
**CAN2(slave)**
![](https://i.imgur.com/twTFcth.png)

**Clock Config**
![](https://i.imgur.com/Edq20yk.png)
Just make sure the clock config are the same.
### STM32 HAL library
```c
HAL_StatusTypeDef HAL_CAN_Start(CAN_HandleTypeDef *hcan);
```
Important Parameters:
hcan -> point to can object;

Return value:
If successfully launched, return HAL_OK, else HAL_ERROR
```c
HAL_StatusTypeDef HAL_CAN_ConfigFilter(CAN_HandleTypeDef *hcan, CAN_FilterTypeDef *sFilterConfig);
```
Important Parameters:
hcan -> point to can object;
sFilterConfig -> point to filter object;

Return value:
If successfully launched, return HAL_OK, else HAL_ERROR
```c
HAL_StatusTypeDef HAL_CAN_AddTxMessage(CAN_HandleTypeDef *hcan, CAN_TxHeaderTypeDef *pHeader, uint8_t aData[], uint32_t *pTxMailbox);
```
Important Parameters:
hcan -> point to can object;
pHeader -> the can id that the message should be sent to.
aData -> data to be sent

Return value:
If message successfully added, return HAL_OK, else HAL_ERROR.

These are some common HAL function that will be used in CAN communication.
A library using these HAL functions are written for you to use to control motors.
Example code will be shown below.
### Example code
You will use can_trigger_motor to control your motor. Please read the code below;
```c
void can_trigger_motor(int16_t speed){
	/* motor rotate clockwise if speed is positive, vice versa.
	 * CAN ID have been defined for you 
	 * as FIRST_GROUP_ID which is 0x200
	 * Motor Max Speed is 16384.
	 * */
	if(speed>16384){
		speed = 16384;
	}else if (speed<-16384){
		speed = -16384;
	}
can_transmit(&hcan1,FIRST_GROUP_ID,speed,speed,speed,speed);
}
```
In the main.c, you need to start the CAN object and disable the can-filter before entering the while loop. So that your can object can help you deliver your data.
```c
can_filter_disable(&hcan1);
HAL_CAN_Start(&hcan1);
```
After that, you can control the motor by invoking the can_trigger_motor function in the while loop

```c
while(1){
    can_trigger_motor(100);
}
```
This will keep your motor keep rotating clockwise.



## RM motor features

### Introduction
RoboMaster Motor(RM motor), is the black cylindrical box as shown in the photo below:
![](https://stormsend1.djicdn.com/tpc/uploads/photos/1928/large_fcfcea55-88e3-4521-a81e-fa7ea136ac0d.jpg)
and we need a motor driver to convert the signal to what the motor "understand" which is called a RM-ESC:
![](https://product1.djicdn.com/uploads/sku/cover/08aa5010-64ec-447d-a568-1d2eb597f345@medium.png)

### Wiring and power supply
![](https://i.imgur.com/HSK66yp.png)
- Both [1] and [2] will attached to the RM motor.
- Connect the [5] to a 24V cells
- [7] is CAN port [8] is PWM port
    - should only choose **1 port** to use

### Potential Damage
1. Collision
    * Dropping
    * Somehow your robot has the RM motor or ESC exposed to external objects and your robot crashed into them
2. Overheating
    * Cause
        1. Overdriving (Forcing the motor to provide excessive amount of torque)
        2. Short circuit
            * 24V power wire is shorted and so the motor is provided with too much current
            * Heating effect (Hopefully you have learnt high school physics)
    * Results
        1. Non Permanant Damage
            * Just some warmth
        2. Permanant Damage
            * Electrical component gets damaged
            * More serious cases: **Sparks** and **Smoke**
##### Note
> Please check the temperature of both ESC and RM motor if something weird happened. E.g. Your motor didn't drive when it should, Irrigating smell

### Prevention
1. Make sure that the ESC is tightly tied to the robot internally
2. Ensure there is nothing that blocks the rotation motion of the RM motor (E.g. Loose screws)
    * Turn off the 24V power supply
    * Remove the obstacle if you can or else find a mech member to deal with it
    
2. Overheating
    - Irritating Smell (Smells like celery)
    - Cool down in whatever way you can (Preferably use compressed air in the lab to cool it down)

**Always turn off 24V power supply first when accidents happen**

### Configuration of RM ID
Steps:
1. Find a hard rod with a small end (We use tweezers most of the time)
2. Poke the SET Button[4] in the upper image to set the CAN ID
    - Number of times to press the SET Button = CAN ID + 1
    - The first press is to tell the RM ESC to enter CAN ID configuration mode
3. Watch the LED blink and make sure the CAN ID is correct
    - ID = LED blink time - 1

### Callibarion of the RM motor

#### Background
RM motors can only be controlled when the ESC knows the condition of the motor.
(E.g. Current and inductance)
Even though the RM motor is a commercialized product, 
it is extremely difficult for every single motors to be identical.
Thus, we need to let the ESC to know the actual condition of the RM motor through callibration.

#### Procedures
1. Again, find a hard rod with a small end
2. Press the SET Button for a long time and it will enter the callibration mode
3. Wait untill the LED blinks green quickly.
4. The motor will start spinning but no worries, just wait untill it stops.
5. Please ensure that there is no obstacle bothering the RM motor spinning motion.

### What does the LED signal mean?
TLDR: Any LED signal except for a green blinking LED is not a good sign.
![](https://i.imgur.com/5lHllL5.png)
![](https://i.imgur.com/OcNbReB.png)
![](https://i.imgur.com/saNpZGB.png)

### Electrical Structure of a RM control system
![](https://i.imgur.com/ROzjtKy.png)

### Common Errors
> Arranged in order of occurence

#### *24V power*


| Symptom                | Meaning                           | Solution                                                                                                           |
| ---------------------- |:--------------------------------- |:------------------------------------------------------------------------------------------------------------------ |
| No LED Light at all    | No input power source             | 1. Check if the wire is plugged in or not <br>  2. Check if the fuse is burnt <br> 3. Ask hardware people for help |
| Solid red LED          | No CAN signal from the RM Motor   | Check the 7-pin wire connected to the RM Motor                                                                    |
| Red LED blinking twice | The 3-phase cable is not connected. | Check the connection between RM-Motor and ESC (3-phase cable)                                                              |

#### *Signal*
> Do these when you are sure it's not the power's problem

*CAN:*


| Possible Problem | Diagnosis                                                                                                                                                                                                                                                                                           | Solution                                        |
|:---------------- |:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:----------------------------------------------- |
| Wrong CAN ID     | Check Check Check                                                                                                                                                                                                                                                                                   | Reinitialize the correct CAN ID                 |
| Wrong CAN Bus    | See if any motors are not moving the way you want                                                                                                                                                                                                                                                   | Send message to the right bus instead           |
| Wrong CAN Port   | Again, Check Check Check                                                                                                                                                                                                                                                                            | I mean, just don't plug it into the wrong hole |
| Broken CAN wire  | 1. Find a hardware member <br> 2. Check with a [DMM in beep mode](https://cdn.sparkfun.com/assets/learn_tutorials/1/01_Multimeter_Tutorial-09.jpg?__hstc=250566617.4b44870ec4a577029c49e44b73bd3bee.1627603200060.1627603200061.1627603200062.1&__hssc=250566617.1.1627603200063&__hsfp=3390389424) | Just get a new wire                             |

*PWM:*


| Possible Problem                                     | Diagnosis                                                                                                                                                                                                                                                                                           | Solution                            |
|:---------------------------------------------------- |:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |:----------------------------------- |
| Incorrect frequency/ on time initialized in the code | 1. Check with Osciloscope <br> 2. Check your code                                                                                                                                                                                                                                                   | Change it back lol                  |
| False soldering for the pwm pins on the mainboard    | Find a hardware member for help                                                                                                                                                                                                                                                                     | Hardware member should deal with it |
| Broken PWM Wire                                      | 1. Find a hardware member <br> 2. Check with a [DMM in beep mode](https://cdn.sparkfun.com/assets/learn_tutorials/1/01_Multimeter_Tutorial-09.jpg?__hstc=250566617.4b44870ec4a577029c49e44b73bd3bee.1627603200060.1627603200061.1627603200062.1&__hssc=250566617.1.1627603200063&__hsfp=3390389424) | Just get a new wire                 |
| Broken ESC (Basically never happened)                | When you are a **100%** sure you did everything correctly and the motor is not working properly <br>Swap with an ESC that works and see if it's actually faulty                                                                                                                                     | Just get a new ESC                  |




#### CAN vs PWM
Both PWM and CAN can be used to contorl the motor. 
##### PWM:
Advantange:

- Easier to use and implement

Disadvantage:

- PWM can only output signal and can't get the feedback data back. As in real-life condition, the motor cannot be exactly identical (As said from above). If we just output a signal without getting feedback, the speed of motor may not be the same. Therefore, the wheelbase may not move as you want it to be.

##### CAN:
Advantange:

- Can take encoder feedback so we can perform better control for the motors. (Details: [PID_control_method]())

Disadvantage:

- A bit annoying to implement(requires more work)
- Hard to debug when there are problems
    - Could be because of:
        - CAN IC
        - CAN wire
        - Hardware soldering
        - CAN port
        - Code
        - or EVERYTHING

:::    danger
Please tell me (Amber) that you want to use which control method and say which group you are from and how many wheels you plan to use. So that I can finish the motor configuration for you.
:::

# How many motors for the wheelbase?

- You can choose to use whatever kind of wheelbase(straight wheels or omni-wheels) **Completely optional!!!**
- Straight wheels requires 2 or 4 motors, Omni-wheels requires 3 or more motors
- Omni-wheel robots can walk in **all direction without rotating** which may make your game flow smoother by a lot.(Depends on the game flow, omni-wheel may not be the best solution for all situations)
- For more detail. Please refer to: [Omni-wheel: theory and control]()
<br>
<br>
<br>
Reference: https://rm-static.djicdn.com/tem/17348/RoboMaster%20C620%20Brushless%20DC%20Motor%20Speed%20Controller%20V1.01.pdf
