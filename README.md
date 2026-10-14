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
