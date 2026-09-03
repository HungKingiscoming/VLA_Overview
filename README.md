# Vision-Language-Action Models for Autonomous Driving

> **Personal overview / study notes** về cách Vision-Language-Action (VLA) được nghiên cứu và ứng dụng trong autonomous driving.  
> Phần đầu tiên của tài liệu thiết lập nền tảng về **các mức độ tự động hóa lái xe theo SAE J3016** trước khi đi vào VLA.

---

# 1. Levels of Driving Automation

## 1.1. Vì sao cần phân biệt các mức độ tự lái?

Cụm từ **“self-driving car”**, **“autonomous vehicle”** hay **“xe tự lái”** thường được sử dụng khá rộng trong truyền thông. Tuy nhiên, trong nghiên cứu và công nghiệp, mức độ tự động hóa thường được mô tả theo chuẩn **SAE J3016 – Taxonomy and Definitions for Terms Related to Driving Automation Systems for On-Road Motor Vehicles**.[^sae-j3016]

SAE chia driving automation thành **6 mức, từ Level 0 đến Level 5**:

```text
Level 0 ── Level 1 ── Level 2 ── Level 3 ── Level 4 ── Level 5
   │          │          │          │          │          │
   └──────── Driver Support ────────┘          │          │
                         └──── Automated Driving ──────────┘
```

Theo biểu đồ chính thức của SAE, **Levels 0–2** thuộc nhóm mà con người vẫn là người lái và phải giám sát hệ thống; **Levels 3–5** là các mức mà Automated Driving System (ADS) thực hiện toàn bộ **Dynamic Driving Task (DDT)** khi tính năng tự động hóa được kích hoạt.[^sae-chart]

> **Điểm chuyển quan trọng nhất không phải Level 4 → Level 5, mà là Level 2 → Level 3.**  
> Ở Level 2, người lái vẫn chịu trách nhiệm giám sát môi trường. Ở Level 3, hệ thống tự động thực hiện toàn bộ DDT trong phạm vi hoạt động được thiết kế, nhưng người lái phải sẵn sàng tiếp quản khi hệ thống yêu cầu.[^sae-summary][^nhtsa-levels]

---

## 1.2. Ba khái niệm cần biết trước

### Dynamic Driving Task — DDT

**Dynamic Driving Task (DDT)** là các nhiệm vụ vận hành và chiến thuật cần thực hiện theo thời gian thực để điều khiển xe trên đường. Theo SAE, nó bao gồm các chức năng như:

- điều khiển chuyển động ngang — **lateral control / steering**;
- điều khiển chuyển động dọc — **acceleration và deceleration**;
- quan sát môi trường;
- phát hiện và phản ứng với object/event;
- lập kế hoạch maneuver;
- signaling và các hành vi vận hành liên quan.

DDT **không bao gồm** các nhiệm vụ chiến lược cấp cao như lựa chọn điểm đến hoặc lập lịch chuyến đi.[^sae-summary]

Có thể hình dung:

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

**OEDR** là phần của DDT chịu trách nhiệm:

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

Ví dụ:

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

Khái niệm này đặc biệt quan trọng với VLA vì nhiều VLA driving models đang cố gắng kết hợp:

```text
Perception → Reasoning → Planning → Action
```

trong một mô hình đa phương thức thống nhất.

---

### Operational Design Domain — ODD

**Operational Design Domain (ODD)** mô tả **những điều kiện mà một hệ thống driving automation được thiết kế để hoạt động**. Các giới hạn có thể liên quan tới:

- khu vực địa lý;
- loại đường;
- tốc độ;
- thời gian trong ngày;
- thời tiết;
- điều kiện ánh sáng;
- tình trạng giao thông;
- đặc điểm cơ sở hạ tầng.[^odd]

Ví dụ một hệ thống có thể có ODD:

```text
Highway only
+ daytime
+ clear weather
+ mapped roads
+ speed ≤ 100 km/h
```

Một robotaxi khác có thể có:

```text
Geofenced urban area
+ selected streets
+ speed ≤ 50 km/h
+ no severe weather
```

**Level 4 vẫn có thể bị giới hạn bởi ODD.**  
**Level 5** được định nghĩa là full driving automation có thể thực hiện nhiệm vụ lái trong mọi điều kiện đường và môi trường mà một người lái có thể xử lý; vì vậy Level 5 không còn bị giới hạn bởi một ODD cụ thể theo cách Level 3/4 thường bị giới hạn.[^nhtsa-levels][^odd]

---

# 2. SAE Levels 0–5

## Level 0 — No Driving Automation

Ở **Level 0**, con người thực hiện toàn bộ nhiệm vụ lái xe.

Hệ thống có thể cung cấp:

- cảnh báo;
- hỗ trợ trong thời gian ngắn;
- emergency intervention.

Ví dụ mà NHTSA dùng để giải thích Level 0 gồm:

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

### Lưu ý

Trong SAE J3016, các hệ thống active safety chỉ can thiệp **tạm thời**, chẳng hạn Automatic Emergency Braking, không được xem là driving automation theo nghĩa thực hiện DDT một cách **sustained**.[^sae-j3016]

---

## Level 1 — Driver Assistance

Ở **Level 1**, hệ thống có thể hỗ trợ liên tục:

```text
Steering
   OR
Acceleration / Braking
```

nhưng **không đồng thời đảm nhiệm cả hai** như một Level 2 feature.[^sae-chart]

Ví dụ:

- Adaptive Cruise Control;
- Lane Centering / Lane Keeping assistance.

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

Người lái vẫn chịu trách nhiệm cho nhiệm vụ lái và phải liên tục giám sát hệ thống.[^nhtsa-levels]

---

## Level 2 — Partial Driving Automation

Ở **Level 2**, hệ thống có thể thực hiện **đồng thời**:

```text
Steering
   +
Acceleration / Braking
```

Ví dụ điển hình là sự kết hợp:

```text
Lane Centering
      +
Adaptive Cruise Control
```

Tuy nhiên:

> **Level 2 không có nghĩa là xe có thể tự lái mà không cần người giám sát.**

Người lái vẫn phải:

- quan sát đường;
- giám sát hệ thống;
- phát hiện tình huống nguy hiểm;
- sẵn sàng steering/braking khi cần.[^nhtsa-sgo]

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

NHTSA phân biệt rõ **Level 2 ADAS** với **ADS Levels 3–5**: Level 2 chỉ cung cấp partial driving automation cho một người lái đang chú ý, trong khi ADS hướng tới thực hiện toàn bộ DDT trong phạm vi hoạt động của hệ thống.[^nhtsa-sgo]

---

# 3. The Critical Transition: Level 2 → Level 3

Đây là ranh giới quan trọng nhất để hiểu autonomous driving.

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

SAE nhấn mạnh rằng khác biệt cốt lõi giữa Level 2 và Level 3 là **ai thực hiện toàn bộ Dynamic Driving Task**.[^sae-summary]

---

## Level 3 — Conditional Driving Automation

Ở **Level 3**, khi ADS hoạt động trong ODD của nó:

- hệ thống thực hiện toàn bộ DDT;
- hệ thống giám sát môi trường;
- con người không cần liên tục thực hiện DDT;
- **nhưng phải sẵn sàng tiếp quản khi hệ thống yêu cầu**.[^nhtsa-levels]

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

Do đó Level 3 tạo ra một vấn đề nghiên cứu đặc biệt quan trọng:

> **Takeover / handover:** người lái có thể phản ứng đủ nhanh và chính xác khi hệ thống yêu cầu tiếp quản hay không?

---

## Level 4 — High Driving Automation

Ở **Level 4**, hệ thống có thể thực hiện toàn bộ DDT **và không phụ thuộc vào người lái để xử lý fallback** trong ODD mà nó hỗ trợ.[^sae-chart]

Người ngồi trong xe có thể chỉ là passenger.

Tuy nhiên Level 4 **không có nghĩa là xe chạy được ở mọi nơi**.

Ví dụ về giới hạn ODD:

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

Theo NHTSA, Level 4 có thể vận hành mà không cần human driver nhưng chỉ trong **limited service areas / operating conditions**.[^nhtsa-levels]

```text
                    Level 4

                ┌────────────┐
                │    ODD     │
                │            │
                │    🚗      │
                │            │
                └────────────┘

Inside ODD:
ADS performs complete driving task.

Outside ODD:
Feature is not designed to operate.
```

---

## Level 5 — Full Driving Automation

**Level 5** là mức cao nhất trong SAE J3016.

Hệ thống thực hiện toàn bộ driving task:

- không cần người lái;
- không cần takeover;
- không bị giới hạn vào một service area hoặc một nhóm điều kiện lái cụ thể như Level 4;
- về định nghĩa, có thể vận hành trong mọi điều kiện roadway/environment mà một human driver có thể xử lý.[^sae-summary][^nhtsa-levels]

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

Steering wheel và pedals về nguyên tắc có thể không cần thiết đối với một vehicle được thiết kế hoàn toàn cho Level 5 operation.[^sae-chart]

---

# 4. Summary Table

| SAE Level | SAE name | Steering / Acceleration / Braking | Ai giám sát môi trường? | Human takeover | Operating scope |
|---|---|---|---|---|---|
| **0** | No Driving Automation | Human | Human | Human luôn lái | Không phải sustained driving automation |
| **1** | Driver Assistance | System hỗ trợ **lateral OR longitudinal** | Human | Human luôn chịu trách nhiệm | Feature-specific |
| **2** | Partial Driving Automation | System hỗ trợ **lateral AND longitudinal** | **Human** | Human phải can thiệp khi cần | Feature-specific |
| **3** | Conditional Driving Automation | **ADS thực hiện complete DDT** | **ADS** | **Human phải takeover khi được yêu cầu** | Limited ODD |
| **4** | High Driving Automation | **ADS** | **ADS** | **Không yêu cầu human fallback trong ODD** | Limited ODD |
| **5** | Full Driving Automation | **ADS** | **ADS** | Không cần | All conditions a human driver could manage |

Nguồn taxonomy và trách nhiệm giữa human/system: SAE J3016 và biểu đồ chính thức SAE Levels of Driving Automation.[^sae-j3016][^sae-chart]

---

# 5. Driver Support vs Automated Driving

Một cách đơn giản hơn để nhớ:

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

NHTSA hiện cũng sử dụng sự phân biệt:

- **Level 2 ADAS**: hỗ trợ một người lái vẫn phải liên tục chú ý;
- **ADS**: thuật ngữ dùng cho **SAE Levels 3–5**.[^nhtsa-sgo][^nhtsa-ads]

---

# 6. Một chiếc xe không nhất thiết có “một Level cố định”

Một điểm thường bị hiểu sai là gán cho cả chiếc xe một mức duy nhất.

SAE J3016 xác định level theo **driving automation feature đang được kích hoạt trong thời điểm cụ thể**. Một chiếc xe có thể chứa nhiều feature có mức automation khác nhau.[^sae-j3016]

Ví dụ khái niệm:

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

Do đó câu:

> “This car is Level 3”

thường kém chính xác hơn:

> “This vehicle provides a Level 3 automated-driving feature under a defined ODD.”

---

# 7. SAE Level không phải là thước đo độ thông minh của AI

SAE Levels trả lời chủ yếu câu hỏi:

> **Ai chịu trách nhiệm thực hiện Dynamic Driving Task khi feature được kích hoạt?**

Nó **không trực tiếp đo**:

- model có bao nhiêu parameters;
- perception accuracy;
- reasoning capability;
- VLM/VLA benchmark score;
- mức độ “thông minh” của AI.

SAE cũng lưu ý taxonomy này mang tính **technical/descriptive**, không phải một thứ tự bắt buộc về cách sản phẩm phải phát triển hoặc một chứng nhận pháp lý tự động.[^sae-summary]

Điều này cực kỳ quan trọng khi nghiên cứu VLA:

```text
Better VLA benchmark
        ≠
Higher SAE automation level
```

Một VLA có thể là một module mạnh trong hệ thống Level 2, Level 3 hoặc Level 4; chỉ riêng việc sử dụng VLA **không đủ để tuyên bố chiếc xe đạt một SAE Level cao hơn**.

---

# 8. VLA nằm ở đâu trong Autonomous Driving?

Sau khi hiểu SAE Levels, ta mới có thể đặt VLA đúng vị trí.

Pipeline autonomous driving truyền thống thường được mô tả:

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

VLA đang nghiên cứu khả năng học một policy thống nhất hơn:

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

Vì vậy, trong phần tiếp theo của overview, câu hỏi trung tâm sẽ là:

> **How can vision and language be grounded into safe physical driving actions?**

---

# 9. Takeaways

1. **SAE J3016 có 6 mức: Level 0 → Level 5.**
2. **Levels 0–2:** human vẫn là người lái và phải giám sát.
3. **Levels 3–5:** ADS thực hiện complete Dynamic Driving Task khi feature được kích hoạt.
4. **Level 2 → Level 3** là ranh giới rất quan trọng: responsibility for environment monitoring chuyển từ human sang ADS.
5. **Level 3:** human phải sẵn sàng takeover.
6. **Level 4:** không cần human takeover trong ODD, nhưng hệ thống vẫn bị giới hạn bởi ODD.
7. **Level 5:** full automation trong mọi điều kiện mà human driver có thể xử lý.
8. SAE Level mô tả **vai trò và trách nhiệm trong driving task**, không phải độ mạnh của AI.
9. Một VLA model tốt **không tự động đồng nghĩa** với một Level 4/5 autonomous vehicle.

---

# References

[^sae-j3016]: **SAE International.** *J3016_202104: Taxonomy and Definitions for Terms Related to Driving Automation Systems for On-Road Motor Vehicles.* Revised April 30, 2021. DOI: https://doi.org/10.4271/J3016_202104  
    Official SAE page: https://saemobilus.sae.org/standards/j3016_202104-taxonomy-definitions-terms-related-driving-automation-systems-road-motor-vehicles

[^sae-chart]: **SAE International.** *SAE J3016 Levels of Driving Automation — Visual Chart.* 2021. https://www.sae.org/binaries/content/assets/cm/content/blog/sae-j3016-visual-chart_5.3.21.pdf

[^sae-summary]: **SAE International.** *Automated Driving: Summary of SAE International's Levels of Driving Automation for On-Road Vehicles.* https://www.sae.org/binaries/content/assets/cm/content/news/press-releases/pathway-to-autonomy/automated_driving.pdf

[^nhtsa-levels]: **National Highway Traffic Safety Administration (NHTSA).** *Automated Vehicle Safety — The Road to Full Automation.* https://www.nhtsa.gov/vehicle-safety/automated-vehicle-safety

[^nhtsa-sgo]: **NHTSA.** *Standing General Order on Crash Reporting — Understanding the Differences: ADS vs Level 2 ADAS.* https://www.nhtsa.gov/laws-regulations/standing-general-order-crash-reporting

[^nhtsa-ads]: **NHTSA.** *Automated Driving Systems — Scope and Applicability.* https://www.nhtsa.gov/vehicle-manufacturers/automated-driving-systems

[^odd]: **Automated Vehicle Safety Consortium / SAE.** *Best Practice for Evaluation of Behavioral Competencies for Automated Driving System Dedicated Vehicles (ADS-DVs).* Includes SAE J3016 definitions of DDT, OEDR and ODD. https://go.sae.org/rs/525-RCG-129/images/AVSC00008202111.pdf

---

## Next

**Part 2 — From Traditional Autonomous Driving to Vision-Language-Action (VLA)**

Planned topics:

- Modular autonomous-driving pipeline
- End-to-End Autonomous Driving
- Vision-Language Models for driving
- VLA for driving
- Action representation: direct control vs waypoints vs trajectory tokens
- VLA4AD model evolution
