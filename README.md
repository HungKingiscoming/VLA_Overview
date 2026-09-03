# Vision-Language-Action (VLA) Models

> Tóm tắt nội dung tài liệu **“From Perception to Action: The Evolution of Vision-Language-Action Models”** của **Tien-Dat Nguyen** (VAL Lab / VinSpace).

---

## 1. Tổng quan

Tài liệu trình bày quá trình phát triển từ các hệ thống AI chỉ làm **perception** (nhận thức) sang các hệ thống có khả năng **hiểu đa phương thức, suy luận và trực tiếp tạo hành động vật lý**. Trọng tâm là **Vision-Language-Action (VLA)** và hai hướng ứng dụng chính:

- **Robot VLA (R-VLA)**: robot thao tác trong môi trường vật lý dựa trên hình ảnh, câu lệnh ngôn ngữ và trạng thái robot.
- **Autonomous-Driving VLA (AD-VLA)**: xe tự hành kết hợp quan sát đa camera, trạng thái xe, lệnh điều hướng và ngữ cảnh để dự đoán quỹ đạo hoặc điều khiển xe.

Thông điệp xuyên suốt của tài liệu là sự chuyển dịch từ pipeline robot truyền thống gồm nhiều module độc lập sang một policy đa phương thức có thể ánh xạ trực tiếp từ **quan sát + ngôn ngữ + trạng thái** sang **hành động**.

---

## 2. Từ hệ thống modular truyền thống đến VLA

### 2.1. Pipeline robot truyền thống

Một pipeline robotics truyền thống thường có chuỗi xử lý:

```text
Video Camera / Sensors
        ↓
    Perception
        ↓
 State Estimation
        ↓
Task & Motion Planning
        ↓
      Control
        ↓
       Robot
```

Cách thiết kế này chia hệ thống thành các thành phần chuyên biệt. Mỗi module xử lý một nhiệm vụ riêng, sau đó truyền kết quả cho module tiếp theo.

### 2.2. Khó khăn của modular pipeline

Tài liệu minh họa autonomous driving như một trường hợp mà pipeline modular trở nên khó quản lý. Trong môi trường thực tế, perception, hiểu ngữ cảnh, ra quyết định, planning và control phụ thuộc mạnh vào nhau; lỗi ở một module có thể lan truyền sang toàn bộ pipeline.

Điều này tạo động lực cho các mô hình end-to-end và các mô hình đa phương thức tích hợp perception, reasoning và action trong cùng một kiến trúc.

---

## 3. Sự tiến hóa của Machine Intelligence

Tài liệu mô tả một tiến trình phát triển tổng quát:

```text
Traditional ML
   ↓
Deep Learning
   ↓
Generative AI
   ↓
AI Agents
   ↓
Agentic AI / Embodied Intelligence
```

Điểm thay đổi quan trọng là AI chuyển từ:

- nhận biết / phân loại,
- sang sinh nội dung,
- sang hiểu ngữ cảnh đa phương thức,
- rồi cuối cùng có khả năng **lập kế hoạch và hành động trong thế giới thực**.

---

## 4. Vision-Language Model (VLM)

### 4.1. Ý tưởng

VLM kết hợp hai nguồn thông tin chính:

- **Vision**: ảnh hoặc video.
- **Language**: câu hỏi, mô tả hoặc instruction.

Mục tiêu là tạo biểu diễn chung giữa hình ảnh và ngôn ngữ để mô hình có thể mô tả ảnh, trả lời câu hỏi về ảnh hoặc thực hiện reasoning đa phương thức.

### 4.2. Cách xử lý đa phương thức

Sơ đồ trong tài liệu minh họa việc:

- encode ảnh bằng image encoder,
- encode text/token bằng language encoder hoặc word embedding,
- đưa các token vision và language vào một bộ xử lý chung,
- học alignment giữa hai modality thông qua các nhiệm vụ như contrastive matching hoặc masked prediction.

VLM là nền tảng quan trọng để tiến tới VLA vì nó cung cấp khả năng **nhìn + hiểu ngôn ngữ** trước khi thêm khả năng sinh **action**.

---

## 5. Vision-Language-Action là gì?

### 5.1. Định nghĩa

Một VLA model ánh xạ:

- quan sát thị giác,
- instruction bằng ngôn ngữ,
- trạng thái của hệ thống,

thành một hành động vật lý.

Công thức trong tài liệu:

```text
a_t = π_θ(o_t, ℓ, s_t)
```

Trong đó:

- `o_t`: visual observation tại thời điểm `t`.
- `ℓ`: language instruction.
- `s_t`: system state.
- `π_θ`: VLA policy có tham số `θ`.
- `a_t`: physical action được sinh ra.

### 5.2. Ví dụ

#### Robot manipulation

```text
Observation:
- A red cup is on the table

Instruction:
- "Pick up the red cup"

State:
- Arm position
- Gripper state

Action:
- Move the arm
- Close the gripper
```

#### Autonomous driving

```text
Observation:
- A pedestrian is crossing the road

Instruction:
- "Follow the planned route"

State:
- Vehicle speed
- Vehicle position

Action:
- Slow down
- Stop
```

VLA vì vậy có thể xem là bước mở rộng từ VLM:

```text
VLM: Vision + Language → Understanding / Text
VLA: Vision + Language + State → Physical Action
```

---

## 6. Bên trong một VLA Model

Tài liệu sử dụng **Qwen-VLA / QwenVL-based architecture** để minh họa pipeline tổng quát.

Các thành phần chính gồm:

1. **Visual observations**
   - một ảnh,
   - hoặc chuỗi frame/video.

2. **Task instruction**
   - ví dụ: `"Pick up the red cup"`.

3. **Embodiment-aware prompt / system state**
   - thông tin về robot,
   - trạng thái embodiment,
   - proprioception,
   - các biến điều khiển liên quan.

4. **Vision-Language backbone**
   - biểu diễn các modality thành token,
   - trộn các vision token, text token và proprioceptive/system token trong một chuỗi multimodal interleaved.

5. **Action Expert / Action Head**
   - chuyển hidden representation của VLM thành action representation.

6. **Action generation**
   - tạo action liên tục hoặc action trajectory/chunk.

---

## 7. Bên trong QwenVL Decoder

Sơ đồ trong tài liệu cho thấy quy trình xử lý nhiều loại input trong cùng một token stream:

```text
Visual frames/images ─┐
Task instruction ─────┼─→ Interleaved multimodal token sequence
Robot/system state ───┘
                            ↓
                       QwenVL Decoder
                            ↓
                   Multimodal hidden states
                            ↓
                  Action / output modules
```

Ý tưởng chính là tất cả modality được đưa về một biểu diễn thống nhất để transformer decoder có thể học quan hệ giữa quan sát, instruction và trạng thái embodiment.

---

## 8. Format dữ liệu cho VLA

Một sample VLA điển hình có ba nhóm thông tin:

### 8.1. Visual observation

- image hoặc video frame,
- có thể gồm nhiều camera hoặc nhiều thời điểm.

### 8.2. Task instruction

Ví dụ:

```text
"Pick up the red cup"
```

Instruction được tokenize thành text tokens.

### 8.3. Embodiment-aware / system prompt

Có thể chứa:

- robot state,
- end-effector pose,
- joint information,
- gripper state,
- embodiment description,
- proprioceptive data.

Ba nguồn dữ liệu được convert về token rồi ghép thành một **interleaved multimodal input sequence** cho model.

---

## 9. Đánh giá VLA

Tài liệu chia evaluation thành bốn nhóm chính.

| Nhóm đánh giá | Metric thường gặp | Ý nghĩa |
|---|---|---|
| Task completion | Success Rate (SR) | Robot/xe có hoàn thành task hay không |
| Generalization | OOD Success Rate | Có làm được khi đổi object, background, vị trí hoặc instruction hay không |
| Long-horizon behavior | Avg. sequence length, multi-step success | Có thực hiện liên tục nhiều sub-task hay không |
| Safety / efficiency | Collision rate, route completion, comfort, latency | Đặc biệt quan trọng với autonomous driving |

Điểm đáng chú ý là VLA không chỉ được đánh giá bằng accuracy như các model perception mà phải đánh giá **khả năng hoàn thành nhiệm vụ thực tế**.

---

# Part I - Robot VLA (R-VLA)

## 10. R-VLA trong thực tế

Các ví dụ demo trong tài liệu cho thấy cùng một model có thể thành công hoặc thất bại tùy task.

Một số failure mode được minh họa:

- mất cân bằng khi thực hiện động tác,
- vô tình đá vào hộp,
- dừng quá muộn,
- thực hiện đúng một số task nhưng thất bại ở task khác.

Điều này nhấn mạnh rằng VLA vẫn phải đối mặt với vấn đề robustness, control precision và long-horizon execution.

---

## 11. Từ task đến chuyển động robot

Ví dụ instruction:

```text
"Put the apple into the bowl."
```

Một task có thể được chia thành chuỗi hành động:

```text
1. Reach
2. Align
3. Grasp
4. Lift
5. Move
6. Place
7. Release
```

### 11.1. Action space của robot

Tài liệu trình bày ba cách điều khiển phổ biến.

#### A. End-effector control

Điều khiển trực tiếp pose của end-effector, ví dụ:

```text
Δx, Δy, Δz,
Δroll, Δpitch, Δyaw,
gripper
```

#### B. Joint control

Dự đoán hoặc điều khiển trực tiếp các joint command:

```text
q = [q1, q2, ..., qn]
```

#### C. Mobile manipulation

Kết hợp:

- arm action,
- gripper,
- base velocity.

---

## 12. Thu thập dữ liệu Robot VLA

Tài liệu nêu bốn phương pháp chính.

### 12.1. Teleoperation

```text
Human → controller → robot
```

Ví dụ thiết bị:

- VR controller,
- joystick,
- leader arm.

Ưu điểm: người điều khiển trực tiếp robot và tạo demonstration có chất lượng.

### 12.2. Kinesthetic Teaching

Người trực tiếp cầm hoặc dẫn robot thực hiện động tác.

Đặc điểm:

- trực quan,
- robot ghi lại trajectory,
- phù hợp với các task manipulation.

### 12.3. Autonomous Rollouts

Policy hiện tại tự chạy và hệ thống ghi trajectory.

Sau đó có thể:

- lọc trajectory,
- relabel,
- sử dụng lại làm training data.

### 12.4. Simulation

Dùng simulator để tạo demonstration tổng hợp.

Ưu điểm:

- dữ liệu quy mô lớn,
- an toàn,
- chi phí thấp,
- dễ tạo nhiều kịch bản.

Pipeline dữ liệu tổng quát:

```text
Human / Simulator
      ↓
Observation + State + Action
      ↓
Robot Demonstration Dataset
```

---

## 13. Dataset tiêu biểu cho Robot VLA

Tài liệu đề cập ba dataset lớn:

- **Open X-Embodiment**
- **DROID**
- **Bridge Data**

Open X-Embodiment được minh họa như một tập hợp dữ liệu lớn từ nhiều embodiment, nhiều task và nhiều dataset robot khác nhau, giúp huấn luyện model có khả năng transfer giữa các robot/platform khác nhau.

---

## 14. Tiến hóa của Robot VLA Models

Timeline trong tài liệu:

### 2022 - RT-1

- large-scale robot policy,
- bước đầu đưa transformer-scale model vào robot manipulation.

### 2023 - RT-2

- kết hợp **VLM knowledge + robot actions**,
- tận dụng knowledge từ vision-language model để cải thiện robot reasoning và generalization.

### 2024 - OpenVLA

- open-source VLA,
- giúp cộng đồng tiếp cận mô hình robot VLA dễ dàng hơn.

### 2024 - π0

- tập trung vào **continuous action generation**.

### 2025 - π0.5

- mạnh hơn về generalization và data training.

### 2025 - GR00T N1 / Gemini Robotics

- hướng tới foundation model cho robot ở quy mô lớn,
- tích hợp reasoning và action cho embodied intelligence.

### 2026 - Qwen-VLA

- sử dụng multimodal backbone kết hợp **flow-based action expert**.

Timeline này cho thấy R-VLA phát triển từ action-token policy sang các mô hình foundation lớn và action generator liên tục.

---

## 15. Cách R-VLA biểu diễn Action

Tài liệu tổng hợp bốn hướng chính.

### 15.1. Action Tokens

```text
Continuous Action
      ↓
Discretization
      ↓
<Action_127>, <Action_...>
```

Action liên tục được lượng tử hóa thành token để language model có thể sinh action giống như sinh text token.

**Ví dụ:** RT-2, OpenVLA.

Ưu điểm:

- tích hợp dễ với autoregressive LLM/VLM.

Hạn chế:

- discretization làm mất độ mịn của continuous control.

### 15.2. Direct Regression

```text
Features
   ↓
MLP Head
   ↓
[x, y, z, ...]
```

Model dự đoán trực tiếp action liên tục.

Ưu điểm:

- đơn giản,
- trực tiếp.

Hạn chế:

- khó mô hình hóa phân phối action đa mode.

### 15.3. Diffusion Policy

```text
Noise
  ↓
Denoising
  ↓
Action Chunk
```

Model sinh action trajectory bằng quá trình diffusion/denoising.

Ưu điểm:

- mô hình hóa được multimodal trajectories,
- tạo action mượt.

### 15.4. Flow Matching

```text
Noisy Action
     ↓
Learned Flow
     ↓
Action Chunk
```

Ưu điểm được nêu trong tài liệu:

- continuous action generation,
- efficient trajectory modeling.

Các ví dụ được minh họa gồm π-family và Qwen-VLA-style action expert.

---

# Part II - Autonomous Driving VLA (AD-VLA)

## 16. AD-VLA ở mức tổng quan

AD-VLA mở rộng mô hình vision-language-action sang bài toán xe tự hành.

Thay vì chỉ phát hiện lane, vehicle hoặc pedestrian rồi đưa cho planner riêng, Driving VLA có thể tiếp nhận nhiều nguồn input và trực tiếp sinh trajectory/action.

Các slide demo cho thấy model sử dụng nhiều camera xung quanh xe và dự đoán trajectory/decision như:

- turn right,
- keep forward.

---

## 17. AD-VLA quan sát gì và dự đoán gì?

### 17.1. Input

#### A. Multi-camera images / video

Ví dụ:

- front,
- left,
- right,
- rear.

#### B. Ego vehicle state

Có thể gồm:

- speed,
- pose `(x, y, z, roll, pitch, yaw)`,
- heading.

#### C. Previous trajectory

Quỹ đạo trước đó của xe.

#### D. Route / navigation command

Ví dụ:

```text
"Turn right at the next intersection"
```

#### E. Optional LiDAR / map information

Có thể bổ sung:

- LiDAR point cloud,
- HD map / lane information.

### 17.2. Bên trong Driving VLA

Model thực hiện đồng thời hoặc liên kết ba khả năng:

```text
Perception
Reasoning
Planning
```

### 17.3. Output

Hai nhóm output chính:

1. **Future trajectory / waypoints**
2. **Low-level control**
   - steering,
   - acceleration,
   - brake.

---

## 18. Thu thập dữ liệu cho AD-VLA

Tài liệu chia thành bốn nguồn chính.

### 18.1. Real Driving Logs

Thu thập từ xe thật:

- camera,
- LiDAR / radar,
- ego state,
- human/autonomous driving trajectory.

### 18.2. Human Driving

Sử dụng:

- expert trajectories,
- driver decisions.

Đây là dạng driving demonstration tương tự imitation learning.

### 18.3. Language Annotation

Bổ sung annotation dạng ngôn ngữ như:

- scene description,
- driving reasoning,
- navigation instruction.

Ví dụ:

```text
"Heavy cross traffic from the left"
"Slow down and yield to pedestrians"
"Turn right onto Oak St."
```

Language annotation là thành phần quan trọng giúp xe không chỉ bắt chước trajectory mà còn học được mô tả và reasoning về tình huống giao thông.

### 18.4. Simulation

Simulator được dùng để tạo:

- rare scenarios,
- safety-critical cases,
- tình huống khó thu thập ngoài đời thật.

---

## 19. Dataset cho AD-VLA

Slide dataset nhấn mạnh ba đặc điểm quan trọng của dữ liệu Driving VLA hiện đại:

### 19.1. Long-tail driving scenarios

Dataset cần chứa nhiều tình huống hiếm hoặc khó như:

- traffic control / signage anomaly,
- work zone / roadwork,
- vulnerable road users,
- animals on road,
- emergency vehicles,
- stationary vehicle obstruction.

### 19.2. Comprehensive reasoning annotations

Không chỉ có trajectory mà còn có annotation reasoning, ví dụ:

- spatial reasoning,
- 2D/3D object understanding,
- map/topology understanding,
- causal driving reasoning,
- counterfactual reasoning.

### 19.3. Large-scale data

Tài liệu minh họa rằng quy mô dữ liệu lớn và reasoning annotation đầy đủ có thể cải thiện cả:

- reasoning score,
- planning score.

---

## 20. Tiến hóa của AD-VLA Models

Timeline trong tài liệu:

### 2023-2024 - DriveLM

- bridge toward Driving VLA,
- structured perception-prediction-planning reasoning.

### 2024 - EMMA

- multimodal reasoning,
- kết hợp camera + navigation + ego state để sinh planner trajectory.

### 2025 - SimLingo

- closed-loop language-action alignment,
- liên kết driving + VLM understanding + aligned actions.

### 2025 - OpenDriveVLA

- end-to-end trajectory generation,
- 2D/3D perception + ego state + driver command → trajectory.

### 2025 - AutoVLA

- unified reasoning + action tokens,
- trajectory tokenization kết hợp RL fine-tuning.

### 2025-2026 - Alpamayo-R1

- reasoning-oriented planning,
- trajectory planning cho các tình huống driving phức tạp.

Nhìn tổng thể, AD-VLA tiến hóa theo hướng:

```text
Reasoning & Understanding
        ↓
Closed-loop Alignment
        ↓
Action & Trajectory Generation
        ↓
End-to-End Driving VLA
```

---

## 21. Cách AD-VLA biểu diễn Action

Tài liệu nêu hai hướng chính.

### 21.1. Direct Control

```text
Features
   ↓
Control Head
   ↓
Steering / Throttle / Brake
```

Đây là low-level control trực tiếp.

**Ưu điểm:**

- direct,
- simple.

**Hạn chế:**

- khó verify,
- khó constrain,
- tính interpretability thấp hơn trajectory-level planning.

### 21.2. Waypoint / Trajectory Prediction

```text
Multimodal Features
        ↓
Trajectory Decoder
        ↓
Future Waypoints / Trajectory
```

Ví dụ output:

```text
(x1, y1)
(x2, y2)
...
(xn, yn)
```

**Ưu điểm:**

- dễ diễn giải,
- dễ kết hợp với controller,
- thuận lợi hơn cho việc kiểm tra và áp constraint.

Theo slide, đây là cách biểu diễn phổ biến hơn trong Driving VLA.

---

# 22. So sánh Robot VLA và Autonomous-Driving VLA

| Thành phần | Robot VLA | Autonomous-Driving VLA |
|---|---|---|
| Vision input | RGB/video robot camera | Multi-camera, đôi khi LiDAR/map |
| Language | Manipulation instruction | Navigation + reasoning instruction |
| State | Joint, end-effector, gripper | Speed, pose, heading, previous trajectory |
| Action | End-effector / joint / gripper / base | Steering-throttle-brake hoặc trajectory |
| Data source | Teleoperation, kinesthetic teaching, rollout, simulation | Driving logs, human driving, language annotation, simulation |
| Common output | Action token / continuous action chunk | Waypoint / trajectory / control command |
| Evaluation | Task success, OOD generalization, long-horizon | Route completion, collision, comfort, latency, planning quality |

---

# 23. Các ý chính cần nhớ

1. **VLA là bước phát triển tiếp theo của VLM.** VLM hiểu vision + language; VLA thêm khả năng biến hiểu biết đó thành hành động vật lý.

2. **VLA không chỉ nhận ảnh và text.** State/embodiment information rất quan trọng vì cùng một instruction nhưng robot/vehicle ở trạng thái khác nhau phải tạo action khác nhau.

3. **Dữ liệu là thành phần cốt lõi.** Robot VLA dựa nhiều vào demonstration; Driving VLA cần real-world logs, expert driving, language annotation và simulation.

4. **Action representation là quyết định kiến trúc quan trọng.** Robot VLA có thể dùng action token, direct regression, diffusion hoặc flow matching; Driving VLA thường dùng direct control hoặc trajectory/waypoint prediction.

5. **Generalization quan trọng hơn accuracy đơn thuần.** Model cần hoạt động được khi môi trường, object, background, vị trí hoặc instruction thay đổi.

6. **Long-horizon behavior là bài toán khó.** Thành công ở một action đơn lẻ không đồng nghĩa với thành công trong một chuỗi nhiều bước.

7. **Safety đặc biệt quan trọng trong autonomous driving.** Collision rate, route completion, comfort và latency là các metric không thể bỏ qua.

8. **Xu hướng mô hình đang chuyển về continuous action generation.** Diffusion và flow matching giúp mô hình hóa action trajectory liên tục, đa mode và mượt hơn action token đơn giản.

9. **AD-VLA đang chuyển từ modular reasoning sang end-to-end reasoning + planning + action.** Các model mới hướng đến xử lý trực tiếp từ multimodal observations tới future trajectory.

10. **Mục tiêu dài hạn là embodied intelligence.** AI không chỉ trả lời câu hỏi mà có thể quan sát, hiểu, suy luận, lập kế hoạch và thực hiện hành động trong môi trường thực tế.

---

# 24. Pipeline VLA tổng quát

```text
          ┌───────────────────────┐
          │ Visual Observation    │
          │ Images / Video        │
          └──────────┬────────────┘
                     │
          ┌──────────▼────────────┐
          │ Vision Encoder / VLM  │
          └──────────┬────────────┘
                     │
Language Instruction ┼───────────────┐
                     │               │
System / Robot State ┼───────────────┤
                     │               │
             ┌───────▼───────────────▼─────┐
             │ Multimodal Representation    │
             │ + Reasoning / Planning       │
             └──────────────┬───────────────┘
                            │
                  ┌─────────▼─────────┐
                  │ Action Head /     │
                  │ Action Expert     │
                  └─────────┬─────────┘
                            │
               ┌────────────▼────────────┐
               │ Physical Action         │
               │ Robot / Vehicle Motion  │
               └─────────────────────────┘
```

---

# 25. Kết luận

Tài liệu mô tả một quá trình tiến hóa rõ ràng của AI:

```text
Perception
    ↓
Vision-Language Understanding
    ↓
Reasoning & Planning
    ↓
Vision-Language-Action
    ↓
Embodied / Agentic Intelligence
```

VLA là cầu nối giữa **AI hiểu thế giới** và **AI có khả năng tác động lên thế giới**. Hai lĩnh vực robot manipulation và autonomous driving đang hội tụ quanh cùng một ý tưởng: sử dụng foundation multimodal model để kết hợp perception, language understanding, system state, reasoning và action generation trong một framework thống nhất.

---

## Tài liệu nguồn

- *From Perception to Action: The Evolution of Vision-Language-Action Models* - Tien-Dat Nguyen, Vision and Learning Laboratory (VAL Lab), VinSpace.
