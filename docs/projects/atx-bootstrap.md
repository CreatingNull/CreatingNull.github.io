---
layout: default
title: ATX Bootstrap
nav_order: 3
---

# ATX PSU Bootstraping Microcontroller

Status: Concept

ATX power supplies are very useful for integrating with projects, as they supply stable 3V3, 5V and 12V rails as an off the shelf product.
However, there are a couple of rigid characteristics that make them a little tricky to use, one of which is their shutdown characteristics.
By [protocol](https://www.intel.com/content/dam/www/public/us/en/documents/guides/power-supply-design-guide-june.pdf) ATX power supplies enter a latched shutdown state when a fault occurs. 
While there are a number of circumstances that can cause this, one annoying one is input power being removed.
This means the default behaviour of an ATX power supply is to remain off after a brownout, unless instructed otherwise via an active low cycled pulse PS_ON# TTL line.
In many scenarios this is not desirable, as you would like what ever is being supplied to be powered on after the fault has been resolved.

In their primary use case, as computer power supplies, the motherboard will handle this logic. 
Often providing you with the ability to alter the behaviour in such circumstances through the bios.
This project aims to provide the same functionality but via a single low-cost IC.

The suggested method to avoid shutdown on ATX power supply is by cycling a 10 - 100ms pulse on the PSU_ON# signal during decay of the power rails (3.3.3).
Alternatively, if you end up in the shutdown state with 5VSB active and PSU_OK disasserted, you can leave the PSU_ON# signal high for 1s before pulling it low to reboot the supply (3.4). 
