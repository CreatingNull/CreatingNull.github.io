## Null Packet Comm's Protocol

I got sick of writing one off unique serial communication protocols during in my honors year in 2015.
So I laid out a set of guidelines for a generic packet structure that I could use in any project. 
This allows the communication between devices to be standardised, 
and to provide more robust error detection than typical UART handlers.

A common, reusable, rigid packet-based system trumps the one-off string parsing UART systems I consistently see being deployed. 
The NPC protocol is still simple enough that computational overhead is still relatively low for most embedded application.

### Architecture

The protocol relies on omni-directional binary packets being transmitted over a serial protocol. 
The packets are addressed, however in most configurations there are only two nodes involved in the communication (a host and a remote).

#### Packet Structure

_Enclosed square brackets `[]` are used to denote a byte._

`[start packet symbol][to address][from address][payload length]payload len*[payload byte][checksum][end packet symbol]`

1. **Start packet symbol**. 
   This one byte is a convenience-char used to trigger the start of packet decoding. 
   It is represented by the char '>' or 0x3E. 
   It is not a unique char and can appear elsewhere in the packet. 
   It is required that checksums and packet lengths are still validated to ensure device to device serial synchronisation.
2. **To address**. 
   This is a one byte address that is used to prompt a remote system on which action the serial packet is targeting (ie: a port expander, specific IO function, OS command ect). 
   Generally the embedded system is build to accept addresses 1-255, and the local host system is built to only use the address 0. 
   However, in the case of bidirectional control these address could be divvied by arbitrarily.
3. **From address**. 
   Also a one byte address denoting where / which subsystem the serial packet originated.
4. **Payload length**. 
   A one byte numeric indicator of how many bytes the packet has in its payload. 
   This is a binary number between 0-255, however with most implementations total packet size is less than 32 bytes. 
   Usually the payload is only several bytes in length.
5. **Payload data**. 
   x data bytes where x is the payload length from the prior byte. 
   Note there is no restriction on the format of the payload besides defining the length. 
   Format is system dependant.
6. **Checksum**. 
   This is just your standard LRC checksum. 
   It is generated off all the bytes in the packet, besides the start symbol, end symbol and checksum bytes.
7. **End packet symbol**. 
   This one byte is a convenience-char used to signal the end of a packet. 
   It is represented by the char `<` (0x3C). 
   Like the start packet symbol, this is not a unique char and can appear elsewhere in the packet.

#### Error Detection

To get the best coverage of errors several steps can be taken. These are all built into the library on the remote side but the host side should also use these techniques.

##### Checksum validation: 

Using the LRC checksum byte provides a method to validate the format and data existing in the packet.
This is just using ISO 1155 compliant longitudinal redundancy checking.

With a zeroed initial LRC byte each byte is added to LRC, masking overflowed bits (this may not be required for 8-bit unsigned numerics on platforms with safe overflow-handling, but it is safer to always mask if you are unsure).

`LRC = (LRC + packet[byte_index]) & 0xff`

This is done in sequential order from the start of the packet to the end (ignoring the start end and checksum bytes). Once all the bytes have been included, we then take the twos compliment of the LRC value to get our final checksum. [Exclusive or plus one].

`LRC = ((LRC ^ 0xff) + 1) & 0xff`

This should be done on be done on both TX and RX packets on both ends of the serial link.

##### Payload Length validation: 

Using the known position of the payload length byte, this numeric value should be used to validate against the received length of the payload.
Once the packet sequence has been completed and received, the payload length should be validated to ensure synchronisation with the serial bit-stream.

##### Start and end symbol validation: 

Once the packet has been captured, depending on the method used, the start and end symbol should be checked against the known 0x3E and 0x3C known values.

##### Acknowledge packets: 

The comms system provides a-symmetrical ACK packets, so when it has processed a command it will return a ACK/NACK to indicate that the data it received was formatted correctly, there were no bitwise errors and the action was completed.
This packet is formatted in accordance with the protocol, however there will just have a singular databyte, with a value of 0 (ACK), or a NACK error code. This will be sent from the target address of the command it processed. This system is asymmetrical as the embedded system does not require / work with a acknowledge packets being sent to it in response, and you only should handle receiving them.

###### NACK/ACK Codes:

0. ACK Acknowledge: Successful response to the packet. 
1. Packet Receive Error: The device did not receive the packet without errors. 
2. Process Instruction error: There was a problem while handling the instruction on the device.
