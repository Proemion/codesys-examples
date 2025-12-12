# J1939 Example

This example configures two J1939 instances in CAN1 and CAN2, both addresses = 0

Reads DM1 (16#FECA) and EEC1 (16#F004) from both CAN1 and CAN2. 

Uses Proemion CANlink_mobile CODESYS library to log the "EngSpeed" in "EEC1_F004_CAN1" and the "ActualEngPercentTorque_1" in "EEC1_F004_MSG"

In PRG "log_DTCs_DM1" an aututomatic logic to detect changes in DM1 is presneted, this logic also logs DM1 source address 0 if the payload changes. 