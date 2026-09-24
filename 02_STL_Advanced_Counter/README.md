# Siemens S7-300 Accelerated Up/Down Counter (Pure STL)

## Overview
This repository contains the source code (`.awl`) for an advanced, custom-built up/down counter programmed entirely in Siemens Statement List (STL). Designed for a SIMATIC S7-300 CPU, this logic bypasses standard PLC counter blocks to accommodate values that exceed the standard 16-bit integer limit. The counter value is stored in a 32-bit double word, allowing it to safely track numbers up to 100,000.

## Core Features
* **Dynamic Acceleration:** Pressing and holding the UP or DOWN inputs increments or decrements the counter by 1 every second. After a continuous 5-second hold, the step size automatically accelerates to 5 per second.
* **Self-Clearing Timers:** Utilises a non-retentive On-Delay timer (`SD`) that evaluates the logic state continuously. The exact millisecond the input is released, the timer drops instantly, returning the acceleration state to baseline without requiring complex manual reset logic.
* **Single-Scan Pulse Generation:** Features a custom 1-second oscillator. By routing the timer's output bit through a memory marker, the logic generates a pulse that remains high for exactly one scan cycle, ensuring arithmetic operations execute only once per second regardless of the PLC's overall scan time.
* **Post-Arithmetic Clamping (Hard Limits):** To prevent mathematical overflows or out-of-bounds errors, limit bounds are enforced *after* the arithmetic blocks. If an operation pushes the counter above 100,000 or below 0, double-integer comparison instructions (`<=D`, `>=D`) immediately intercept and clamp the value back to the absolute limit within the same scan cycle.
* **Master Reset:** A dedicated master switch instantly forces the memory double word back to zero when deactivated.

## Hardware Address Mapping
| Input / Address | Function |
| :--- | :--- |
| `I 0.0` | Master Switch ON/OFF (Resets to 0 when OFF) |
| `I 0.1` | UP Button (Hold to increment) |
| `I 0.2` | DOWN Button (Hold to decrement) |
| `MD 10` | 32-bit Counter Storage (Min: 0, Max: 100,000) |
| `T 1`   | 5-Second Acceleration Timer |
| `T 2`   | 1-Second Pulse Timer |
| `M 0.1` | Acceleration Status Bit (High after 5 seconds) |
| `M 0.2` | 1-Second Single-Scan Pulse Bit |

## Implementation Details
The logic relies strictly on double-integer maths (`+D`, `-D`) and 32-bit constants (e.g., `L L#100000`). For a complete breakdown of the scan-cycle execution and timer reset behaviour, refer to the inline comments within the `.awl` source file.