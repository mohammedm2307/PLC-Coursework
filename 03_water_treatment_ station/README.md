# Water Treatment Station PLC Control

## Overview
This project features an automated control system for a water treatment and filtration station, programmed in Siemens Ladder Logic (LAD). The system manages raw and treated water tanks, coordinates multiple pumps and valves, and executes complex time-based and condition-based filter backwash sequences.

## System Architecture
* **Tank 1 (Raw Water):** Stores incoming untreated water.
* **Tank 2 (Treated Water):** Stores filtered water.
* **Sensors:** Each tank is equipped with Low, Medium, and High level sensors. Two analog pressure sensors (PS1 and PS2) monitor the filter's condition.
* **Actuators:** The station utilizes multiple valves (V1-V4) and pumps (P1, P2, P4, P5, P6) to control flow, filtration, and chemical dosing.

## Core Control Logic & Workflows

### 1. Level Control & Auto-Refill
* **Tank 1:** If the water level drops below the Medium sensor, Valve 4 opens to fill the tank until the High sensor is triggered.
* **Tank 2:** If the level drops below Medium, Valves 1 and 2 open. Pump 1 then pumps water through the filter into Tank 2 until the High sensor triggers. 
* **Dry-Run Interlock:** The filling operation for Tank 2 is strictly interlocked and will immediately halt if Tank 1's Low sensor reads 0 (empty).

### 2. The 5-Second Valve Delay Rule
To protect the piping network from water hammer and dead-heading, a strict sequencing rule is enforced across the entire program: **valves must be fully open for 5 seconds before any corresponding pump is permitted to start.**

### 3. Time-Based Water Backwash (24-Hour Cycle)
To prevent standard filter clogging, an automated water backwash runs every 24 hours:
1. Valve 3 opens.
2. After 5 seconds, Pump 2 draws clean water from Tank 2 to backwash the filter.
3. The wash continues until Tank 2 reaches the Low level (sensor = 0).
4. Valve 3 and Pump 2 shut down, and the standard Tank 2 refill sequence initiates to recover the water level.

### 4. Condition-Based Chemical Backwash (State Machine)
If standard backwashing is insufficient, contaminants cause a pressure differential across the filter. A comparator monitors PS1 (pre-filter) and PS2 (post-filter). If `(PS1 - PS2) > 1.5 bar` for more than 5 continuous minutes, a multi-stage chemical wash triggers.

* **Prerequisite:** Tank 2 must be completely full (High = 1) before the chemical sequence is allowed to start.
* **Stage 1 (Pump 4):** Valve 3 opens for 5 seconds. Pumps 2 and 4 turn on until the Medium level is reached. Pump 4 turns off, and Pump 2 continues until the Low level. The system then completely refills Tank 2.
* **Stage 2 (Pump 5):** The exact same draining and washing sequence repeats using Pump 5, followed by another full Tank 2 refill.
* **Stage 3 (Pump 6):** The sequence repeats a final time using Pump 6, followed by a final system reset and refill.

## Repository Contents
* `Water_Treatment_Source.awl` - The compiled source code containing all OB1 and FC logic blocks.
* PDF documentation of the visual Ladder Logic networks.
