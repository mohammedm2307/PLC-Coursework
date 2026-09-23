# Advanced STL Up/Down Counter

## Objective
Design a custom accelerated up/down counter strictly using math operation blocks and Statement List (STL) programming, without relying on standard PLC counter blocks.

## Hardware Inputs & Addressing
* **ON/OFF Switch:** `I0.0`
* **Up Push Button:** `I0.1`
* **Down Push Button:** `I0.2`

## Logic Workflow
* **Active State:** The system only operates when the switch (`I0.0`) is in the "ON" mode. 
* **Reset Condition:** If the switch is turned to the "OFF" state at any point, the counter value immediately returns to zero.
* **Counting Behaviour:** 
  * When the Up or Down push button is held, the screen value continuously increases or decreases.
  * **First 5 seconds:** The value increments/decrements by `1` every second.
  * **After 5 seconds:** The acceleration kicks in, incrementing/decrementing by `5` every second.
* **Strict Boundaries:** The counter must not exceed a maximum limit of `100,000` and cannot drop below a minimum limit of `0` under any circumstances.

## Technical Constraints
* Standard PLC counter blocks are strictly prohibited.
* Logic must be built exclusively using Math Operation blocks.
* Code must be written entirely in Statement List (STL).
* The accumulated value must be stored using only one Memory Double Word (MD).