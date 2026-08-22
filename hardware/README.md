# Hardware

This half of the project answers a different question from the simulation.

The simulation proves the **logic**: that an absence can be detected, and that a crash can
be told apart from an ordinary turn-off using corroborating evidence from the traffic
behind. That is the hard conceptual problem, and it is the part that has to be right
before anything else matters.

The hardware track proves the **pod is buildable** — that everything the logic assumes a
pod can do is achievable on a microcontroller costing a few hundred rupees, running on
intermittent rural power, mounted on a pole.

The two tracks are being developed in parallel and converge at integration: the firmware
implements the same detection loop the simulation demonstrates, against real sensor input
instead of modelled traffic.

Nothing in this directory is finished. It is scoped, not built.

---

## What goes where

| Folder | Contents |
|---|---|
| `firmware/` | ESP32 source — sensor reading, timestamping, pod-to-pod protocol, display driver, power management |
| `diagrams/` | Circuit diagrams, sensor wiring, power layering, enclosure and clamp mounting |
| `simulation/` | VirtualBox / emulation notes for running and testing firmware without physical hardware |

---

## What a pod has to do

Five jobs. Each one is a requirement the detection logic depends on, so each is worth
stating in terms of what would break if it were not met.

### 1. Read the sensor array

Detect a vehicle passing beneath, and from that passage derive speed, a coarse length
class, and a repeatable signature.

*Why it matters:* speed determines the arrival window, length class is a matching gate,
and the signature is what allows a second pod to confirm the same vehicle arrived rather
than merely a vehicle. Without the signature, concurrent tracking of six or seven vehicles
in a segment becomes guesswork.

*Open question:* which sensor. The logic assumes measurements that a dual inductive loop
or a radar unit can provide, but selecting the actual part and characterising it in rain,
heat, dust and pole vibration is unstarted work.

### 2. Timestamp the passage accurately

Record when the vehicle passed, with clocks well enough aligned between paired pods that
arrival windows mean the same thing at both ends.

*Why it matters:* the entire method is arithmetic on time differences. Clock drift between
two pods translates directly into a window offset, which turns into either missed
detections or false absences. Over a 700 m segment with windows tens of seconds wide, the
tolerance is not tight — but it is not unlimited, and drift accumulates over weeks
unattended.

### 3. Exchange arrival windows with the neighbour pod

Short-range radio link. Pod A sends the expected arrival window and signature; Pod B
matches against its own passages and reports back.

*Why it matters:* this is what makes the pair a detector rather than two counters. It also
carries the alert when one is raised — the same link is the relay path to the pod that has
cellular coverage. There is no SIM in every pod; the message hops until it finds one.

### 4. Drive the display face

An amber dot-matrix panel, angled toward oncoming traffic. Status in normal operation, a
warning to approaching drivers when an incident is confirmed ahead.

*Why it matters:* the pod is not only a sensor. A driver approaching a blind blockage at
night who slows down early is safer, and is less likely to become the second crash. It is
also the part of the system the public actually sees, which matters for trust.

### 5. Manage layered power

Pole supply where one exists, solar where it does not, battery beneath both, with clean
handover between them and honest reporting of remaining capacity.

*Why it matters:* rural power is intermittent, and a pod that treats an outage as a fault
is a pod that spends half its life offline. The system also needs to know when a pod is
about to go dark, because a pod dropping out changes what its neighbours can conclude —
the segment widens, the window loosens, and confidence in that stretch drops a tier.

---

## Cost target

**Under ₹2,500 per pod.**

This is a constraint on the design rather than a hope about it. The entire argument for
ANTAR is that existing incident detection is not too primitive for rural roads, it is too
expensive for them, at roughly ₹133 crore to install and ₹52 crore a year to maintain per
corridor. A system that solves the detection problem elegantly but costs ₹40,000 a pole
has not solved the actual problem.

Every hardware decision — clamp-on rather than new civil works, relay rather than a SIM
per pod, microcontroller rather than an embedded PC, on-pod decisions rather than a cloud
service — is downstream of that number.
