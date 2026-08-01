# Capstone Reflection — "Bed 4-A" Integrated ECG Bedside Monitor

**Author:** Brian McKinley  
**Course:** EEE 4775 Real-Time Systems  

---

## Executive Summary / BLUF
Integrating five real-time systems assignments into a single ESP32-S3 medical monitor firmware highlighted critical lessons in graceful degradation, schedule timing, and system verification.

---

## 1. What I Would Do Differently

* **Recalibrate the Arrhythmia Detector:** The variance threshold (`std > 6000.0f`) carried over from earlier assignments triggers too frequently on synthetic signals. I would calibrate it against realistic ECG waveforms to prevent continuous false triggers.
* **Record Direct Microsecond WCET Data:** Design-time WCET budgets were used for scheduling analysis. In future builds, I would log exact microsecond execution measurements for all tasks during hardware/simulation runs.
* **Model Multi-Core and Memory Interference:** The timing analysis modeled Core 1 in isolation. I would include Core 0 Wi-Fi/lwIP activity and ESP32-S3 memory/cache stalls in the response-time model.
* **Configure Simulation Parameters Upfront:** Setting switch bounce settings (`"bounce": "0"` in `diagram.json`) from the start would prevent false ISR count spikes during initial testing.

---

## 2. What Was Harder Than Expected

* **Resolving Boot-Time Watchdog Race Conditions:** Fixing a startup bug where `task_degr_manager` ran immediately on creation, compared heartbeat baselines instantly, and triggered an unnecessary fallback into `SAFE_MINIMAL` mode required adding an explicit delay and baseline timing fix.
* **Handling Hardware/Simulator Artifacts:** Switch contact bounce in Wokwi caused a single button press to fire dozens of ISR triggers until simulation properties were updated.
* **Tuning Degradation Hysteresis:** Setting up asymmetric recovery and escalation timing (300 ms escalation vs. 1000 ms recovery) was critical to stop the system from flapping between normal and degraded states during temporary load spikes.
* **Coordinating Multiple IPC Primitives:** Integrating mutexes, counting semaphores, queues, event groups, and task notifications across eight tasks without introducing priority inversion or lock contention required careful priority mapping.

---

## 3. Most Valuable Thing Learned

* **Designing for Graceful Degradation:** Safeguarding core patient functions (vitals sampling, detection, alarm outputs) by shedding non-critical tasks (telemetry, logging) first during system overload.
* **Safety Margins in Real-Time Scheduling:** Understanding that maintaining low CPU utilization (~13.4%) provides an essential safety buffer for jitter, mutex timeouts, and future expansion rather than wasted processing power.
* **Right-Sizing IPC Primitives:** Selecting synchronous direct execution for real-time alarms while using queues for telemetry and event groups for pipeline synchronization.
* **Medical Software Hazard Analysis:** Applying IEC 62304 and risk management principles, such as separating system fault indicators from patient alarm alerts to prevent clinician confusion.