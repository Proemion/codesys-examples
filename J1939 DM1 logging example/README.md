# J1939 DM1 logging example

This example contains how to read DM1 messages  using the IoDrvj1939 function block on CAN1 and CAN2 and use the proemion library to log them on change.

## J1939 Setup and Configuration

### Hardware Mapping
Ensure your CODESYS project has the J1939 stack configured:

 * Add a J1939 Manager to your CAN interface.

* Add a Remote ECU under the manager.

* Note the name of the ECU in the device tree (e.g., J1939_ECU_Receiver).

### Implementation
In the provided code, ensure the itfSourceECU input of the fbDM1 block points to your ECU:

    fbDM1.itfSourceECU := J1939_ECU_Receiver; // Match your device tree name
    
The fbDM1 block listens to the J1939 stack. When a full message arrives (even a multi-packet one), fbDM1.xReceived triggers for one scan.

The fbWriter block takes the raw interface data and converts it into a readable array of J1939.DTC structures. The code loops through the active DTCs and packs them into a byte array (aDM1_Bytes). It then compares this array to the previous cycle. Then it also logs them on change. 

