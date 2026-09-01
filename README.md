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
Tôi là sinh viên trường CKC và đây là mã nguồn Drone trong dự án môn học của tôi. Tôi đã mất hơn 2 tháng cho phần lập trình Drone này, bắt đầu từ những thứ cơ bản nhất cho đến việc cân bằng máy bay tốt nhất với khả năng của mình.

## Chi Tiết Kỹ Thuật & Kiến Trúc Thuật Toán

Mã nguồn sử dụng module **GY-87** (gồm **MPU6050** và **QMC5883L/P**, không sử dụng cảm biến BMP180 tích hợp sẵn) kết hợp với **bộ lọc Mahony** chạy thời gian thực ở tần số **1 kHz**.

Toàn bộ chu trình xử lý điều khiển và ước lượng trạng thái được phân tầng rõ ràng qua hai vòng lặp lồng nhau (Cascaded Loops):

### 1. Vòng lặp bên ngoài (Outer Loop - 100 Hz): Xử lý vị trí, độ cao và góc mong muốn
* **Định vị GPS & Thuật toán eSKF:** Khi có tín hiệu GPS, vòng lặp ngoài áp dụng bộ lọc **eSKF (Error-State Kalman Filter)**. Thuật toán sử dụng dữ liệu kinh độ (Longitude) và vĩ độ (Latitude) kết hợp trực tiếp với 4 thành phần Quaternion (q_0, q_1, q_2, q_3) được tính toán từ vòng lặp trong để ước lượng tọa độ vị trí không gian $(X, Y)$ của máy bay.
* **Cảm biến áp suất khí quyển:** Khi kết nối thêm cảm biến áp suất, dữ liệu độ cao đo được sẽ kết hợp (sensor fusion) cùng dữ liệu tốc độ góc (Gyro) và gia tốc trục Z (Accel Z) từ MPU6050 nhằm ước lượng độ cao chuẩn xác.
* **Xử lý góc mong muốn & PID Ngoài (100 Hz):** Dựa vào lệnh điều khiển từ tay cầm hoặc dữ liệu dẫn đường GPS, vòng lặp ngoài tính toán sai lệch và đưa qua bộ điều khiển **PID Ngoài** để cập nhật liên tục các giá trị mục tiêu: `target_roll`, `target_pitch` và `target_altitude`.

### 2. Vòng lặp bên trong (Inner Loop - 1 kHz): Ước lượng tư thế & Điều khiển cân bằng
* **Thu thập & Tiền xử lý cảm biến:** Đọc toàn bộ dữ liệu thô (Raw Data) từ MPU6050 và QMC5883L/P, đưa qua khâu bù trừ sai số (Calibration/Offset) rồi nạp vào **bộ lọc Mahony** ở tần số **1 kHz** để tính toán ma trận Quaternion (q_0, q_1, q_2, q_3) và góc Euler thực tế.
* **PID Trong (Rate Controller - 1 kHz):** Các giá trị đầu ra từ vòng lặp ngoài (`target_roll`, `target_pitch`, `target_altitude`) được truyền vào vòng lặp PID trong. Sự phối hợp giữa **PID Trong (1 kHz)** và **PID Ngoài (100 Hz)** đảm bảo phản hồi tức thì với các biến động góc quay và triệt tiêu rung lắc.
* **Đầu ra điều khiển:** Kết quả sau cùng của vòng lặp là giá trị **Duty Cycle** cập nhật trực tiếp cho 4 kênh PWM Timer điều khiển 4 ESC / Động cơ.

---

## Tối Ưu Hóa Giao Tiếp Ngoại Vi (DMA)

* **I2C / UART / SPI:** Tất cả các giao thức truyền nhận dữ liệu với cảm biến và ngoại vi đều được cấu hình hoạt động hoàn toàn qua **DMA (Direct Memory Access)**, giúp giải phóng CPU Cortex-M4 khỏi việc chờ dữ liệu (non-blocking).
* **Bộ thu RX FS-iA6B:** Đọc gói tin dữ liệu chuẩn **32 bytes** từ bộ thu bằng cơ chế kết hợp **DMA + Ngắt UART (IDLE Line Interrupt)**, đảm bảo dữ liệu tay cầm điều khiển được cập nhật liên tục, không trễ và không làm nghẽn chu kỳ xử lý 1 kHz của hệ thống. 

- **Vi điều khiển**: STM32F411CEU6  
- **Động lực**: ESC40A Brushless + động cơ 930KV 2212  
- **Cánh quạt**: 9 inch (ổn định hơn 10 inch do momen quán tính nhỏ hơn)

---

## Những vấn đề mình cần nhấn mạnh

Đây là những vấn đề cốt lõi được đúc kết trực tiếp qua quá trình thử nghiệm thực tế mà bất kỳ ai tự làm Flight Controller cũng cần đặc biệt lưu ý:

### 1. Vấn đề hiệu chỉnh ESC (ESC Calibration)
* **Không hiệu chỉnh trực tiếp bằng code trong project bay:** Tôi thực hiện hiệu chỉnh dải xung Min/Max của ESC thông qua tay cầm điều khiển.
* **Kiểm tra độc lập trên STM32CubeIDE (v1.9.0):** Để đảm bảo độ an toàn và chuẩn xác tuyệt đối, tôi tạo một project riêng để test và theo dõi giá trị xung Min/Max của cả 4 ESC qua tính năng **Live Expressions / Debug** trong STM32CubeIDE. Việc này giúp xác nhận 4 ESC hoàn toàn đồng pha và nhận đúng dải xung trước khi đưa vào code điều khiển chính.

### 2. Hiện tượng Drone bị trôi hoặc rung lắc (Oscillation & Drift)
Nếu máy bay gặp tình trạng rung lắc hoặc không giữ được vị trí, nguyên nhân **không chỉ nằm ở các hệ số K_p, K_i, K_d của PID**, mà còn phụ thuộc rất lớn vào các yếu tố nền tảng sau:
* **Cấu hình phần cứng cảm biến:** Chế độ lọc số nội bộ (**Filter Mode / DLPF**), việc lựa chọn **dãy đo của Accelerometer** và **dãy đo của Gyroscope** có phù hợp với độ rung của khung máy bay hay không.
* **Thông số K_p, K_i của bộ lọc Mahony:** Bộ lọc Mahony đóng vai trò tối quan trọng quyết định độ mượt và độ chính xác của góc ước lượng. Tinh chỉnh trọng số K_p, K_i của Mahony chuẩn xác sẽ quyết định phần lớn độ ổn định của máy bay trước khi bàn đến PID.

### 3. Hiệu chuẩn 6 trục cảm biến MPU6050
* **Hiệu chuẩn 3 trục Gyroscope:** Đặt máy bay lên một mặt phẳng lý tưởng nhất có thể. Tiến hành lấy mẫu hiệu chỉnh ở **cả 4 hướng xoay khác nhau**, sau đó tính giá trị trung bình để triệt tiêu hoàn toàn độ trôi (offset/drift).
* **Hiệu chuẩn 3 trục Accelerometer trên 6 mặt không gian:** Bắt buộc phải hiệu chỉnh đầy đủ cả 6 hướng tương ứng với trọng lực Trái Đất:
  * Ví dụ ở dãy đo \pm 2g, các giá trị mục tiêu sẽ là: X = -1, X = +1, Y = -1, Y = +1, Z = -1, Z = +1.
  * **Lưu ý thao tác:** Phải đặt máy bay vuông góc nhất có thể với mặt phẳng ngang để thu được các giá trị xấp xỉ lý tưởng thực tế. Không cần ép buộc số đo phải bằng tuyệt đối \pm 1.000, việc lấy được giá trị xấp xỉ thực tế ổn định mới là yếu tố quyết định.

### 4. Hiệu chuẩn 3 trục cảm biến La bàn (QMC5883L)
* **Từ trường cục bộ:** Mỗi khu vực địa lý, vị trí địa lý đều có độ lệch từ trường và mức can nhiễu khác nhau, nên việc hiệu chuẩn lại la bàn là bắt buộc khi thay đổi môi trường bay.
* **Cách thực hiện:** Thao tác tương tự như khi bạn cầm một chiếc điện thoại mới mua và xoay hình số 8 / xoay đa trục để hiệu chuẩn lại la bàn. Việc này rất đơn giản nhưng bắt buộc phải làm để dữ liệu Yaw và định hướng không bị sai lệch.

---

## Hình ảnh minh họa

  <img width="300" alt="Drone" src="https://github.com/user-attachments/assets/d9528ef6-04e1-4c48-8c56-7e9497522348" />
  
## Thiết kế PCB

  <img width="300" alt="PCB" src="https://github.com/user-attachments/assets/60229d58-c28d-4c60-84ab-a91336c02467" />
  
## Màng hình hiển thị thông số trên tay cầm FS_iB6A
chỗ này mình sẽ đăng 1 bài repo riêng về module Esp32_2432S028

  <img width="300" alt="Display" src="https://github.com/user-attachments/assets/6484176b-4d98-43fc-a8e5-21764e5d9472" />

---

## Kết luận
Dự án này là kết quả của quá trình học tập và thử nghiệm nghiêm túc. Tôi hy vọng mã nguồn và kinh nghiệm chia sẻ sẽ giúp ích cho những ai quan tâm đến việc tự chế tạo drone với vi điều khiển STM32.
