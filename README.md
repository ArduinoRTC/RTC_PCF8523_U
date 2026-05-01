# NXP PCF8523

This library is part of a suggestion to create a unified API for handling RTCs, and is a library (device driver) for the
[NXP PCF8523][PCF8523].

## Operation Verification


| CPU | Model | Status |
|---|---|---|
| AVR | Arduino Mega | ○ |
| SAMD | Arduino MKR WiFi1010 | ○ |
| SAM | Arduino Due | ○ |
| ESP32 | SwitchScienceESP developer32 | ○ |


## External Link


- NXP PCF8523 DataSheet - [https://www.nxp.com/docs/en/data-sheet/PCF8523.pdf](https://www.nxp.com/docs/en/data-sheet/PCF8523.pdf)
- Adafruit PCF8523 Real Time Clock Assembled Breakout Board - [https://www.adafruit.com/product/3295](https://www.adafruit.com/product/3295)

## Precautions

The ``RTC_PCF8523_U.h`` contains flags that enable features used for debugging and performance testing, as shown below.
Enable or disable these as needed.
```
#define DEBUG
```

## Sample Program
This sample program is intended to verify whether RTC registers are properly configured when each function of this driver are executed.

Please compile and install the program after enabling the ``DEBUG`` definition of ``RTC_PCF8523_U.h``.

By disabling the following line, detailed messages and verification of the register contents are performed.
```
#undef DEBUG 
```

If you enable the following line, the contents of all registers will be dumped after each test has written to the registers.
```
#define DUMP_REGISTER  // Enable this option (and DEBUG) if you want to view a dump of the register values after writing to the registers.
```
# API
To understand the descriptions of the following each function, you need to know the meaning of bit in the RTC each registers, so please read this while referring to the [datasheet][datasheet].．

## Initialization-Related
### Object
```
RTC_PCF8523_U(TwoWire * theWire, int32_t rtcID=-1)
```
Create an object by specifying the I2C I/F and ID used by the RTC.

### Initialization
```
bool  begin(bool init=true, uint8_t addr=RTC_PCF8523_DEFAULT_ADRS)
```
Initializing the RTC.If ``false`` is passed as an argument, only minimal interface initialization is performed without setting the time, which is convenient when restarting the Arduino after the RTC has already been configured.
Specify the second argument if you are not using the RTC's default I2C address.

| Return Value | Meaning |
|---|---|
|true|Initialization successful|
|false|Initialization failed|


## Retrieving RTC Information
A member function that retrieves information about the RTC chip type and its features.
```
void  getRtcInfo(rtc_u_info_t *info)
```


## Time Information-Related
### Time Settings
```
bool  setTime(date_t* time)
```
Sets the time specified in the argument to the RTC.
| Return Value | Meaning |
|---|---|
|true|Set successfully|
|false|Set failed|

### Getting Time
```
bool  getTime(date_t* time)
```
Stores the time information obtained from the RTC in a structure passed as an argument.
| Return Value | Meaning |
|---|---|
|true|Retrieval successful|
|false|Retrieval failed|


## Power Supply

### Configuring the Power Supply Voltage Monitoring Function
```
int   setLowPower(uint8_t mode)
```

The argument ``mode`` specifies the value to be written to the Control 3 register (register 0x02).
The meaning of each bit is as shown in the table below; however, please refer to the datasheet for details on PM.

| bit | Name | Description |
|---|---|---|
| 5–7 | PM | Selection of the low power supply voltage monitoring function |
| 2–4 |  | Unused |
| 1 | BSIE | Whether to use the flag indicating a power supply switch event |
| 0 | BLIE | Whether to use the flag indicating low battery voltage |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS |Configuration successful|
|RTC_U_FAILURE |Configuration failed|
|RTC_U_ILLEGAL_PARAM |Setting of unsupported parameters, etc.|

### Supply Voltage
```
int   checkLowPower(void)
```

The return value when information acquisition is successful is as follows.
| bit | Name | Content | Meaning |
|---|---|---|---|
| 7 | OS | Data of bit 7 of the seconds register (0x03) | 0 indicates no error |
| 4 to 6 | None | Unused | No meaning |
| 0 to 3 | None | Data of bits 0 to 3 of Control Register 3 (0x02) | Refer to datasheet |

Roughly speaking, if the 7th bit is set to 1, some kind of abnormality has occurred, and the specific details are output in bits 0 through 3.
If retrieving the information (register access) fails, ``RTC_U_FAILURE`` is returned.


### Clearing the Flag Related to a Drop in Supply Voltage
```
int   clearPowerFlag(void)
```
Clear information regarding power supply abnormalities (write zeros).
- Set bits 2 and 3 of the Control 3 register (0x02) to 0
- Set bit 7 of the seconds register (0x03) to 0

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


### Configuring the Power Monitoring Function
```
setLowPower(uint8_t mode)
```

Overwrite the corresponding bits in control3 with the values of partially bits in mode. The bits to be overwritten are as follows.

| Control 3 Register Bits | Name | Writable |
|---|---|---|
| 5–7 | PM | Yes |
| 4 | - | No |
| 3 | BSF | No |
| 2 | BLF | No |
| 1 | BSIE | Yes |
| 0 | BLIE | Yes |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

## Interrupt Relationships
The flags indicating that an interrupt has occurred are concentrated in bits 3 through 7 of the Control 2 register (0x01).
### Retrieving Information about an Interrupt that has Occurred
```
int   checkInterupt(void)
```

Returns the data from Control Register 2 (0x01) after shifting it 3 bits to the right, leaving only the interrupt-related data.
Refer to the datasheet for the information indicated by each bit.

| Return Value | Meaning |
|---|---|
| 0 or greater | Data read from Control2 |
| RTC_U_FAILURE | Failure to read the Control2 register |
| RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


### Clearing Interrupt Information
```
int   clearInterupt(uint16_t type)
```
Clears the bits specified by type (bits set to 1) in the data from bits 3 to 7 of the Control 2 register (0x01) to 0.
The correspondence between the bits in type and the bits in the Control 2 register is such that the least significant bit of type corresponds to the 3rd bit of the Control 2 register.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

## Clock Setting
In addition to the function for adjusting the clock's rate of advance, the PCF8523 also features a function to completely stop the clock when it is battery-powered. It also includes an RTC software reset function.

### Stop/Start the Clock
```
int   controlClockHalt(uint8_t mode)
```

| Mode value | Action |
|---|---|
| 0 | Stop the clock |
| 1 | Advance the clock |


| Return value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |

### Determining Whether the Clock has Stopped
```
int   clockHaltStatus(void)
```

| Return Value | Meaning |
|---|---|
| 0 | The clock is running |
| 1 | The clock has stopped |
| RTC_U_FAILURE | Failed to retrieve information |

### Software Reset
```
int   controlClock(void)
```
| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |

### Adjusting the Clock's Speed
```
int   setOscillator(uint8_t mode)
```
You need to call the function by directly assigning the value to be set in the offset register to the ``mode`` argument. For details on the offset register, please refer to the datasheet.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |

### Refer to the Clock Rate Adjustment Settings.
```
int   getOscillator(void)
```
Returns the data in the offset register (0x0E) as-is.

| Return Value | Meaning |
|---|---|
| 0 or greater | Contents of the offset register |
| RTC_U_FAILURE | Read failure |

## Clock Signal Output
Since this RTC can output only one signal at a time, the first argument of each of the following functions is limited to 0.



### Change Clock Signal Output Settings
```
int   setClockOutMode(uint8_t num, uint8_t freq)
```
The output frequency is changed by specifying the output frequency with the second argument ``freq``.
Since the value of ``freq`` is overwritten by bits 3 through 5 (COF) of the timer and CLKOUT control register (0x0F), ``freq`` is also a 3-bit binary value.

Please refer to the datasheet to determine which frequency corresponds to the value of ``freq``.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


### Signal Output ON/OFF
```
int   controlClockOut(uint8_t num, uint8_t mode)
```

|mode value | Meaning |
|---|---|
| 0 | Stop signal output |
| 1 | Resume signal output (resume after stopping) |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |



### Clock Signal Output Settings
```
int   setClockOut(uint8_t num, uint8_t freq, int8_t pin=-1)
```
Since this RTC does not have a function to adjust the voltage applied to specific RTC pins in order to control the clock signal output, this API executes the above ``setClockOutMode(num,mode)`` and ``controlClockOut(num,1)`` functions together.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

## Alarm-related
Since the alarm of PCF8523 has only one type, the first argument ``num`` of each of the following functions is limited to 0.

### Alarm Settings
```
int   setAlarm(uint8_t num, alarm_mode_t * mode, date_t* timing)
```

If the structure member useInterruptPin of the mode is 1, the system is configured to generate an interrupt signal on the INT pin.

The PCF8523 alarm allows you to specify the minute, hour, day, and day of the week. For any items you do not intend to use, assign 0xFF to the corresponding timing member variable.

For example, when specifying only the hour and minute, you must perform this function as ``timing.mday=0xFF`` and ``timing.wday=0xFF``.


| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |



### Alarm Mode Settings
```
int   setAlarmMode(uint8_t num, alarm_mode_t * mode)
```

The meaning and behavior of the arguments are the same as those of ``setAlarm()``.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


### Alarm Control
```
int   controlAlarm(uint8_t num, uint8_t action)
```

Use ``action`` to specify whether the alarm is ON (1) or OFF (0).

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


## Timer
There are four timers, but not all of them are available simultaneously. Internally, two circuits (A and B) handle  one (B is independent) and three (A handles itself and the other two timers) timers, which results in limitations. Please refer to the datasheet for details.

In addition, in the following functions, the relationship between the first argument ``num` and the four types of timers is shown in the table below.

| Value of ``num`` | Timer |
|---|---|
| 0 | Timer B |
| 1 | Timer A |
| 2 | watchdog timer |
| 3 | Fixed-period (Second) timer |

Refer to the datasheet for the detailed operation of each timer.

### Parameters for Each Timer

Timer parameters are set using two types of arguments in the timer functions: ``rtc_timer_mode_t mode`` and ``uint16_t multi``. The following summarizes how these arguments are handled by each timer (which bits of which registers they are assigned to). Please refer to the datasheet for the meanings of the bits in each register.

| Members of rtc_timer_mode_t | Timer B (num=0) | Timer A (num=1) | Watchdog timer (num=2) | Fixed-period (Second) timer (num=3) |
|---|---|---|---|---|
| uint8_t  repeat |TBM for the timer and CLKOUT control registers  |TAM for the timer and CLKOUT control registers  | Same as left | Same as left |
| uint8_t  useInterruptPin | CTBIE in the Control 2 register | CTAIE in the Control 2 register | WTAIE in the Control 2 register | (Unused) |
| uint8_t  interval | TBQ in the Timer B frequency control register | TAQ in the Timer A frequency control register | Same as left | (Unused) |
| uint8_t  pulse  | Timer B frequency control register's TBW | (Unused)    | (Unused) | (Unused) |

|Timer Type| multi |
|---|---|
|Timer B|Assign the lower 8 bits to the Timer B register|
|Timer A|Assign the lower 8 bits to the Timer A register|
|Watchdog Timer|Same as above|
|Fixed-period (Second) Timer| (Not used) |

### Timer Settings
```
int setTimer(uint8_t num, rtc_timer_mode_t * mode, uint16_t multi)
```
According to the table above, the value of multi is processed for each timer type, and ``setTimerMode(mode)`` and ``controlTimer(num, 1)`` are called internally in sequence.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


### Changing Timer Parameters
```
int setTimerMode(uint8_t num, rtc_timer_mode_t * mode)
```

Here, following the parameter table above, the values of each structure member of the ``mode`` argument's are assigned to registers.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |

### Timer On/Off
```
int controlTimer(uint8_t num, uint8_t action)
```

|action value|Meaning|
|---|---|
| 0 | Stop the timer |
| 1 | Resume the timer (only if it has been stopped) |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |



[datasheet]:https://www.nxp.com/docs/en/data-sheet/PCF8523.pdf
[AdafruitPCF8523]:https://www.adafruit.com/product/3295
[PCF8523]:https://www.nxp.com/products/peripherals-and-logic/signal-chain/real-time-clocks/rtcs-with-ic-bus/100-na-real-time-clock-calendar-with-battery-backup:PCF8523


[github]:https://github.com/sparkfun/SparkFun_DS3231_RTC_Arduino_Library
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[AdafruitUSD]:https://github.com/adafruit/Adafruit_Sensor
[shield]:https://www.seeedstudio.com/Base-Shield-V2-p-1378.html
[M0Pro]:https://store.arduino.cc/usa/arduino-m0-pro
[Due]:https://store.arduino.cc/usa/arduino-due
[Uno]:https://store.arduino.cc/usa/arduino-uno-rev3
[UnoWiFi]:https://store.arduino.cc/usa/arduino-uno-wifi-rev2
[Mega]:https://store.arduino.cc/usa/arduino-mega-2560-rev3
[LeonardoEth]:https://store.arduino.cc/usa/arduino-leonardo-eth
[ProMini]:https://www.sparkfun.com/products/11114
[ESPrDev]:https://www.switch-science.com/catalog/2652/
[ESPrDevShield]:https://www.switch-science.com/catalog/2811/
[ESPrOne]:https://www.switch-science.com/catalog/2620/
[ESPrOne32]:https://www.switch-science.com/catalog/3555/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[Arduino]:http://https://www.arduino.cc/
[Sparkfun]:https://www.sparkfun.com/
[SwitchScience]:https://www.switch-science.com/



