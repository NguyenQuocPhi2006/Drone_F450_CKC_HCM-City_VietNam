# Drone_F450_CKC_HCM-City_VietNam

ESP32 drone project with F450 frame in HCM City

## Video test drone

<p align="center">
  <a href="https://youtube.com/shorts/pp7vdn5QxUU?si=EoC83cz4GQpj6je0">
    <img width="360" src="https://img.youtube.com/vi/pp7vdn5QxUU/0.jpg" alt="Video 1"/>
  </a>
  <a href="https://youtube.com/shorts/e0xyz8K_Xz0?si=DAhz2X3Vl13OeQYI">
    <img width="360" src="https://img.youtube.com/vi/e0xyz8K_Xz0/0.jpg" alt="Video 2"/>
  </a>
</p>

---

## Giới thiệu
Tôi là người Việt Nam và đây là mã nguồn Drone trong dự án môn học của tôi. Tôi đã mất hơn 2 tháng cho phần lập trình Drone này, bắt đầu từ những thứ cơ bản nhất cho đến việc cân bằng máy bay tốt nhất với khả năng của mình.

## Chi tiết kỹ thuật
Mã nguồn này dùng **GY87** (MPU6050 và QMC5883P, không sử dụng BMP180) và kết hợp với **bộ lọc Mahony** chạy ở tần số 1Khz.  
Quy trình bao gồm:
- Đọc dữ liệu thô, hiệu chỉnh, đưa vào bộ lọc Mahony.  
- Xử lý góc nghiêng mong muốn (tay cầm hoặc GPS).  
- Nếu có GPS: sử dụng vòng lặp ngoài eSKF để tính vị trí X, Y trong không trung (dựa vào quaternion q0, q1, q2, q3).  
- Nếu có cảm biến áp suất: kết hợp dữ liệu áp suất với gyro + accel trục Z.  
- Qua PID để điều chỉnh **target_roll**, **target_pitch**, **target_altitude**.  
- Đầu ra cuối cùng là **Duty Cycle** của 4 kênh PWM.  

Tất cả giao tiếp qua **I2C, UART, SPI** đều chạy với DMA.  
Đọc dữ liệu từ RX của tay cầm FS-iAB6 bằng DMA + ngắt để lấy 32 byte dữ liệu.  

- **Vi điều khiển**: STM32F411CEU6  
- **Động lực**: ESC40A Brushless + động cơ 930KV 2212  
- **Cánh quạt**: 9 inch (ổn định hơn 10 inch do momen quán tính nhỏ hơn)

---

## Kinh nghiệm & lưu ý
- **Hiệu chỉnh ESC**: không làm bằng code, mà hiệu chỉnh qua tay cầm và debug bằng STM32CubeIDE v1.9.0.  
- **Tinh chỉnh PID**: không chỉ phụ thuộc vào KP/KI/KD, mà còn vào cấu hình cảm biến (filter mode, dãy đo accel/gyro) và thông số bộ lọc Mahony.  
- **Hiệu chỉnh cảm biến**:
  - **MPU6050**: hiệu chỉnh 3 trục gyro trên mặt phẳng lý tưởng, và 6 mặt của accel (±X, ±Y, ±Z).  
  - **QMC5883L (la bàn)**: mỗi vùng có từ trường khác nhau, cần hiệu chỉnh lại giống như la bàn trên điện thoại mới.

---

## Hình ảnh minh họa

  <img width="300" alt="Drone" src="https://github.com/user-attachments/assets/d9528ef6-04e1-4c48-8c56-7e9497522348" />
## Thiết kế PCB

  <img width="300" alt="PCB" src="https://github.com/user-attachments/assets/60229d58-c28d-4c60-84ab-a91336c02467" />
## Màng hình hiển thị thông số trên tay cầm FS_iB6A

  <img width="300" alt="Display" src="https://github.com/user-attachments/assets/6484176b-4d98-43fc-a8e5-21764e5d9472" />

---

## Kết luận
Dự án này là kết quả của quá trình học tập và thử nghiệm nghiêm túc. Tôi hy vọng mã nguồn và kinh nghiệm chia sẻ sẽ giúp ích cho những ai quan tâm đến việc tự chế tạo drone với vi điều khiển STM32.
