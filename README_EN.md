This is an automatic translation and may be incorrect in some places. See the source README and examples for authoritative information.

[![latest](https://img.shields.io/github/v/release/GyverLibs/GyverMotor2.svg?color=brightgreen)](https://github.com/GyverLibs/GyverMotor2/releases/latest/download/GyverMotor2.zip)
[![PIO](https://badges.registry.platformio.org/packages/gyverlibs/library/GyverMotor2.svg)](https://registry.platformio.org/libraries/gyverlibs/GyverMotor2)
[![Foo](https://img.shields.io/badge/Website-AlexGyver.ru-blue.svg?style=flat-square)](https://alexgyver.ru/)
[![Foo](https://img.shields.io/badge/%E2%82%BD%24%E2%82%AC%20%D0%9F%D0%BE%D0%B4%D0%B4%D0%B5%D1%80%D0%B6%D0%B0%D1%82%D1%8C-%D0%B0%D0%B2%D1%82%D0%BE%D1%80%D0%B0-orange.svg?style=flat-square)](https://alexgyver.ru/support_alex/)
[![Foo](https://img.shields.io/badge/README-ENGLISH-blueviolet.svg?style=flat-square)](https://github-com.translate.goog/GyverLibs/GyverMotor2?_x_tr_sl=ru&_x_tr_tl=en)  

[![Foo](https://img.shields.io/badge/ПОДПИСАТЬСЯ-НА%20ОБНОВЛЕНИЯ-brightgreen.svg?style=social&logo=telegram&color=blue)](https://t.me/GyverLibs)

# GyverMotor2
Library for managing collector motors through a driver, improved version of the library[GyverMotor](https://github.com/GyverLibs/GyverMotor)
- Speed and direction control
- Working with PWM of any permission
- Smooth start and speed change
- Minimum PWM threshold
- software deadtime
- Support for 7 Types of Drivers

### Compatibility
Compatible with all Arduino platforms (Arduino features are used)

## Contents
- [Use of use](#usage)
- [Versions](#versions)
- [Installation](#install)
- [Bugs and feedback](#feedback)

<a id="usage"></a>

## Use of use
### Driver type
| Driver | Description | Pines | speed > 0 | speed < 0 | stop | brake |
|----------------------|-------------------------|-------------------|-------------|-------------|-----------|-------------|
| `GM2::ONLY_PWM`| PWM only | (PWM) | (PWM) | (PWM) | (0) | (0) |
| `GM2::DIR_DIR`Only direction | (GPIO, GPIO) | (0, 1) | (1, 0) | (0, 0) | (1, 1) |
| `GM2::DIR_PWM`| One PWM without inversion | (GPIO, PWM) | (0, PWM) | (1, PWM) | (0, 0) | (1, MAX) |
| `GM2::DIR_PWM_INV`| One PWM with inversion | (GPIO, PWM) | (0, PWM) | (1, ~PWM) | (0, 0) | (1, MAX) |
| `GM2::PWM_PWM_SPEED`| Two PWM, speed mode | (PWM, PWM) | (0, PWM) | (PWM, 0) | (0, 0) | (MAX, MAX) |
| `GM2::PWM_PWM_POWER`| Two PWM torque mode | (PWM, PWM) | (~PWM, MAX) | (MAX, ~PWM) | (0, 0) | (MAX, MAX) |
| `GM2::DIR_DIR_PWM`| Two directions and PWM | (GPIO, GPIO, PWM) | (0, 1, PWM) | (1, 0, PWM) | (0, 0, 0) | (1, 1, MAX) |

Notes:

- In brackets, pins are indicated in the order of the designer, i.e. for example, for`DIR_PWM`initialization will`GyverMotor2<GM2::DIR_PWM> motor(gpio, pwm)`where gpio is a regular digital pin (digitalWrite), pwm is a PWM-enabled pin (analogWrite)
- `MAX`- maximum PWM value at the specified permit
- `PWM`- straight CHIM`(0.. MAX)`
- `~PWM`- reverse CHIM`(MAX.. 0)`

Mode selection for the H-bridge driver (2 pins, usually IN1 and IN2) - most of the driver modules for the Arduino:

- For symmetrical speed of operation in both directions, a mode with two PWM is recommended, where`PWM_PWM_SPEED`has a higher speed, and`PWM_PWM_POWER`It is more stable at low revs and has higher torque on some motors and drivers. Perfect for wheeled robots
- To work in both directions, when the symmetry of the engine is not important, you can use the mode.`DIR_PWM_INV`- he saves one PIN. Drivers who work with`DIR_PWM`(without inversion) are rare.
- For one-way operation with speed control (MOSFET or closed-pin bridge) the mode is suitable`ONLY_PWM`and uses only one pin.
- For relay control (without speed control), you can use the mode`DIR_DIR`

Drivers with separate direction control and speed IN1+IN2+EN need a mode`DIR_DIR_PWM`.

### Compilation settings
Before connecting the library

```cpp
#define GMOTOR2_NO_ACCEL    // cut out
```

### Class description
```cpp
// initialization indicating the driver type, PWM resolution (bit) and pins
GyverMotor2<driver, res = 8>(uint8_t pinA, uint8_t pinB = 0xff, uint8_t pinC = 0xff);

// set deadtime (delay in changing direction) in microseconds (default 0)
void setDeadtime(uint8_t us);

// Get deadtime in microseconds
uint8_t getDeadTime();

// set the reverse direction (silent. false)
void setReverse(bool rev);

// reverse
bool getReverse();

// set the minimum PWM (failed 0)
void setMinDuty(uint16_t duty);

// set the minimum PWM in % of the maximum (failed 0)
void setMinDutyPerc(uint8_t perc);

// get the minimum PWM
uint16_t getMinDuty();

// maximise
uint16_t getMaxDuty();

// set the PWM speed (-max. max. max.
void runSpeed(int16_t speed);

// set the speed in % of the maximum PWM (-100). 100%
void runSpeedPerc(int8_t perc);

// Get the current speed of the motor in PIM
int16_t getSpeed();

// receive direction: motor turns (1 and -1), motor stops (0)
int8_t getDir();

// stop. If the acceleration is activated, the
void stop();

// Active braking (leaving driver pins active)
void brake();

// stop and turn off the motor
void disable();

// set the acceleration in PWM per second. Real minimum - 5.0 to turn off
void setAccel(uint16_t accel);

// set the acceleration as a percentage per second. 0 to turn off
void setAccelPerc(uint8_t perc);

// get a period at least less than which you need to call a tick, ms
uint8_t getDt();

// Get the target speed (acceleration mode)
int16_t getTarget();

// smooth change to the specified speed with acceleration setAccel, call in loop. Return true when you reach speed
bool tick();
```

`res` sets the value range used by the library, but does not configure the hardware resolution of `analogWrite`. Configure it separately with the platform API. Standard Arduino AVR uses 8-bit `analogWrite`, so keep `res = 8`.

### Description of the fit
#### Launch and stop
- `runSpeed`starts the motor at the specified speed, maintains negative values for rotation in the opposite direction. meaning`0`switch off
- `stop`- stop and disconnect, equivalent to a call`runSpeed(0)`
- `setReverse`- engine reverse, when installed`true`It will turn in the opposite direction.
- `brake`Active braking through the driver (not supported everywhere)
- `disable`Stop and disable the driver

#### DeadTime
Inside.`runSpeed`When changing the direction of rotation, it turns off the driver and creates a delay for the specified number of microseconds, after which it gives the desired signal to the motor. This is necessary for homemade simple H-bridge drivers to avoid closing the bridge. Drivers in the form of chips and modules usually have built-in hardware deadtime and do not need to include it in the library.

#### MinDuty
Allows you to adjust the minimum signal to the motor. The value submitted in`runSpeed`It will scale linearly from MinDuty to maximum (if not zero). This allows you to "skip" the values at which the motor can not move without changing the logic of the speed setting. Example of values for 8 bits and`MinDuty=50`:

| `runSpeed`| Actual PWM |
|-----------|---------------- |
| 0 | 0 |
| 1 | 50 |
| 50 | 90 |
| 150 | 170 |
| 200 | 210 |
| 255 | 255 |

#### Accel
When setting non-zero acceleration (in PWM units per second) call`runSpeed`does not apply the specified speed immediately, instead, the speed will change smoothly to the specified. To work, you need to call a ticker in the main cycle of the program at least less often than`getDt`milliseconds (ranging from 30 to 200 depending on acceleration) A challenge`stop`will make a smooth stop, and`brake`and`disable`Stop the engine abruptly:

```cpp
void setup() {
    motor.setAccel(80);
    motor.runSpeed(170);
}

void loop() {
    motor.tick();   // smoothly accelerate the motor to a speed of 170
}
```

It can be useful when driving a heavy wheeled robot to avoid jerks during acceleration and braking.

## Examples
```cpp
#include <GyverMotor2.h>

// GyverMotor2<GM2::DIR_DIR> motor(5, 6);
// GyverMotor2<GM2::DIR_PWM> motor(5, 6);
GyverMotor2<GM2::DIR_PWM_INV> motor(5, 6);
// GyverMotor2<GM2::PWM_PWM_SPEED> motor(5, 6);
// GyverMotor2<GM2::PWM_PWM_POWER> motor(5, 6);
// GyverMotor2<GM2::DIR_DIR_PWM> motor(5, 6, 7);

void setup() {
    // motor.setDeadtime(5);
    // motor.setMinDuty(50);
    // motor.setReverse(true);
}

void loop() {
    int speed = map(analogRead(0), 0, 1023, -255, 255);
    motor.runSpeed(speed);
}
```

<a id="versions"></a>

## Versions
- v1.0

<a id="install"></a>
## Installation
- The library can be found under the name **GyverMotor2** and installed through the library manager in:
    - Arduino IDE
    - Arduino IDE v2
    - PlatformIO
- [Download the library](https://github.com/GyverLibs/GyverMotor2/archive/refs/heads/main.zip).zip archive for manual installation:
    - Unpack and put in *C:\Program Files (x86)\Arduino\libraries* (Windows x64)
    - Unpack and put in *C:\Program Files\Arduino\libraries* (Windows x32)
    - Unpack and put in *Documents/Arduino/libraries/ *
    - (Arduino IDE) Automatic installation from .zip: *Sketch/Connect library/Add .ZIP library...* and specify downloaded archive
- Read more detailed instructions for installing libraries[here](https://alexgyver.ru/arduino-first/#%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0_%D0%B1%D0%B8%D0%B1%D0%BB%D0%B8%D0%BE%D1%82%D0%B5%D0%BA)
### Update
- I recommend always updating the library: new versions fix errors and bugs, as well as optimize and add new features.
- Through the library manager IDE: find the library as when installing and click "Update"
- Manually: **Delete the folder with the old version** and then put the new one in its place. “Replacement” can not be done: sometimes new versions delete files that will remain when replaced and can lead to errors!

<a id="feedback"></a>

## Bugs and feedback
If you find bugs, create **Issue**, or better write to the mail immediately.[alex@alexgyver.ru](mailto:alex@alexgyver.ru)  
The library is open for revision and your **Pull Requests*!

When reporting bugs or incorrect work of the library, it is necessary to specify:
- Library version
- What is used by the IC
- SDK version (for ESP)
- Arduino IDE version
- Are embedded examples that use features and designs that cause bugs in your code working correctly?
- What code was downloaded, what work was expected from it and how it works in reality
- Ideally, attach the minimum code in which the bug is observed. Not a canvas of a thousand lines, but a minimum code.
