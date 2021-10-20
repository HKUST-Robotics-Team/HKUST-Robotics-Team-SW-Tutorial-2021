# Tutorial 3 - Advanced IO [Homework]

#### Main Author: Caner Demirer

#### Edited by: Amber (yytangad@connect.ust.hk)

In this homework you will design and implement a "ninja cut". The setup consists of a development board (which has 2 buttons and 3 LEDs ), a servo, and a tiny 3D-printed "melee weapon" attached to the servo. You need to use PWM to control the servo to achieve the desired functionality described below. 

The main goal is to have a controller that works and feels like those in Action RPG video games. Since having movements as complex as those in the video games is very difficult to achieve with real actuators, this task is very simplified. 

### Details

There are **three(+) types of attack**: 
 - Light Attack
 - Heavy Attack (Chargeable)
 - Special Attack (Designed by you)
 - Some combo attacks defined further below. 
     - After the combo counter hits 3, Special Attack is activated for single use.

The target is imagined to be directly in front. All movement is rotation on a single axis. There are only two attack directions: attack from the left and attack from the right. The attack direction will alternate. 

When there is nothing to do, the weapon stays at Resting Angle of either side. All attacks start at one side's Resting Angle and end at the other side's Resting Angle.


#### Button Arrangement
[Light Attack] [Heavy Attack] [Special Attack]

An attack request is made when the corresponding button is pressed. Notice that there are only 2 buttons on the board, u guys can decide how to use 2 button to do 3 buttons task.

#### Light Attack
It means a sweeping. A Light Attack needs to move slower then the Heavy Attack.

> Sweeping: It means starts at 0deg, moves towards the target and ends at 180deg (of the opposite side).

#### Heavy Attack
It consists of 2 stages. First, it start at 0 deg and pausing at 40deg (charging for 2 seconds) until the player releases the Heavy Attack button. (If it is at 180 deg then it will pausing at 140 deg)

While performing the heavy attack, you should also use LEDs to indicate the charging status as shown:



| State | LED Status | Duration |
| ----- |:---------- | -------- |
| 1     | [0][0][0][0]  | 0.0~0.499s     |
| 2     | [1][0][0][0]  | 0.5~0.999s     |
| 3     | [1][1][0][0]  | 1~1.499s     |
| 4     | [1][1][1][0]  | 1.5~1.999s     |
| 5     | [1][1][1][1]  | >=2.0s     |

> Note: **[0]** = LED off, **[1]**: LED on.

**Non fully-charged Heavy Attack**
If the player had already released the button while the Heavy Attack was not charged enough, then there won't be any charging nor pausing. It then sweep for only one time. *Notice that the Heavy attack need to move faster then the Light Attack.*

**A fully-charged Heavy Attack** is a Heavy Attack that has been charged for at least 2 seconds. After the button being released. It then sweep for 3 times. *Notice that the Heavy attack need to move faster then the Light Attack.*

## Marking scheme
The task is split into several checkpoints. To get the points of a checkpoint, you need to satisfy the requirements of both that checkpoint and all checkpoints above it. 

*  Having the weapon oscillate from side to side based on button clicks [3]
*  Having 2 stages at Heavy Attack mode. [3]
*  Having correct LED blinking action when holding the button.[4] 
*  Having correct Non fully-charged Heavy Attack [2]
*  Having correct Fully-charged Heavy Attack [3]



## Bonus:
You can design your own Special Attack:
**A Special Attack** is a custom attack animation that must be designed by yourself, the only requirements are:
- It looks cool. At least not similar to Light and Heavy attack(3 pt)
- Use some LEDs. (2 pt)



### Story:
There are 3 villains with differnt HP. You now need to beat them(make ther HP become 0 or less). At first user will enter a selecting mode which will allow them to select the villain to attack.

Each of the attack will output different anount of damage with different effects:
###
Light Attack: -10 HP to single villains
###
Heavy Attck(not fully charged): -10 HP to the selected villains and -5 HP to the villains next to the selected one.
###
Heavy Attack(fully charged): -15 HP to the selected villains and -10 HP to the villains next to the selected one.
###
Special Attack: -20 HP to all villain.

There are 2 modes:
- Selecting mode: which let user select which to attack first.
- Attack mode: which user can choose the attack.

Usage of button in Selecting mode:
- Button 1: Choose villain to attack.
- Button 2: Exit the "Selecting mode" and automatically swap to "Attack Mode".

Usage of button in Attack mode:
- Button 1: Light Attack.
- Button 2: Heavy Attack.

There are several rules have to obey:
- Can use Light Attack at any time.
- 2 Light Attack can unlock a chance to use Heavy Attack. When Heavy attack is used, the counter for it will minus 2.
- When any one villain down or 2 fully-chraged Heavy attack is used, it will auto-triggered Special Attack. The counter for it will minus 2.

![](https://i.imgur.com/gxacDJd.jpg)

Instructions[Point(s)]:
- Display 3 HP (50,150,70) in tft. [3]
- Indicate the target that will be attack [2]
- Subtract the HP correctly with all 3 types of attack [7]
- Correct sequence of villain selection [4 + 5]
  - At the beginning, the player can select a villain [4]
  - After the first round, the villain on the right hand side will be selected automatically(no need button pressed)(wrap around if needed) [5]
- Perform attack rule checking [10]
  - e.g. after two light attacks, a heavy attack can be performed
- Add a Winning message when all villain has 0 or less hp. [1]


