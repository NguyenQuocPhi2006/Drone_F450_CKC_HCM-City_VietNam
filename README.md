# Drone_F450_CKC_HCM-City_VietNam

## Giới thiệu dự án
Đây là mã nguồn Drone F450 cho đồ án môn học, được xây dựng theo hướng tự làm từ các phần cơ bản đến tối ưu khả năng cân bằng bay.

## Phần cứng sử dụng
- **MCU**: STM32F411CEU6
- **IMU + Compass**: GY-87 (dùng MPU6050 và QMC5883L, không dùng BMP180)
- **ESC / Motor**: ESC 40A brushless + động cơ 2212 930KV
- **Cánh quạt**: đã thử 9 inch và 10 inch; thực tế 9 inch ổn định hơn một chút do mô-men quán tính thấp hơn

## Kiến trúc điều khiển
- Vòng lặp trong chạy ở **1 kHz**:
  1. Đọc dữ liệu thô cảm biến
  2. Hiệu chỉnh dữ liệu thô
  3. Đưa vào bộ lọc **Mahony**
  4. Xử lý góc nghiêng mong muốn
  5. PID trong và trộn động cơ ra **Duty Cycle PWM 4 kênh**
- Vòng lặp ngoài chạy khoảng **100 Hz**:
  - Khi có GPS: dùng thêm vòng ngoài eSKF (kết hợp kinh/vĩ độ với quaternion q0 q1 q2 q3 từ vòng trong) để ước lượng vị trí x/y
  - Khi có cảm biến áp suất: kết hợp dữ liệu áp suất với gyro + accel trục z để ổn định độ cao
  - Đầu ra vòng ngoài qua PID để tạo target_roll, target_pitch, target_độ_cao cho vòng trong

## Giao tiếp và thu thập dữ liệu
- I2C/UART/SPI được cấu hình làm việc với **DMA**
- Dữ liệu RX từ tay cầm **FS-iA6B** dùng DMA + ngắt để lấy frame 32 byte

## Lưu ý quan trọng khi hiệu chỉnh và tinh chỉnh
1. **Hiệu chỉnh ESC**  
   Không hiệu chỉnh bằng code trong repo này. ESC đã được hiệu chỉnh trực tiếp bằng tay cầm và kiểm tra ở project khác để xác nhận min/max của 4 ESC đồng đều (debug bằng STM32CubeIDE 1.9.0).

2. **Drone bị trôi/rung không chỉ do PID**  
   Ngoài Kp/Ki/Kd, độ ổn định còn phụ thuộc mạnh vào:
   - cấu hình filter của cảm biến
   - dải đo accel/gyro
   - hệ số Mahony (Kp/Ki)

3. **Hiệu chỉnh MPU6050 (6 trục)**  
   - Gyro 3 trục: đặt trên mặt phẳng càng chuẩn càng tốt, đo nhiều hướng và lấy trung bình
   - Accel 3 trục: hiệu chỉnh đủ 6 mặt (x = -1/+1, y = -1/+1, z = -1/+1 với dải 2G), đặt vuông góc với mặt phẳng chuẩn để có giá trị gần lý tưởng

4. **Hiệu chỉnh la bàn QMC5883L**  
   Từ trường thay đổi theo vị trí địa lý nên phải hiệu chỉnh lại theo khu vực sử dụng, tương tự cách hiệu chỉnh la bàn trên điện thoại.
