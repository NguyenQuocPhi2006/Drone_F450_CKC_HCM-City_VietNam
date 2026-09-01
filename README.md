# Drone_F450_CKC_HCM-City_VietNam

ESP32 drone project with F450 frame in HCM City

## Demo Video
[![Watch the video](https://img.youtube.com/vi/pp7vdn5QxUU/0.jpg)](https://youtube.com/shorts/pp7vdn5QxUU?si=EoC83cz4GQpj6je0)

---

## Giới thiệu
Tôi là người Việt Nam và đây là mã nguồn Drone trong dự án môn học của tôi. Tôi đã mất hơn 2 tháng cho phần lập trình Drone này, bắt đầu từ những bước cơ bản nhất cho đến việc cân bằng máy bay tốt nhất với khả năng của mình.

## Phần cứng sử dụng
- **Vi điều khiển**: STM32F411CEU6  
- **Cảm biến**: GY87 (MPU6050 + QMC5883L, không dùng BMP180)  
- **ESC**: 40A Brushless  
- **Động cơ**: 930KV 2212 phổ thông  
- **Cánh quạt**: 9 inch (ổn định hơn 10 inch do momen quán tính nhỏ hơn)

## Thuật toán & xử lý tín hiệu
- **Bộ lọc**: Mahony filter chạy ở tần số 1kHz  
- **PID**: Kết hợp PID trong (1kHz) và PID ngoài (100Hz)  
- **Quy trình**:
  - Đọc dữ liệu thô từ cảm biến → hiệu chỉnh → đưa vào bộ lọc Mahony  
  - Xử lý góc nghiêng mong muốn (tay cầm hoặc GPS)  
  - Nếu có GPS: dùng vòng lặp ngoài eSKF để tính vị trí X, Y trong không trung  
  - Nếu có cảm biến áp suất: kết hợp dữ liệu áp suất với gyro + accel trục Z  
  - Đưa kết quả qua PID để điều chỉnh **target_roll**, **target_pitch**, **target_altitude**  
  - Đầu ra cuối cùng là **Duty Cycle** cho 4 kênh PWM

## Giao tiếp & điều khiển
- **I2C, UART, SPI**: tất cả đều chạy với DMA  
- **Tay cầm FS-iAB6**: đọc dữ liệu qua DMA + ngắt, lấy 32 byte dữ liệu

## Kinh nghiệm & lưu ý
- **Hiệu chỉnh ESC**: tôi không làm bằng code, mà hiệu chỉnh qua tay cầm và debug bằng STM32CubeIDE (v1.9.0)  
- **Tinh chỉnh PID**: không chỉ phụ thuộc vào KP/KI/KD, mà còn vào cấu hình cảm biến (filter mode, dãy đo accel/gyro) và thông số bộ lọc Mahony  
- **Hiệu chỉnh cảm biến**:
  - **MPU6050**: hiệu chỉnh 3 trục gyro trên mặt phẳng lý tưởng, và 6 mặt của accel (±X, ±Y, ±Z)  
  - **QMC5883L (la bàn)**: mỗi vùng có từ trường khác nhau, cần hiệu chỉnh lại giống như la bàn trên điện thoại mới

---

## Kết luận
Dự án này là kết quả của quá trình học tập và thử nghiệm nghiêm túc. Tôi hy vọng mã nguồn và kinh nghiệm chia sẻ sẽ giúp ích cho những ai quan tâm đến việc tự chế tạo drone với vi điều khiển STM32.
