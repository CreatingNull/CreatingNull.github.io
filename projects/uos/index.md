[![NullTek Documentation](../../resources/NullTekDocumentationLogo.png)](https://creatingnull.github.io)

---

# ![UOS Logo](../../resources/UOSBlackAndRedSmall.png) UOS Remote Control System

The UOS remote control system is a project that allows dynamic real-time control over embedded systems operation, using the Null Packet Comms Protocol.

This system aims to provide an all encompassing serial controlled device, which requires little to no further development for new applications. 
The entire system can be configured in runtime over UART, pins can be mapped levels set ect. 
The AOS system uses blanket remote address distribution, so address 1-255 are all uses for the embedded system. 
Address 0 always refers to the Host device. 
A command is sent over UART any time you need to set or request some information from the UOS device.

## Device Implementations

* [Arduino (Device 0)](arduino.md) - Running on top of the arduino bootloader, designed for ATMega328, namely the Uno and Nano.

## Interfaces

* [Official Interface](interface.md) - Python interface for communicating with UOS devices.

## System Architecture

The system is segregated using addresses to refer to specific functionality. 

At a high level the system is mapped as:

*   0-19: Reserved for very low level register functionality 
*   20-29: Reserved for EEPROM operations
*   30-49: Reserved for RAM operations
*   50-59: Reserved for future functionality
*   60-79: Reserved for DIO operations
*   80-89: Reserved for AIO operations
*   100-109: Reserved for NPC 1 Wire IO operations
*   110-249: Reserved for future functionality
*   250-255: High level UOS System Information.

For a rule of thumb you can consider higher numbered operations to be 'higher-level' functionality.

## Volatility Paradigm

This system uses several 'authorities' for configuring the microcontroller, known as volatility levels. 
These volatility levels change how the microcontroller processes instructions and how deeply they get committed to memory.

*   **Super-Volatile Level.** 
    This level is purely over-riding the current settings on the device. 
    This will be persistent until the system reboots or reloads saved settings. 
    It is not committed to memory at all. 
    This is good for temporary configurations. 
    
*   **Semi-Volatile Level.** 
    This level over-rides the current settings and saves those settings to ram on the device. 
    This means if the setting is over-written by a super volatile write, the settings can be refreshed from RAM, however they will still be cleared if the system is restarted. 
    This is good for runtime default settings that need to be reset periodically during a session.
    
*   **Non-Volatile Level.** 
    This level saves the configuration as the pin defaults to the controllers EEPROM. 
    This means the configuration will be loaded as default on each boot. 
    This is useful for changing the persistent default behaviour of the controller from its factory defaults. 
    (Generally this shouldn't be set by an automated process as EEPROM has a limited number of write cycles).

## Address Map

The following table enumerates currently accepted instructions. 

| Address | Volatility       | Name / Description                                                                                                           | Payload                                               | Response                                                        | 
| :------ | :--------------: | :--------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------  | :-------------------------------------------------------------- |
| 64      | Super-Volatile   | Digital IO Control - Over_Rides onboard settings, doesn't update RAM values and can be overwritten.                          | 3 bytes per-pin. Pin index, IO type, level            | ACK (, boolean packet if IO Type = 1)                           |
| 68      | Volatile         | Reset IO from RAM                                                                                                            | None                                                  | ACK Packet                                                      |
| 85      | N/A              | Sample ADC - 10 bit little endian read.                                                                                      | 1 byte per-pin. Pin index.                            | ACK Packet, 2n little endian byte data packet                   |
| 250     | N/A              | Get UOS Version - Gets the version information of the embedded software and device type `[PATCH][MINOR][MAJOR][DEVICE INDEX]`| None                                                  | ACK Packet, 4 byte data packet.                                 |
| 251     | N/A              | Get Digital Pin Config - Returns the current, ram, and eeprom settings for the pin `[MODE][LEVEL][MODE][LEVEL][MODE][LEVEL]` | 1 byte per-pin. Pin index.                            | ACK Packet, 6n byte data packet.                                |

## Pin Modes

Pins modes are defined using the following int look-up table when referenced in UOS packets.

### Digital Pins

0. Digital Output
1. High Impedance Input
2. 255 Not configured or unknown pin

### Analog Pins

1. DAC Output
2. ADC Input
2. 255 Not configured or unknown pin

## EEPROM Usage

Most implementations use onboard EEPROM to store non-volatile information. 
The system is designed to use less than 4K Bytes.

The following address spaces are used:

* 100-200: Reserved for GPIO non-volatile configuration. 
  Even indexes show [pin mode](#digital-pins), odd show level. 
  eg: Pin 13 level is located at address `index = 100 + (2 * 13 + 1)`.
