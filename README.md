# 🚁 UAVDepthLite-UQ: Lightweight Uncertainty-Aware Monocular Metric Depth Estimation for UAVs

> **Đề tài Nghiên cứu Khoa học** — Khoa Công nghệ Thông tin, Trường Đại học Thăng Long
>
> Nghiên cứu mô hình nhẹ **ước lượng độ sâu tuyệt đối từ ảnh đơn (Monocular Metric Depth Estimation)** có xét **độ bất định (Uncertainty)**, phục vụ **ước lượng khoảng cách đến đối tượng trên UAV** và triển khai trên thiết bị biên **Raspberry Pi 5**.

---

## 📋 Mục lục

- [Giới thiệu](#-giới-thiệu)
- [Mục tiêu nghiên cứu](#-mục-tiêu-nghiên-cứu)
- [Kiến trúc hệ thống](#-kiến-trúc-hệ-thống)
- [Các thành phần chính](#-các-thành-phần-chính)
- [Cấu trúc dự án](#-cấu-trúc-dự-án)
- [Yêu cầu môi trường](#️-yêu-cầu-môi-trường)
- [Cài đặt](#-cài-đặt)
- [Dataset](#-dataset)
- [Đánh giá](#-đánh-giá)
- [Teacher Model](#-teacher-model)
- [Student Model](#-student-model)
- [Uncertainty Estimation](#-uncertainty-estimation)
- [Object Ranging](#-object-ranging)
- [UAV Simulation](#-uav-simulation)
- [Raspberry Pi 5 Deployment](#-raspberry-pi-5-deployment)
- [Lộ trình thực nghiệm](#-lộ-trình-thực-nghiệm)
- [Kết quả](#-kết-quả)
- [Tác giả](#-tác-giả)

---

# 📖 Giới thiệu

Các nền tảng UAV ngày càng được sử dụng trong giám sát, khảo sát, nông nghiệp, kiểm tra công trình và nhiều bài toán thị giác máy tính. Tuy nhiên, việc xác định khoảng cách và cấu trúc không gian từ camera RGB đơn vẫn là một thách thức, đặc biệt khi hệ thống phải hoạt động trực tiếp trên thiết bị biên có tài nguyên hạn chế.

Dự án **UAVDepthLite-UQ** tập trung nghiên cứu một mô hình nhẹ có khả năng:

- Ước lượng **metric depth** từ một ảnh RGB đơn.
- Dự đoán **uncertainty** tương ứng với kết quả depth.
- Sử dụng uncertainty để tăng độ tin cậy của bài toán **object ranging**.
- Triển khai mô hình trên **Raspberry Pi 5**.
- Đánh giá đồng thời:
  - độ chính xác;
  - độ tin cậy;
  - tốc độ suy luận;
  - tài nguyên phần cứng;
  - khả năng triển khai trên UAV.

Dự án không tập trung nghiên cứu thuật toán điều khiển bay, SLAM hoặc thiết kế flight controller. UAV được sử dụng như một **nền tảng mang camera và thiết bị biên** phục vụ bài toán perception.

---

# 🎯 Mục tiêu nghiên cứu

Các mục tiêu chính của đề tài gồm:

1. Xây dựng mô hình nhẹ ước lượng **độ sâu tuyệt đối từ ảnh đơn** phù hợp với dữ liệu UAV.
2. Sử dụng **Knowledge Distillation** từ một mô hình Teacher mạnh để cải thiện Student Model.
3. Tích hợp cơ chế dự đoán **uncertainty** cho từng pixel.
4. Khai thác uncertainty trong bài toán **ước lượng khoảng cách đến đối tượng**.
5. Triển khai mô hình trên **Raspberry Pi 5** bằng ONNX Runtime.
6. Đánh giá sự đánh đổi giữa:

```text
Accuracy
   ↕
Reliability
   ↕
Inference Speed
   ↕
Resource Usage
```
---

# 🧠 Kiến trúc hệ thống

Giai đoạn huấn luyện:
                 Aerial RGB Image
                        │
                        ▼
              ┌──────────────────┐
              │  Teacher Model   │
              │ Depth Anything V2│
              │   Metric Depth   │
              └────────┬─────────┘
                       │
              Teacher Metric Depth
                       │
                       ▼
              Knowledge Distillation
                       │
                       ▼
             ┌─────────────────────┐
             │   UAVDepthLite-UQ   │
             │                     │
             │ MobileNetV3-Small   │
             │ Lightweight Decoder │
             └──────────┬──────────┘
                        │
                 ┌──────┴──────┐
                 ▼             ▼
             Metric Depth   Uncertainty

Giai đoạn suy luận:
                      RGB Camera
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
      Object Detector          UAVDepthLite-UQ
              │                       │
            BBox               Depth + Uncertainty
              │                       │
              └───────────┬───────────┘
                          ▼
                    ROI Processing
                          │
                          ▼
               Uncertainty Filtering
                          │
                          ▼
                    Object Ranging
                          │
                          ▼
              Distance + Reliability

# 🧩 Các thành phần chính
| Thành phần | Vai trò |
|---|---|
| **Depth Anything V2 Metric** | Teacher Model ban đầu |
| **MobileNetV3-Small** | Backbone của Student |
| **Lightweight Decoder** | Khôi phục bản đồ depth |
| **Depth Head** | Dự đoán metric depth |
| **Uncertainty Head** | Dự đoán độ bất định |
| **Knowledge Distillation** | Truyền tri thức Teacher → Student |
| **Object Detector** | Xác định vùng đối tượng |
| **Uncertainty-aware Ranging** | Ước lượng khoảng cách đáng tin cậy |
| **ONNX Runtime** | Inference trên Raspberry Pi 5 |
| **PX4 + Gazebo** | Mô phỏng UAV |
| **QGroundControl** | Ground Control Station |

#📁 Cấu trúc dự án
UAVDepthLite-UQ/
│
├── src/
│   │
│   ├── datasets/
│   │   └── aerialmetric.py
│   │
│   ├── teachers/
│   │   └── depth_anything_v2.py
│   │
│   ├── models/
│   │   ├── backbone.py
│   │   ├── decoder.py
│   │   ├── depth_head.py
│   │   ├── uncertainty_head.py
│   │   └── uavdepthlite_uq.py
│   │
│   ├── losses/
│   │   ├── depth_loss.py
│   │   ├── kd_loss.py
│   │   └── uncertainty_loss.py
│   │
│   ├── evaluation/
│   │   └── metrics.py
│   │
│   ├── ranging/
│   │   └── object_ranging.py
│   │
│   └── utils/
│
├── scripts/
│   ├── test_metric.py
│   ├── test_teacher.py
│   ├── benchmark_teacher.py
│   ├── train_student.py
│   ├── evaluate_student.py
│   ├── evaluate_uncertainty.py
│   ├── evaluate_ranging.py
│   ├── export_onnx.py
│   └── benchmark_onnx.py
│
├── configs/
│   ├── teacher.yaml
│   ├── student.yaml
│   └── deployment.yaml
│
├── simulation/
│   └── px4_gazebo/
│
├── deployment/
│   └── raspberry_pi/
│
├── outputs/
│
├── .gitignore
├── requirements.txt
└── README.md

#⚙️ Yêu cầu môi trường
Môi trường phát triển
| Thành phần | Khuyến nghị |
|---|---|
| OS | Debian / Ubuntu / Linux |
| Python | 3.10 – 3.12 |
| PyTorch | Phiên bản tương thích môi trường |
| NumPy | ≥ 1.24 |
| Pillow | ≥ 9.0 |
| OpenCV | ≥ 4.x |
| Transformers | Phiên bản hỗ trợ Depth Anything V2 |
| ONNX | Latest stable |
| ONNX Runtime | Latest stable |
| scikit-learn | Latest stable |
| matplotlib | Latest stable |

Phần cứng
Máy phát triển
Không yêu cầu GPU NVIDIA cho việc:
- viết code;
- kiểm tra metric;
- xử lý dữ liệu;
- test inference nhỏ;
- export ONNX.
Các tác vụ huấn luyện nặng được thực hiện trên:
- Google Colab;
- Kaggle Notebook;
- hoặc máy có NVIDIA GPU.
Edge Device
Raspberry Pi 5
+ RGB Camera
+ Active Cooling
+ 5V/5A Power Supply / UBEC

🚀 Cài đặt
1. Clone repository
git clone https://github.com/<username>/UAVDepthLite-UQ.git
cd UAVDepthLite-UQ

2. Tạo virtual environment
python3 -m venv .venv

Kích hoạt:
source .venv/bin/activate

Kiểm tra:
which python
python --version
pip --version

3. Cài dependencies cơ bản
pip install --upgrade pip
pip install numpy

Cài PyTorch CPU:
pip install torch --index-url https://download.pytorch.org/whl/cpu

Cài các thư viện bổ sung:
pip install pillow matplotlib pandas scikit-learn opencv-python

Cài thư viện cho Teacher Model:
pip install transformers safetensors huggingface_hub

📦 Dataset
Dự án sử dụng các nhóm dữ liệu chính sau:
Dataset	Vai trò
AerialMetric	Dataset chính cho Metric Depth
Mid-Air	Synthetic dataset phục vụ pretraining / bổ sung
WildUAV	External test / generalization
Own Ranging Dataset	Đánh giá Object Ranging


AerialMetric
AerialMetric được sử dụng làm dataset chính trong nghiên cứu.
Dataset làm việc được tổ chức:
dataset/
├── train/
├── val/
└── test/

Trong đó:
Train
→ Huấn luyện Student

Validation
→ Teacher benchmark
→ Model selection
→ Hyperparameter tuning

Test
→ Final evaluation

Dataset kích thước lớn được lưu trên Google Drive, không được push trực tiếp lên GitHub.
Ví dụ mount Google Drive trên Google Colab:
from google.colab import drivedrive.mount("/content/drive")


Sau đó thiết lập đường dẫn:
DATA_ROOT = (    "/content/drive/MyDrive/"    "UAVDepthLite/datasets/AerialMetric/dataset")


Kiểm tra:
import osprint(os.path.exists(DATA_ROOT))print(os.listdir(DATA_ROOT))


Kết quả mong đợi:
True
['train', 'val', 'test']

📊 Đánh giá
Các metric chính cho bài toán Metric Depth gồm:
Absolute Relative Error
AbsRel ↓

Công thức:
AbsRel = mean(|Pred - GT| / GT)

Càng thấp càng tốt.
Root Mean Squared Error
RMSE ↓

Công thức:
RMSE = sqrt(mean((Pred - GT)^2))

Càng thấp càng tốt.
Delta Accuracy
δ1 ↑

Điều kiện:
max(Pred / GT, GT / Pred) < 1.25

Càng cao càng tốt.
Test metrics
PYTHONPATH=. python scripts/test_metric.py

File metric:
src/evaluation/metrics.py

Các metric hiện tại:
AbsRel
RMSE
δ1

👨‍🏫 Teacher Model
Teacher ban đầu của dự án:
Depth Anything V2
Metric Outdoor Base

Teacher được sử dụng để:
- benchmark trên dữ liệu UAV;
- cung cấp supervision bổ sung;
- hỗ trợ Knowledge Distillation;
- tạo pseudo metric depth khi cần.
Pipeline:
AerialMetric RGB
       ↓
Teacher
       ↓
Metric Depth
       ↓
Ground Truth
       ↓
AbsRel / RMSE / δ1

Teacher được benchmark trên Validation Set, không sử dụng Test Set để lựa chọn model.
Ví dụ cấu trúc:
src/
└── teachers/
    └── depth_anything_v2.py

Script kiểm thử:
scripts/test_teacher.py

Script benchmark:
scripts/benchmark_teacher.py

🧑‍🎓 Student Model
Student dự kiến sử dụng:
MobileNetV3-Small
        ↓
Lightweight Multi-scale Decoder
        ↓
Depth Head
        +
Uncertainty Head

Quá trình phát triển:
Student Baseline
      ↓
Metric Depth Training
      ↓
Knowledge Distillation
      ↓
Uncertainty Estimation
      ↓
UAVDepthLite-UQ

Các thí nghiệm chính:
B0: Student Baseline

B1: Student + Knowledge Distillation

B2: Student + KD + Uncertainty

Mục tiêu của Student không chỉ là đạt độ chính xác tốt mà còn phải có kích thước và độ phức tạp phù hợp để triển khai trên Raspberry Pi 5.
🎯 Uncertainty Estimation
Ngoài metric depth:
D(x, y)

mô hình dự kiến sinh thêm:
U(x, y)

Trong đó:
- D(x, y) là metric depth tại pixel;
- U(x, y) biểu diễn mức độ bất định của dự đoán tại pixel tương ứng.
Mục tiêu mong muốn:
Depth Error tăng
        ↕
Uncertainty cũng tăng

Uncertainty được đánh giá bằng các metric:
Spearman Correlation
Sparsification
AUSE
Risk-Coverage

Uncertainty sau đó được sử dụng cho Object Ranging để loại bỏ hoặc giảm ảnh hưởng của các pixel có dự đoán kém tin cậy.
📏 Object Ranging
Object Detector được sử dụng để xác định vùng quan tâm:
RGB
 ↓
Detector
 ↓
Bounding Box / ROI

Cùng ảnh RGB được đưa qua UAVDepthLite-UQ:
RGB
 ↓
UAVDepthLite-UQ
 ↓
Metric Depth
+
Uncertainty

Sau đó:
BBox / ROI
    +
Depth
    +
Uncertainty
    ↓
ROI Processing
    ↓
Object Distance
+
Reliability

Các phương pháp dự kiến so sánh:
Center Pixel
Mean ROI
Median ROI
Inner ROI
Outlier Filtering
Uncertainty Hard Filtering
Uncertainty Weighted Ranging

Các metric đánh giá:
MAE
RMSE
Relative Error
P95 Error
Failure Rate

Dự án dự kiến xây dựng thêm một tập dữ liệu ranging thực tế sử dụng thiết bị đo khoảng cách để cung cấp Ground Truth.
🚁 UAV Simulation
Phần mô phỏng dự kiến sử dụng:
PX4 SITL
+
Gazebo
+
QGroundControl

Model UAV ban đầu:
X500 / Quadrotor tương đương

Các chức năng kiểm thử:
Arm
 ↓
Takeoff
 ↓
Hover
 ↓
Waypoint Mission
 ↓
Landing

Sau đó tích hợp:
RGB Camera
+
MAVLink / MAVSDK

để lấy các dữ liệu cần thiết như:
RGB Stream
Altitude
Attitude
GPS
Velocity
Flight State

UAV simulation được sử dụng để kiểm tra khả năng tích hợp hệ thống, không thay thế dữ liệu thực trong đánh giá độ chính xác của mô hình Metric Depth.
🖥️ Raspberry Pi 5 Deployment
Model cuối được triển khai theo pipeline:
PyTorch
   ↓
ONNX
   ↓
ONNX Runtime
   ↓
Raspberry Pi 5

Hai phiên bản dự kiến được benchmark:
FP32
vs
INT8

Chỉ số Edge Benchmark
Các chỉ số được đo gồm:
Mean Latency
P50 Latency
P95 Latency
FPS
RAM Usage
CPU Usage
Temperature
Model Size
Throttling
Sustained Performance

Hai loại latency được phân biệt:
Model-only latency
Input Tensor
     ↓
Model
     ↓
Output

End-to-End latency
Camera Capture
      ↓
Preprocessing
      ↓
Object Detector
      ↓
Depth + Uncertainty
      ↓
Object Ranging
      ↓
Result

Có thể biểu diễn:
T_E2E =
T_capture
+ T_preprocess
+ T_detector
+ T_depth
+ T_ranging

🛠️ Kiến trúc phần cứng dự kiến
                  UAV
                   │
                   ▼
            Flight Controller
                   │
                MAVLink
                   │
                   ▼
           Raspberry Pi 5
                   │
                   ▼
              RGB Camera
                   │
                   ▼
           UAVDepthLite-UQ
                   │
          ┌────────┴────────┐
          ▼                 ▼
      Metric Depth      Uncertainty
          │                 │
          └────────┬────────┘
                   ▼
            Object Ranging
                   │
                   ▼
         Distance + Reliability

Raspberry Pi 5 đóng vai trò companion computer, không trực tiếp điều khiển motor.
Nguồn cấp cho Raspberry Pi 5 cần được thiết kế riêng phù hợp với hệ thống UAV, không lấy trực tiếp từ cổng telemetry của flight controller.
🧪 Lộ trình thực nghiệm
Pipeline nghiên cứu tổng thể:
AerialMetric
    ↓
Dataset Pipeline
    ↓
Teacher Benchmark
    ↓
Student Baseline
    ↓
Knowledge Distillation
    ↓
Uncertainty Estimation
    ↓
Object Ranging
    ↓
ONNX Export
    ↓
Raspberry Pi 5
    ↓
PX4 / UAV Integration

Các ablation chính:
Thí nghiệm	So sánh
Student	Baseline vs KD
Uncertainty	Depth-only vs Depth + UQ
Ranging	Median ROI vs UQ Filtering / Weighting
Edge	FP32 vs INT8
Input	Các độ phân giải đầu vào khác nhau
Runtime	Model-only vs End-to-End


📈 Kết quả
⚠️ Dự án đang trong quá trình nghiên cứu.
Các kết quả sẽ được cập nhật sau khi hoàn thành từng giai đoạn thực nghiệm. Không sử dụng các giá trị giả hoặc chưa được đo thực tế.

Metric Depth
Model	AbsRel ↓	RMSE ↓	δ1 ↑	Params	Model Size
Teacher	TBD	TBD	TBD	TBD	TBD
Student Baseline	TBD	TBD	TBD	TBD	TBD
Student + KD	TBD	TBD	TBD	TBD	TBD
UAVDepthLite-UQ	TBD	TBD	TBD	TBD	TBD


Uncertainty
Model	Spearman ↑	AUSE ↓	Risk-Coverage	Ghi chú
UAVDepthLite-UQ	TBD	TBD	TBD	TBD


Object Ranging
Method	MAE ↓	RMSE ↓	Relative Error ↓	P95 ↓	Failure Rate ↓
Center Pixel	TBD	TBD	TBD	TBD	TBD
Mean ROI	TBD	TBD	TBD	TBD	TBD
Median ROI	TBD	TBD	TBD	TBD	TBD
Robust ROI	TBD	TBD	TBD	TBD	TBD
UQ Filtering	TBD	TBD	TBD	TBD	TBD
UQ Weighting	TBD	TBD	TBD	TBD	TBD


Raspberry Pi 5
Model	Precision	FPS ↑	Mean Latency ↓	P95 ↓	RAM	Temperature
UAVDepthLite-UQ	FP32	TBD	TBD	TBD	TBD	TBD
UAVDepthLite-UQ	INT8	TBD	TBD	TBD	TBD	TBD


📂 Quản lý dữ liệu
Repository GitHub chỉ lưu:
Source Code
Config
Scripts
Documentation

Không lưu:
Dataset
Checkpoints
Pseudo Labels
Large Outputs
Virtual Environment

Ví dụ .gitignore:
.venv/
__pycache__/
*.pyc

dataset/
datasets/

checkpoints/
outputs/

*.pth
*.pt
*.onnx

.vscode/
.idea/

Các file lớn được lưu trên Google Drive:
UAVDepthLite/
├── datasets/
│   ├── AerialMetric/
│   ├── MidAir/
│   └── WildUAV/
│
├── checkpoints/
│   ├── teacher/
│   └── student/
│
├── pseudo_depth/
│
├── results/
│
└── exports/
    └── onnx/

Workflow tổng thể:
Debian VMware
      │
      │ viết code
      ▼
    GitHub
      │
      │ clone / pull
      ▼
 Google Colab GPU ───────── Google Drive
      │                         │
      │                         ├── Dataset
      │                         ├── Checkpoint
      │                         ├── Pseudo Depth
      │                         └── Results
      │
      ▼
Teacher / Training / Evaluation
      │
      ▼
     ONNX
      │
      ▼
Raspberry Pi 5

👥 Tác giả
MSSV	Họ và Tên	GitHub
TBD	Thành viên 1	TBD
TBD	Thành viên 2	TBD
TBD	Thành viên 3	TBD
TBD	Thành viên 4	TBD
TBD	Thành viên 5	TBD


- Lĩnh vực: Khoa học Kỹ thuật và Công nghệ
- Đơn vị: Khoa Công nghệ Thông tin — Trường Đại học Thăng Long
- Giảng viên hướng dẫn: TBD
📌 Trạng thái dự án
- [x] Xây dựng môi trường phát triển
- [x] Xây dựng Depth Evaluation Metrics
- [ ] Hoàn thiện AerialMetric Data Pipeline
- [ ] Benchmark Teacher Model
- [ ] Xây dựng Student Baseline
- [ ] Knowledge Distillation
- [ ] Uncertainty Estimation
- [ ] Object Ranging
- [ ] PX4 + Gazebo Simulation
- [ ] Export ONNX
- [ ] Raspberry Pi 5 Deployment
- [ ] INT8 Quantization
- [ ] Final UAV Integration
📜 License
Dự án được phát triển cho mục đích nghiên cứu và học thuật.
Dataset, pretrained model và các thành phần bên thứ ba tuân theo license tương ứng của tác giả gốc.
📖 Citation
Thông tin citation sẽ được cập nhật khi nghiên cứu hoàn thành và được công bố.
