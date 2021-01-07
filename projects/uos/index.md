[![NullTek Documentation](../../resources/NullTekDocumentationLogo.png)](https://creatingnull.github.io)

---

# UART Operating System

The UART Operating System (UOS) is a project that allows dynamic real-time control over embedded systems operation, using the Null Packet Comms Protocol.

This system aims to provide an all encompassing serial controlled device, which requires little to no further development for new applications. 
The entire system can be configured in runtime over UART, pins can be mapped levels set ect. 
The AOS system uses blanket remote address distribution, so address 1-255 are all uses for the embedded system. 
Address 0 always refers to the Host device. 
A command is sent over UART any time you need to set or request some information from the UOS device. 

## Device Implementations

* [Arduino (Device 0)](arduino.md) - Running on top of the arduino bootloader, designed for ATMega328, namely the Uno and Nano.

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

| Address | Volatility       | Name / Description                                                                                                           | Payload                                               | Response                                                        | 
| :------ | :--------------: | :--------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------  | :-------------------------------------------------------------- |
| 64      | Super-Volatile   | Digital IO Control - Over_Rides onboard settings, doesn't update RAM values and can be overwritten.                          | 3 bytes per-pin. Pin index, IO type, level            | ACK (, boolean packet if IO Type = 1)                           |
| 68      | Volatile         | Reset IO from RAM                                                                                                            | None                                                  | ACK Packet                                                      |
| 85      | N/A              | Sample ADC - 10 bit little endian read.                                                                                      | 1 byte per-pin. Pin index.                            | ACK Packet, 2n little endian byte data packet                   |
| 250     | N/A              | Get UOS Version - Gets the version infomation of the embedded software and device type `[PATCH][MINOR][MAJOR][DEVICE INDEX]` | None                                                  | ACK Packet, 4 byte data packet.                                 |
