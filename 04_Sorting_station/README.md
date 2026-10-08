# Factory I/O Automated Turntable Sorting Station

## Overview
This repository contains the Siemens PLC control logic for an automated turntable sorting station simulated in Factory I/O. Programmed entirely in Siemens Statement List (STL), the system dynamically tracks, queues, and sorts incoming boxes based on their height. The logic is designed to be highly robust, utilizing dynamic memory management and strict mechanical interlocks to handle variable part spacing without the use of timers.

## Key Features
*   **Dynamic Queue Management:** Utilizes a custom 32-bit Shift Register (`MD 200`) and a dedicated counter to track the sequence and sizes of up to 32 boxes simultaneously between the entry sensor and the turntable.
*   **Timer-Free Traffic Control:** Implements precision edge-triggered logic (FP/FN) to handle millimeter-level gaps between boxes. It completely decouples feed conveyors from turntable rollers to physically tear boxes apart and prevent tailgating/jamming, entirely relying on physical sensor feedback rather than arbitrary delay timers.
*   **Bitwise Evaluation:** Decodes the oldest box in the queue using bitwise arithmetic (`SRD`, `AD`) to determine sorting routing (Big vs. Small) exactly when the turntable is ready to accept a new part.
*   **Independent Motor Interlocks:** Prevents mechanical conflicts by isolating turntable rotation commands from roller loading/discharge commands.

## I/O Hardware Mapping

### Inputs (Sensors)
| Address | Symbol | Description |
| :--- | :--- | :--- |
| `I 0.1` | `S1_part_present` | Detects box entry at the start of the feed conveyor |
| `I 0.2` | `S2_height` | High-level sensor to classify box as "Big" |
| `I 0.3` | `at_turntable_entry` | Threshold sensor directly before the turntable |
| `I 0.4` | `at_load_position` | Confirms turntable is resting in its home (feed) position |
| `I 0.5` | `turntable_at_90` | Confirms turntable has successfully rotated 90 degrees |
| `I 0.6` | `at_front` | Confirms box is perfectly centered on the turntable |
| `I 0.7` | `at_left_entry` | Left discharge confirmation sensor |
| `I 1.0` | `at_right_entry` | Right discharge confirmation sensor |

### Outputs (Actuators)
| Address | Symbol | Description |
| :--- | :--- | :--- |
| `Q 0.x` | `cov_1` | Primary upstream feed conveyor |
| `Q 0.x` | `conv_2` | Secondary upstream feed conveyor |
| `Q 0.x` | `load&left` | Turntable rollers (Forward/Left discharge) |
| `Q 0.x` | `right` | Turntable rollers (Reverse/Right discharge) |
| `Q 0.x` | `turn_table` | Turntable rotation motor |
| `Q 0.x` | `left_conv` | Downstream left exit conveyor |
| `Q 0.x` | `right_conv` | Downstream right exit conveyor |

*(Note: Adjust Output `Q` addresses according to your specific hardware configuration).*

## Logic Architecture

The STL program executes sequentially through 8 primary states during each scan cycle:

1.  **System Initialization:** Master run/stop control and continuous downstream motor operation.
2.  **Entry Detection & Sizing:** Captures rising/falling edges of incoming boxes and locks in their size (Big = 1, Small = 0).
3.  **Shift Register (Queue):** Shifts the 32-bit register (`MD 200`) to store the new box's size and increments the active queue counter.
4.  **Exit & Countdown:** Detects boxes successfully clearing the left/right discharge conveyors, decrements the queue counter, and unlocks the system for the next evaluation.
5.  **Evaluation:** When the turntable is empty and home, the logic dynamically calculates the index of the oldest un-sorted box in `MD 200`, isolates its size bit, and locks in the sorting decision.
6.  **Traffic Control:** Employs an "Armed Trap" edge-trigger system. It monitors `at_turntable_entry` to instantly shut down feed conveyors when a second box tailgates, holding it safely at the threshold while `load&left` operates independently to secure the first box.
7.  **Sorting Execution:** Interlocks motor directions and drives the turntable rollers Left or Right based on the evaluation bit.
8.  **Turntable Reset:** Automatically commands the turntable to rotate back to home (`I 0.4`) immediately after a box clears the exit sensors.

## Setup & Execution
1. Open **Factory I/O** and load the turntable sorting station scene.
2. Ensure the optical sensors (`S1`, `at_turntable_entry`, etc.) are properly aligned with the conveyor thresholds.
3. Import the STL logic into **Siemens TIA Portal** or **SIMATIC Manager** (OB1).
4. Connect Factory I/O to PLCSIM or the physical PLC.
5. Trigger the `start` command to initiate the sequence. 

## Author
**Mohammed**