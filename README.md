# ANTAR

### The road notices when you don't arrive.

ANTAR is a roadside detection system for unlit rural roads. It works out that a crash
has happened by noticing that a vehicle which passed one pod never reached the next one —
and then checking with the traffic behind it whether the road is actually blocked.

---

## The problem

India loses roughly **1.5 lakh people to road crashes every year**. In a large share of
those deaths, the crash itself was survivable. What kills is the hour afterwards.

Trauma medicine calls it the golden hour. Internal bleeding, a blocked airway, shock —
these are treatable if someone reaches you in time. The clock does not start when the
ambulance is called. It starts at impact. Every minute nobody knows is a minute spent.

Picture a district road at 2 a.m. A vehicle leaves the carriageway. There is no
streetlight, no camera, no toll gantry, no patrol. The next vehicle may be four minutes
behind, and may not see anything off the road in the dark. Nobody sees it. Nobody calls.
The golden hour is spent before anyone even knows there is a clock running.

The existing answer to this is smart highway infrastructure — gantry-mounted cameras,
incident detection, control rooms. It works. It also costs on the order of
**₹133 crore to install and ₹52 crore a year to maintain per corridor**. At that price it
only ever gets built on expressways and flagship national highways. The roads where people
actually die — state highways, district roads, the two-lane undivided routes between
towns — will not see that infrastructure in any realistic timeframe. Not because the
technology doesn't exist, but because nobody can justify the bill for a road with
seven vehicles a minute on it.

The gap is not detection technology. The gap is detection technology cheap enough to put
where the deaths are.

---

## The inversion

Almost every crash-response product on the market waits for the victim to report.

| Approach | What it needs |
|---|---|
| In-car crash detection | A recent car with the feature fitted |
| Phone crash detection | The phone intact, charged, in coverage, and the model right |
| SOS buttons and apps | A conscious person able to reach and press it |

Each of these asks something of the person the crash just happened to. That is precisely
the thing a serious crash takes away. The cases where help is needed most urgently are the
cases where the victim is least able to ask for it.

ANTAR asks nothing of the vehicle or the person in it. It does not need them to carry a
device, install an app, or stay conscious. It watches the road instead of the car, and
it looks for an **absence** rather than an impact.

Nothing arriving is a signal. It is just a signal nobody has been reading.

---

## How the detection works

Two pods sit on existing roadside poles, a known distance apart — about **700 m** in the
current model. Each pod senses vehicles passing beneath it. It does not photograph them or
identify them. It records that something passed, how fast it was going, and roughly how
long it was.

When a vehicle passes **Pod A**, the pod does one piece of arithmetic. It knows the
distance to **Pod B**, and it just measured the vehicle's speed. From those two numbers it
computes when that vehicle should reach Pod B, and opens an **arrival window** around that
time. Pod B is told to expect it.

There are only three things that can then happen.

| Outcome | What it means |
|---|---|
| **Arrives inside the window** | Normal pass. Track closes. Nothing further happens. |
| **Arrives late but arrives** | Traffic, a slow truck ahead, a careful driver. Still fine — this is why the window has tolerance. |
| **Never arrives** | The vehicle left the segment somehow. This is the only case worth thinking about. |

The window needs tolerance, and getting that tolerance right matters more than it sounds.
A motorcycle measured at 75 km/h can end up crawling behind a tractor for the next half
kilometre. If the window is tight, that motorcycle looks like it vanished, and the system
cries wolf on ordinary traffic. So the late edge of the window is not set by the vehicle's
own speed alone — it is also floored by the slowest thing realistically sharing that road.
No vehicle can cross the segment slower than the slowest vehicle in front of it.

The result is a window generous enough that ordinary rural traffic — overtaking, bunching,
a bus pulling out, a bullock cart — stays comfortably inside it, and only a genuine
disappearance falls outside.

Every vehicle in the segment carries its own window, all running at the same time. On a
road with seven vehicles a minute, that is six or seven windows open concurrently.

Full detail: **[docs/how-it-works.md](docs/how-it-works.md)**

---

## The hard problem

Here is the part that makes this difficult, and it is the part worth judging.

**From Pod A, a crash and a legal turn-off are the same event.**

A vehicle passes Pod A. It never reaches Pod B. That is all the pod knows. The vehicle
might be in a ditch. It might equally have turned off onto a village road, stopped at a
tea stall, pulled onto the shoulder to take a call, or turned into a field track. All of
these produce an identical signal: a window that opened and never closed.

A system that alerts on every unmatched window will fire dozens of times a night on a road
with a single turn-off on it. It will be switched off within a week — and rightly so,
because a false alarm sends an ambulance away from someone who needs it.

So an unmatched window is not treated as a crash. It is treated as a **candidate**, and
nothing is dispatched. Instead, the system stops looking at the vehicle that vanished and
starts looking at the vehicles behind it.

This is the whole idea:

> **A crash changes the behaviour of the traffic behind it. A turn-off does not.**

When a vehicle is blocking a two-lane road, the vehicles that arrive next have no choice
about it. They brake hard. They slow to a crawl. They swerve across the centre line to get
past. Some stop entirely. A queue builds upstream. And critically — fewer vehicles reach
Pod B than Pod A let through, and the ones that do get through arrive slower than they
should.

When a vehicle simply turns off at a junction, none of that happens. The traffic behind
does not notice and does not react. Flow at Pod B continues to match inflow at Pod A.
Speeds hold. Nothing else goes missing.

So ANTAR does not try to detect the crash. It asks the road to corroborate it. The
evidence for a crash is not on the vehicle that disappeared — it is on the vehicles behind
it.

### Two stages

1. **Candidate.** A window expires unmatched. Confidence starts low. Nothing is
   dispatched, and the console says so explicitly.
2. **Corroboration.** For the next **90 seconds**, the system watches downstream flow
   against upstream inflow, downstream speed against upstream speed, and whether any other
   windows are going unmatched too. Confidence moves continuously as evidence accumulates.
3. **Decision.** At the end of the window — or earlier if the evidence is overwhelming —
   the accumulated confidence either clears the dispatch threshold or it doesn't.

Confidence is a continuous range, not a switch. It is reported in tiers — low, medium,
high — with the numbers that produced it shown alongside.

**If the evidence is insufficient, no alert is sent.** Not a cautious guess, not a
low-priority ping. The candidate is closed, logged with its timestamp and its reasoning,
and the system goes quiet. The cost of a false alarm stays with the system rather than
being passed to a hospital.

---

## What happens when it does decide

An alert goes to three recipients at once, not in sequence:

- **The nearest hospital** — so a bed and a trauma team know before the patient is moving.
- **The road police** — jurisdiction, traffic control, and the formal response.
- **Registered local responders** — people who have opted in, living within a couple of
  kilometres of that stretch.

The third tier is the one that is usually missing, and it is the one that matters most for
the golden hour. An ambulance dispatched from the district hospital may be twenty-five
minutes away. Someone two fields over is four minutes away. They cannot perform surgery,
but they can find the vehicle in the dark, keep an airway clear, stop bleeding, and — just
as importantly — tell the ambulance exactly where to stop.

> **The ambulance is the right responder. The neighbour is the fast one.**

You need both, and you need them started at the same moment.

---

## Deployment

The economics are the point of the design, so the hardware is deliberately boring.

- **Clamp-on retrofit.** A pod is a housing that clamps to a pole that already exists.
  No new civil works, no trenching, no foundations, no gantries. Installation is one
  person with a ladder and a spanner.
- **Layered power.** Pole supply where there is one, solar where there isn't, battery
  underneath both. Rural power is intermittent; the pod is designed to expect that rather
  than to fail on it.
- **Pod-to-pod relay.** Pods talk to their neighbours over short-range radio and hand the
  message along the chain until it reaches a pod that has cellular coverage. Only that pod
  talks to the tower. There is no SIM, no data plan and no recurring connectivity cost for
  every pod on the road.
- **Target cost: under ₹2,500 per pod.**

That last number is the entire argument. Existing incident detection is not too primitive
for these roads — it is too expensive for them. A system that costs a few thousand rupees
per pole can be deployed on the roads where the deaths actually happen.

> Pods carry it to the tower. The tower carries it to people.

---

## Resilience

Rural deployments lose nodes. Equipment is stolen, struck, flooded, or simply runs out of
battery in a bad monsoon week. A system that stops working when one pod dies is not a
system anyone should install.

When a pod goes offline, its two neighbours re-pair with each other across the wider gap.
Coverage of the road is preserved. The consequences are honest and visible:

- The segment being monitored is now longer.
- The arrival window is therefore wider and less precise.
- Confidence in any judgement about that segment **drops by one tier**, and the console
  says why.

The system degrades. It does not fail, and it does not quietly pretend it is still as
certain as it was.

---

## Running the simulation

Open **[`simulation/antar-sim.html`](simulation/antar-sim.html)** in any modern browser.
Double-click it. That is the whole process.

There is also an interactive physical pod concept at **[`pod/index.html`](pod/index.html)**.
The repository root **[`index.html`](index.html)** is a small hub that links to both demos.

No dependencies. No build step. No package manager. No server. No internet connection.
Everything — the traffic model, the detection logic, the rendering — is in that one file.

This is worth a sentence of explanation, because "it's just an HTML file" can read as a
shortcut. It isn't one. It mirrors how the product itself is meant to work: the reasoning
happens locally, on the pod, with no cloud service in the loop and no dependency on
anything being reachable. A demo that needs a server to prove a system that must work
without one would be arguing against itself.

The top half of the screen is the road — **ground truth**, what actually happened.
The bottom half is the console — **only what ANTAR can actually perceive**. The
simulation never tells the detector that a crash occurred. The detector has access to
exactly two kinds of event: a vehicle passed Pod A, and a vehicle passed Pod B. Everything
else it concludes, it concludes from those.

That separation is deliberate, and it is the thing to check if you are sceptical. The
detection logic is not reading the scenario you picked.

---

## The three scenarios

One button each. Everything else — traffic generation, speeds, timing, pod communication —
runs on its own.

### Scenario A · Nominal
A vehicle passes Pod A and arrives at Pod B as predicted. The track closes. No candidate,
no alert, nothing sent.

*Demonstrates: the normal path, which is what happens to virtually every vehicle. A system
that is quiet almost all the time is a system that works.*

### Scenario B · Incident
The tracked vehicle stops on the carriageway between the pods. Its window expires
unmatched. The traffic behind it brakes hard, bunches into a queue, swerves around the
site, and one vehicle stops entirely. Downstream flow at Pod B falls well below inflow at
Pod A, arriving vehicles are much slower than upstream, and further windows start going
unmatched. Confidence climbs into the high tier and the alert dispatches.

*Demonstrates: corroboration working. The system did not see the crash — it inferred a
blockage from the behaviour of vehicles that were never involved in it.*

### Scenario C · Legal exit
The tracked vehicle turns off onto the village road. Its window expires unmatched, exactly
as in Scenario B — the candidate opens identically. But the traffic behind is completely
unaffected. Flow holds, speeds hold, nothing else goes missing. After the full 90 seconds,
confidence is still low. The candidate is dismissed and logged. No alert.

*Demonstrates: the discrimination that makes the system deployable. B and C are
indistinguishable at the moment of absence and are separated only by evidence gathered
afterwards.*

Scenarios B and C are the pair to watch together. If a judge only has time for one thing,
it is running B and then C and noticing that the first thirty seconds are identical.

Detail on all three: **[docs/scenarios.md](docs/scenarios.md)**

---

## What is proven, and what is not

We would rather be trusted than impressive, so here is the honest boundary.

**Demonstrated in this repository**

- The detection loop end to end: measure, predict, open a window, match or fail to match.
- Concurrent tracking of every vehicle in the segment, not one at a time.
- The discrimination logic — crash versus legal exit — from corroborating evidence only.
- Continuous confidence with an explicit dispatch threshold, and refusal to alert when the
  evidence isn't there.
- The relay and hand-off model from pod to pod to tower to recipients.
- A decision console that shows its reasoning step by step, in plain language, with
  timestamps on every event including dismissed candidates.

**Not yet done**

- **Field validation.** The traffic model is a model. Real rural traffic will be messier,
  and the tolerance and confidence parameters will need tuning against real road data.
- **Hardware.** The pod is specified and the firmware track is scoped, but no pod has been
  built and mounted on a pole.
- **Sensor selection.** The simulation assumes a sensor that can time a passage, estimate
  speed and length, and produce a repeatable signature. Choosing the actual sensor and
  characterising it in weather is real work that has not been done.
- **Weatherproofing, certification, road-authority approval.** None of this is started.
- **Emergency service integration.** Dispatching to a real hospital or control room needs
  agreements and interfaces that a hackathon cannot produce.
- **Adversarial cases.** Livestock, waterlogging, a stopped vehicle that is not a crash,
  festival traffic. Some of these will produce false positives and need work.

This is a hackathon prototype. It demonstrates that the idea is sound and that the hard
part — telling a crash apart from an ordinary turn-off — has a workable answer. It does
not demonstrate a product ready for a road.

---

## Repository map

```
antar/
├── README.md                     you are here
├── LICENSE                       MIT
├── index.html                    local demo hub
├── simulation/
│   └── antar-sim.html            road-safety simulator
├── pod/
│   └── index.html                interactive 3D pod explorer
├── docs/
│   ├── how-it-works.md           the detection algorithm in detail
│   ├── architecture.md           system structure and privacy position
│   └── scenarios.md              what each scenario proves
├── hardware/
│   ├── README.md                 the physical pod track
│   ├── firmware/                 ESP32 source (to be added)
│   ├── diagrams/                 circuit diagrams (to be added)
│   └── simulation/               VirtualBox / emulation notes (to be added)
└── assets/                       screenshots
```

---

## Team

**Madhur** — system logic, simulation, design.
**Aditya** — backend, storage, hardware.

---

## Licence

MIT. See [LICENSE](LICENSE).

- [Sensor Systems & Working](docs/SENSOR_SYSTEMS.md) — detailed sensing, wiring, and component behaviour
