# Vision-Language-Action Models for Autonomous Driving

> **Personal overview / study notes** on how Vision-Language-Action (VLA) models are researched and applied in autonomous driving.  
> This first part establishes the foundation of **driving automation levels according to SAE J3016** before moving into VLA.

---

# 1. Levels of Driving Automation

## 1.1. Why do we need to distinguish different levels of driving automation?

The terms **“self-driving car”**, **“autonomous vehicle”**, and **“driverless car”** are often used broadly in media and everyday discussion. However, in research and industry, driving automation is commonly described using **SAE J3016 – Taxonomy and Definitions for Terms Related to Driving Automation Systems for On-Road Motor Vehicles**.[^sae-j3016]

SAE defines **six levels of driving automation, from Level 0 to Level 5**:

```text
Level 0 ── Level 1 ── Level 2 ── Level 3 ── Level 4 ── Level 5
   │          │          │          │          │          │
   └──────── Driver Support ────────┘          │          │
                         └──── Automated Driving ──────────┘
```

According to the official SAE chart, **Levels 0–2** belong to the group in which the human remains the driver and must supervise the system. **Levels 3–5** are the levels in which the Automated Driving System (ADS) performs the complete **Dynamic Driving Task (DDT)** while the automated-driving feature is engaged.[^sae-chart]

> **The most important transition is not Level 4 → Level 5, but Level 2 → Level 3.**  
> At Level 2, the human driver is still responsible for monitoring the driving environment. At Level 3, the automated driving system performs the complete DDT within its designed operating conditions, although the human must be prepared to take over when requested.[^sae-summary][^nhtsa-levels]

---

## 1.2. Three concepts to understand first

### Dynamic Driving Task — DDT

The **Dynamic Driving Task (DDT)** refers to the operational and tactical functions that must be performed in real time to operate a vehicle on the road.

According to SAE, the DDT includes functions such as:

- lateral vehicle motion control — **steering**;
- longitudinal vehicle motion control — **acceleration and deceleration**;
- monitoring the driving environment;
- detecting and responding to objects and events;
- planning tactical maneuvers;
- signaling and other operational driving behaviors.

The DDT **does not include** strategic functions such as choosing a destination or scheduling a trip.[^sae-summary]

A simplified view is:

```text
Dynamic Driving Task
│
├── Lateral control
│      └── Steering
│
├── Longitudinal control
│      ├── Acceleration
│      └── Braking
│
├── Environment monitoring
│      └── Object & Event Detection and Response
│
└── Tactical maneuvering
       ├── Lane change
       ├── Yield
       ├── Overtake
       └── Turn
```

---

### Object and Event Detection and Response — OEDR

**Object and Event Detection and Response (OEDR)** is the part of the DDT responsible for identifying relevant objects and events in the environment and responding appropriately.

Conceptually:

```text
Detect
   ↓
Recognize / Classify
   ↓
Understand the situation
   ↓
Prepare a response
   ↓
Execute an appropriate response
```

For example:

```text
Camera detects a pedestrian
          ↓
Pedestrian is approaching a crosswalk
          ↓
Possible crossing event
          ↓
Vehicle prepares to yield
          ↓
Decelerate / stop
```

This concept is particularly important for VLA because many VLA driving models attempt to integrate:

```text
Perception → Reasoning → Planning → Action
```

within a unified multimodal model.

---

### Operational Design Domain — ODD

The **Operational Design Domain (ODD)** describes **the operating conditions under which a driving automation system or feature is specifically designed to function**.

These conditions may include:

- geographic area;
- road type;
- vehicle speed;
- time of day;
- weather;
- lighting conditions;
- traffic conditions;
- infrastructure characteristics.[^odd]

For example, an automated-driving feature may have an ODD such as:

```text
Highway only
+ daytime
+ clear weather
+ mapped roads
+ speed ≤ 100 km/h
```

A robotaxi service may instead operate under:

```text
Geofenced urban area
+ selected streets
+ speed ≤ 50 km/h
+ no severe weather
```

**Level 4 can still be limited by an ODD.**  
**Level 5**, by definition, refers to full driving automation that can perform the driving task under all roadway and environmental conditions that a human driver could manage, rather than being restricted to a specific ODD in the same way as typical Level 3 or Level 4 systems.[^nhtsa-levels][^odd]

---

# 2. SAE Levels 0–5

## Level 0 — No Driving Automation

At **Level 0**, the human performs the entire driving task.

The vehicle may still provide:

- warnings;
- momentary assistance;
- emergency intervention.

Examples used by NHTSA to explain Level 0 include:

- Forward Collision Warning;
- Lane Departure Warning;
- Automatic Emergency Braking.[^nhtsa-levels]

```text
Human
 ├── Steering
 ├── Acceleration
 ├── Braking
 └── Environment monitoring

Vehicle
 └── Warning / momentary assistance
```

### Important note

Under SAE J3016, active-safety systems that intervene only **momentarily**, such as Automatic Emergency Braking, are not considered driving automation in the sense of continuously performing part of the DDT.[^sae-j3016]

---

## Level 1 — Driver Assistance

At **Level 1**, the system can continuously assist with either:

```text
Steering
   OR
Acceleration / Braking
```

but it does **not simultaneously perform both** as a Level 2 feature would.[^sae-chart]

Typical examples include:

- Adaptive Cruise Control;
- Lane Centering / Lane Keeping Assistance.

```text
                 Human driver
              monitors environment
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
       Steering              Accel/Brake
          │                       │
        Human                  System

              OR vice versa
```

The human driver remains responsible for the driving task and must continuously supervise the system.[^nhtsa-levels]

---

## Level 2 — Partial Driving Automation

At **Level 2**, the system can simultaneously perform:

```text
Steering
   +
Acceleration / Braking
```

A typical example is the combination of:

```text
Lane Centering
      +
Adaptive Cruise Control
```

However:

> **Level 2 does not mean that the vehicle can drive itself without human supervision.**

The human driver must still:

- watch the road;
- supervise the system;
- detect hazardous situations;
- intervene with steering or braking when necessary.[^nhtsa-sgo]

```text
              Level 2

Camera / Radar / Sensors
          ↓
    Driver-assistance
          ↓
 ┌────────┴─────────┐
 ↓                  ↓
Steering        Accel/Brake
 │                  │
 └──────────┬───────┘
            ↓
         Vehicle

BUT

Human driver
     ↓
continuously monitors environment
     ↓
remains responsible
```

NHTSA clearly distinguishes **Level 2 ADAS** from **ADS Levels 3–5**. Level 2 provides partial driving automation to an attentive human driver, whereas an ADS is intended to perform the complete DDT within its operating conditions.[^nhtsa-sgo]

---

# 3. The Critical Transition: Level 2 → Level 3

This is one of the most important boundaries in autonomous-driving research.

```text
LEVEL 2                             LEVEL 3
────────                            ────────

System controls                     System performs
steering + speed                    complete DDT
       │                                  │
       ▼                                  ▼
Human monitors                       ADS monitors
environment                          environment

Human is driving                    System is driving
```

SAE emphasizes that the key distinction between Level 2 and Level 3 is **who performs the complete Dynamic Driving Task**.[^sae-summary]

---

## Level 3 — Conditional Driving Automation

At **Level 3**, while the ADS is operating inside its ODD:

- the system performs the complete DDT;
- the system monitors the driving environment;
- the human does not need to continuously perform the DDT;
- **but the human must be ready to take over when the system issues a request to intervene**.[^nhtsa-levels]

```text
Normal operation

Sensors
   ↓
   ADS
   ↓
Perception
   ↓
Planning
   ↓
Control
   ↓
Vehicle


If ADS reaches its limit:

ADS
 ↓
Request to intervene
 ↓
Human driver
 ↓
Take over
```

Therefore, Level 3 introduces a particularly important research problem:

> **Takeover / handover:** can the human driver respond quickly and correctly enough when the ADS requests intervention?

---

## Level 4 — High Driving Automation

At **Level 4**, the system can perform the complete DDT **and does not depend on the human driver to perform the fallback task** within the ODD it supports.[^sae-chart]

A person inside the vehicle may therefore function only as a passenger.

However, Level 4 **does not mean that the vehicle can operate everywhere**.

For example, the ODD may restrict operation to:

```text
Robotaxi

Allowed:
✓ San Francisco geofence
✓ mapped urban roads
✓ supported weather
✓ defined speed range

Outside ODD:
✗ unsupported city
✗ severe weather
✗ unsupported roadway
```

According to NHTSA, Level 4 can operate without a human driver, but only within **limited service areas or operating conditions**.[^nhtsa-levels]

```text
                    Level 4

                ┌────────────┐
                │    ODD     │
                │            │
                │    🚗      │
                │            │
                └────────────┘

Inside ODD:
ADS performs the complete driving task.

Outside ODD:
The feature is not designed to operate.
```

---

## Level 5 — Full Driving Automation

**Level 5** is the highest level in SAE J3016.

The system performs the entire driving task:

- no human driver is required;
- no takeover is required;
- operation is not restricted to a limited service area or a narrow set of driving conditions in the way Level 4 typically is;
- by definition, the system can operate under all roadway and environmental conditions that a human driver could manage.[^sae-summary][^nhtsa-levels]

```text
             Level 5

        Destination
             ↓
          Vehicle
             ↓
       Full automation
             ↓
        Any supported
   human-drivable roadway/
      environmental case
```

A vehicle designed entirely for Level 5 operation could, in principle, be designed without conventional driver controls such as a steering wheel or pedals.[^sae-chart]

---

# 4. Summary Table

| SAE Level | SAE Name | Steering / Acceleration / Braking | Who monitors the environment? | Human takeover | Operating scope |
|---|---|---|---|---|---|
| **0** | No Driving Automation | Human | Human | Human is always driving | No sustained driving automation |
| **1** | Driver Assistance | System assists **lateral OR longitudinal** control | Human | Human remains responsible | Feature-specific |
| **2** | Partial Driving Automation | System assists **lateral AND longitudinal** control | **Human** | Human must intervene when needed | Feature-specific |
| **3** | Conditional Driving Automation | **ADS performs complete DDT** | **ADS** | **Human must take over when requested** | Limited ODD |
| **4** | High Driving Automation | **ADS** | **ADS** | **No human fallback required within ODD** | Limited ODD |
| **5** | Full Driving Automation | **ADS** | **ADS** | Not required | All conditions a human driver could manage |

Sources for the taxonomy and the division of responsibility between the human driver and the automated system: SAE J3016 and the official SAE Levels of Driving Automation chart.[^sae-j3016][^sae-chart]

---

# 5. Driver Support vs Automated Driving

A simpler way to remember the SAE levels is:

```text
┌─────────────────────────────────────────────────────────┐
│                    DRIVER SUPPORT                       │
│                                                         │
│        Level 0       Level 1       Level 2              │
│                                                         │
│              HUMAN IS DRIVING                           │
│        HUMAN MONITORS THE ENVIRONMENT                   │
└───────────────────────────┬─────────────────────────────┘
                            │
                   Critical boundary
                         L2 → L3
                            │
┌───────────────────────────▼─────────────────────────────┐
│                 AUTOMATED DRIVING                       │
│                                                         │
│        Level 3       Level 4       Level 5              │
│                                                         │
│             SYSTEM PERFORMS THE DDT                     │
└─────────────────────────────────────────────────────────┘
```

NHTSA also uses a similar distinction:

- **Level 2 ADAS**: assistance for a human driver who must remain continuously attentive;
- **ADS**: terminology used for **SAE Levels 3–5**.[^nhtsa-sgo][^nhtsa-ads]

---

# 6. A Vehicle Does Not Necessarily Have One Fixed “Automation Level”

A common misunderstanding is to assign a single SAE level to an entire vehicle.

SAE J3016 defines the level according to the **driving automation feature that is engaged at a particular time**. A vehicle may contain multiple features with different automation levels.[^sae-j3016]

For example:

```text
Same vehicle
│
├── Manual driving                → Level 0
│
├── Adaptive Cruise Control       → Level 1 feature
│
├── Highway assist               → Level 2 feature
│
└── Conditional highway pilot    → Level 3 feature
```

Therefore, the statement:

> “This car is Level 3.”

is often less precise than:

> “This vehicle provides a Level 3 automated-driving feature under a defined ODD.”

---

# 7. SAE Level Is Not a Measure of AI Intelligence

SAE Levels primarily answer the question:

> **Who is responsible for performing the Dynamic Driving Task while the automation feature is engaged?**

They do **not directly measure**:

- model parameter count;
- perception accuracy;
- reasoning capability;
- VLM or VLA benchmark performance;
- the overall “intelligence” of the AI system.

SAE also notes that the taxonomy is **technical and descriptive**. It is not a required development sequence, nor does it automatically act as a legal certification of a product.[^sae-summary]

This distinction is extremely important when studying VLA:

```text
Better VLA benchmark
        ≠
Higher SAE automation level
```

A VLA model may be an important component inside a Level 2, Level 3, or Level 4 system. Merely using a VLA model is **not sufficient to claim a higher SAE automation level**.

---

# 8. Where Does VLA Fit in Autonomous Driving?

After understanding SAE automation levels, we can place VLA more accurately within an autonomous-driving system.

A traditional autonomous-driving pipeline is often described as:

```text
Sensors
   ↓
Perception
   ↓
Prediction
   ↓
Planning
   ↓
Control
   ↓
Vehicle
```

VLA research investigates whether a more unified policy can learn a mapping such as:

```text
Multi-camera / Sensors
        +
Vehicle state
        +
Navigation / Language
        ↓
┌──────────────────────┐
│        VLA           │
│                      │
│  Perception          │
│       ↓              │
│  Reasoning           │
│       ↓              │
│  Planning            │
└──────────┬───────────┘
           ↓
Future trajectory
 / waypoints / actions
           ↓
Controller / Safety Layer
           ↓
Vehicle
```

The central question for the next part of this overview is therefore:

> **How can vision and language be grounded into safe physical driving actions?**

---

# 9. Key Takeaways

1. **SAE J3016 defines six levels: Level 0 → Level 5.**
2. **Levels 0–2:** the human remains the driver and must monitor the driving environment.
3. **Levels 3–5:** the ADS performs the complete Dynamic Driving Task while the feature is engaged.
4. **Level 2 → Level 3** is a critical boundary because responsibility for performing and monitoring the complete DDT shifts from the human driver to the ADS.
5. **Level 3:** the human must be ready to take over when requested.
6. **Level 4:** no human fallback is required within the ODD, but operation can still be ODD-limited.
7. **Level 5:** full driving automation under all roadway and environmental conditions that a human driver could manage.
8. SAE Levels describe **driving-task responsibility**, not the strength or intelligence of an AI model.
9. A strong VLA model **does not automatically imply** a Level 4 or Level 5 autonomous vehicle.

---

# References

[^sae-j3016]: **SAE International.** *J3016_202104: Taxonomy and Definitions for Terms Related to Driving Automation Systems for On-Road Motor Vehicles.* Revised April 30, 2021. DOI: https://doi.org/10.4271/J3016_202104  
    Official SAE page: https://saemobilus.sae.org/standards/j3016_202104-taxonomy-definitions-terms-related-driving-automation-systems-road-motor-vehicles

[^sae-chart]: **SAE International.** *SAE J3016 Levels of Driving Automation — Visual Chart.* 2021. https://www.sae.org/binaries/content/assets/cm/content/blog/sae-j3016-visual-chart_5.3.21.pdf

[^sae-summary]: **SAE International.** *Automated Driving: Summary of SAE International's Levels of Driving Automation for On-Road Vehicles.* https://www.sae.org/binaries/content/assets/cm/content/news/press-releases/pathway-to-autonomy/automated_driving.pdf

[^nhtsa-levels]: **National Highway Traffic Safety Administration (NHTSA).** *Automated Vehicle Safety — The Road to Full Automation.* https://www.nhtsa.gov/vehicle-safety/automated-vehicle-safety

[^nhtsa-sgo]: **National Highway Traffic Safety Administration (NHTSA).** *Standing General Order on Crash Reporting — Understanding the Differences: ADS vs Level 2 ADAS.* https://www.nhtsa.gov/laws-regulations/standing-general-order-crash-reporting

[^nhtsa-ads]: **National Highway Traffic Safety Administration (NHTSA).** *Automated Driving Systems — Scope and Applicability.* https://www.nhtsa.gov/vehicle-manufacturers/automated-driving-systems

[^odd]: **Automated Vehicle Safety Consortium / SAE.** *Best Practice for Evaluation of Behavioral Competencies for Automated Driving System Dedicated Vehicles (ADS-DVs).* Includes SAE J3016 definitions of DDT, OEDR, and ODD. https://go.sae.org/rs/525-RCG-129/images/AVSC00008202111.pdf

---

## Next

**Part 2 — From Traditional Autonomous Driving to Vision-Language-Action (VLA)**

Planned topics:

- Modular autonomous-driving pipeline
- End-to-End Autonomous Driving
- Vision-Language Models for driving
- Vision-Language-Action models for driving
- Action representation: direct control vs waypoints vs trajectory tokens
- Evolution of VLA models for autonomous driving
