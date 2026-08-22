# ANTAR — The Road Notices When You Don't Arrive

> **The missing vehicle creates the suspicion. The road behind it provides the evidence.**

At 2:00 AM on a remote district road, a vehicle can crash without anyone knowing. No call. No witness. No one arrives to help.

ANTAR is a roadside safety system designed to detect that absence. Two roadside pods observe vehicle movement between them. Pod A measures a vehicle and predicts when it should reach Pod B. If it does not arrive, ANTAR does **not** immediately call it a crash. It opens a candidate event and watches the traffic behind it.

If downstream traffic remains normal, the vehicle may simply have taken a legal turn-off.

If vehicles behind it brake, slow, swerve, queue, and downstream flow drops, the road itself provides corroborating evidence.

**Same disappearance. Different evidence. Different decision.**

---

## Problem Statement

Serious crashes on remote roads can go unreported during the critical minutes after impact. Existing approaches often depend on the victim, a phone, a witness, or expensive smart-road infrastructure.

ANTAR targets the gap between **a crash happening** and **someone knowing it happened**.

The system is designed for lower-cost deployment on existing roadside poles rather than requiring an entirely new smart-road corridor.

## Solution

ANTAR uses two roadside nodes to create a monitored road segment:

1. **Vehicle detected at Pod A** — speed, size/class and a simulated vehicle signature are recorded.
2. **Arrival window opened** — Pod A predicts when the same vehicle should reach Pod B.
3. **Pod B observes traffic** — the expected vehicle either arrives or the window expires.
4. **Candidate anomaly created** — a missing arrival is treated as low-confidence; no immediate dispatch.
5. **Corroboration begins** — downstream flow, observed speed and additional missing arrivals are evaluated.
6. **Decision** — confidence rises to an alert or falls back to a cleared/legal-exit event.
7. **Transmission** — confirmed alerts travel through the modeled pod-to-pod relay chain to network coverage and responders.

The simulator deliberately keeps the crash itself outside the detector's input. ANTAR has to infer the event from simulated sensor observations rather than being told that a crash occurred.

---

## Features

### Working Road-Safety Simulation

- Three reproducible scenarios:
  - **A — Normal:** vehicle arrives normally; no alert.
  - **B — Incident:** vehicle disappears; following traffic reacts; confidence rises; alert dispatches.
  - **C — Legal Exit:** vehicle disappears; downstream traffic remains normal; candidate is cleared.
- Arrival-window prediction between Pod A and Pod B.
- Vehicle matching using class, timing and simulated signature.
- 90-second corroboration window.
- Live expected-vs-observed traffic flow.
- Downstream speed comparison.
- Confidence tiers: **LOW → MEDIUM → HIGH**.
- Simulated relay transmission with hop timing and recipient acknowledgement.

### Interactive 3D Pod

- Rugged roadside enclosure and pole-mount concept.
- ESP32 control core.
- Ultrasonic TX/RX sensing channels.
- Forward camera concept.
- RF signal catcher / antenna.
- Battery and power architecture.
- Color-coded power, ground, sensor, display and RF wiring.
- Exploded enclosure view for internal inspection.
- Component inspection/details inside the pod explorer.

### Engineering Documentation

See `docs/` for architecture, scenarios, pod design and detailed sensor-system documentation.

---

## Tech Stack

### Road Safety Simulator

- **HTML5**
- **CSS3**
- **Vanilla JavaScript**
- **HTML Canvas 2D**
- No build step
- No framework
- No backend

The main simulator is intentionally self-contained so it can be opened directly in a browser for a reliable live demo.

### 3D Pod Explorer

- **HTML5 / CSS3 / JavaScript**
- **Three.js**
- WebGL-based 3D rendering

### Architecture

The prototype is front-end only. The road simulator models the sensing and decision pipeline locally; the transmission layer is simulated rather than connected to live roadside hardware.

---

## Setup & Running

### Quick Start

Clone or download the repository, then open:

```text
index.html
