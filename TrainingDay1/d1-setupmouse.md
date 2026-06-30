---
title: D1_SETUPMOUSE<3

---

*Tài liệu này hướng dẫn các bước cần thiết để lập trình micromouse - từ cài đặt môi trường, điều khiển motor, đọc encoder, đọc cảm biến đến các kỹ thuật điều hướng cơ bản. Sau khi hoàn thành, các bạn sẽ có đủ nền tảng để tự xây dựng chương trình điều khiển cho robot của mình.*
# 1. Cấu tạo micromouse
- ESP32S3
- 2 motor
- 2 encoder
- 4 cảm biến khoảng cách
- Driver motor

# 2. Điều khiển motor
```cpp
// cho hai motor trái, phải quay tiến với PWM 255
motorL.movePWM(255);
motorR.movePWM(255); 

vTaskDelay(1000); // Tạm dừng chương trình trong 1000 ms (1 giây).

// cho hai motor dừng
motorL.movePWM(0);     
motorR.movePWM(0);
```
- **motorL** và **motorR** là hai object điều khiển motor đã được khai báo sẵn trong project.
- Hàm movePWM(speed), speed nhận cả giá trị dương và âm. Dương là tiến, âm là lùi.
- giá trị của speed cần được test phù hợp với từng mouse.

# 3. Đọc encoder
## 3.1. Encoder?
- Encoder là cảm biến gắn trên bánh xe, đếm số bước bánh đã quay. Mỗi bước gọi là 1 tick.
- Dùng để biết robot đã đi bao xa hoặc quay bao nhiêu góc.

## 3.2. Đọc encoder

```cpp
// đặt lại bộ đếm về 0
motorL.resetTicks(); 
motorR.resetTicks();

// in ra màn hình Serial số tick
Serial.println(motorL.getTicks());
Serial.println(motorR.getTicks());
```

- resetTicks() đặt lại bộ đếm về 0 — thường gọi trước mỗi hành động mới để đo chính xác.
    - Nếu không reset, robot vừa đi thẳng xong rồi quẹo, ticks sẽ cộng dồn từ lần đi thẳng → đo góc quẹo sai.
# 4. Đọc cảm biến & calibration
Hàm **read_sensor_raw()** đọc giá trị thô từ 4 cảm biến, áp dụng công thức hiệu chỉnh (2-point calibration) rồi cập nhật khoảng cách vào các biến toàn cục.

```cpp
void read_sensor_raw(){ // hàm đọc khoảng cách bằng cảm biến
    
// distL, distM1, distM2, distR là biến lưu khoảng cách thực tế từ cảm biến đến tường
// tính bằng giá trị cảm biến đọc được và cộng thêm calibrate (giá trị bù) tương ứng để tăng độ chính xác
// công thức tính được giải thích ở phần 5.1.
  distL  = aL  * sensorL.readRangeResult()  + bL;
  distM1 = aM1 * sensorM1.readRangeResult() + bM1;
  distM2 = aM2 * sensorM2.readRangeResult() + bM2;
  distR  = aR  * sensorR.readRangeResult()  + bR;
```

Sau khi gọi hàm này thì ta có thể đọc trực tiếp từ biến:
```cpp
read_sensor_raw();
Serial.println(distL);   // mm
Serial.println(distM1);
Serial.println(distM2);
Serial.println(distR);
```
- Gọi read_sensor_raw() trước là vì các biến dist không tự cập nhật, phải gọi hàm thì mới đọc từ sensor
-  Đơn vị là mm.

## 4.1. Calibration cảm biến

### 4.1.1. VẤN ĐỀ
Hai sensor cùng loại nhưng đọc cùng một khoảng cách có thể ra số khác nhau - cần calibrate để bù lại.

Ví dụ: Khoảng cách thực là 50 mm mà sensor đọc ra 47mm hoặc 53m

---
### 4.1.2. CÁCH GIẢI QUYẾT

**B1.** Chọn 2 khoảng cách chuẩn

**B2.** Đo giá trị thô của cảm biến tại mỗi khoảng cách.

**B3.** Lập hệ phương trình từ 2 cặp giá trị đo

**B4.** Giải hệ phương trình để tìm hai hệ số:
- a: hệ số scale (gain)
- b: hệ số offset

**B5.** Cập nhật công thức tính khoảng cách trong chương trình

**B6.** Kiểm tra lại ở một hoặc nhiều khoảng cách khác để đánh giá độ chính xác.

---
### 4.1.3. VÍ DỤ

**B1.** Đặt vật cản ở 2 khoảng cách chuẩn (ví dụ: 50 mm và 150 mm).

**B2.** Đọc giá trị thô từ cảm biến tại mỗi khoảng cách.
- 50 mm → cảm biến đọc 47 mm
- 150 mm → cảm biến đọc 154 mm

**B3.** Tính hai hệ số hiệu chỉnh từ hai cặp giá trị trên bằng hệ phương trình:
$$
\begin{cases}
47a + b = 50 \\
154a + b = 150
\end{cases}
$$

→ a=0.9346
→ b=6.07

**B4.** Áp dụng vào chương trình
```cpp
distL = 0.9346 * sensorL.readRangeResult() + 6.07;
```

**B5.** Kiểm tra lại ở các khoảng cách khác để đánh giá độ chính xác.

# 5. Đọc tường
So sánh distM1, distM2, distL, distR với một ngưỡng threshold để xác định có tường hay không
- Ví dụ (CHỈ LÀ VÍ DỤ):
```cpp
read_sensor_raw();
if(distM1 < 100) // có tường phía trước
if(distL < 100)  // có tường bên trái
if(distR < 100)  // có tường bên phải
```

# 6. Tham khảo 

Code SETUP mẫu: [Code SETUP mẫu](d1-code-setup.md)

Nội dung code:
| Khối | Dòng | Làm gì |
| -------- | -------- | -------- |
| Include & alias | 1–20 | Khai báo thư viện, tên kiểu dữ liệu ngắn gọn (ui8, i32...) |
| Config PWM | 24–29 | Tần số PWM, độ phân giải, kênh PWM cho 2 motor |
| Config pin encoder | 31–36 | Chân A/B của encoder trái/phải |
| Config pin motor | 38–45 | Chân IN1/IN2/PWM của motor trái/phải |
| Biến encoder | 49–62 | Biến đếm tick, thời gian, tốc độ, chiều quay |
| ISR encoder | 64–84 | Hàm ngắt tự động chạy khi bánh quay, cập nhật ticksL/R |
| Motor struct | 86–166 | Định nghĩa struct Motor với các hàm begin(), movePWM(), getTicks(), resetTicks(), stop() |
| Motor instance | 168–185 | Tạo 2 object motorL, motorR từ struct Motor |
| Config sensor | 189–200 | Chân I2C, địa chỉ PCF8574, mapping XSHUT các sensor |
| Sensor instance & dist | 202–214 | Tạo 4 object sensor, 4 biến distL/M1/M2/R |
| Filter buffer | 216–227 | Biến lưu giá trị cũ và hàm median3() để lọc nhiễu |
| PCF control | 229–232 | Hàm xshut_write() bật/tắt từng sensor qua PCF8574 |
| init_sensor() | 234–253 | Hàm nội bộ khởi tạo từng sensor, gán địa chỉ I2C |
| sensor_init() | 255–292 | Khởi tạo toàn bộ 4 sensor lần lượt |
| sensor_read() | 294–343 | Đọc sensor + lọc nhiễu (median + EMA) + calibration tuyến tính |
| read_sensor_raw() | 345–369 | Đọc sensor thô + calibration tuyến tính |
| sensor_print() | 371–377 | In 4 giá trị dist ra Serial để debug |
| setup() | 381–415 | Khởi tạo Serial, motor, encoder, sensor, chạy thử motor tiến/lùi |
| loop() | 418–420 | Để trống - chỗ các bạn viết logic vào đây |
