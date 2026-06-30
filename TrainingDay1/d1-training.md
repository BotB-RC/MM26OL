---
title: D1 Training

---

*Để đảm bảo tiến độ, các bạn vui lòng lưu ý thực hiện các yêu cầu theo hướng dẫn sau:*

*   *Cài đặt môi trường: mục 1*
*   *Hoàn thành 3 bài tập tại mục 2*
*   *Đọc hiểu kỹ tài liệu setup Micromouse tại mục 3.*


# NỘI DUNG CẦN CHUẨN BỊ CHO BUỔI TRAINING TIẾP THEO
## 1. Cài đặt môi trường
- Hướng dẫn lập trình bằng Arduino IDE và VSCode: [Hướng dẫn lập trình bằng Arduino IDE và VSCode](d1-huong-dan-lap-trinh-bang-arduinoide-va-vscode.md)
- Bạn có thể chọn sử dụng một trong hai công cụ trên.
### 1.1 Cài board esp32 (Arduino IDE)
1. File → Preferences
2. Tại **Additional Boards Manager URLs**, thêm:
3. [https://espressif.github.io/arduino-esp32/package\_esp32\_index.json](https://espressif.github.io/arduino-esp32/package_esp32_index.json)
4. Chọn **Tools → Board → Boards Manager...**.
5. Tìm **esp32**.
6. Cài đặt **esp32 by Espressif Systems**.
7. Sau khi cài xong, chọn board **ESP32S3**.

### 1.2 Cài board esp32 (Visual Studio Code)
1. Vào **Extension** (Ctrl + Shift + X) →  cài **PlatformIO, Serial Monitor**
2. **PlatformIO → New Project**
3. Board: **Espressif ESP32-S3-DevKitM-1**
4. Framework: **Arduino**
5. Tìm **platformio.ini**.
6. Sửa file **platformio.ini** thành:
```ini
[env:esp32-s3-devkitm-1]
platform = espressif32
board = esp32-s3-devkitm-1
framework = arduino
board_build.flash_size = 4MB
board_upload.flash_size = 4MB
board_build.flash_mode = dio
board_build.partitions = default.csv
upload_speed = 921600
monitor_speed = 115200
build_flags = 
	-D ARDUINO_USB_MODE=1
	-D ARDUINO_USB_CDC_ON_BOOT=1
````
7. Sau khi cài xong, vào src/**main.cpp** để bắt đầu code.
8. Bấm **Upload** (Ctrl + Alt + U) (nút phía trên bên phải) để nạp code.
9. **Ctrl + Shift + P → Chat: Focus on Serial Monitor View** để xem Serial Monitor

### 1.3 Thư viện cần cài đặt
- **Adafruit VL53L0X**: Thư viện điều khiển cảm biến khoảng cách bằng laser (ToF). Giúp robot đo chính xác khoảng cách đến tường theo đơn vị mm, ít bị sai số bởi màu sắc hay ánh sáng môi trường.
- **Adafruit NeoPixel**: Thư viện điều khiển LED màu (RGB đơn dây). Dùng để đổi màu sắc hệ thống LED trên robot nhằm báo hiệu các trạng thái (đang dò đường, đã tìm thấy đích, hoặc báo lỗi).
- **PCF8574**: Thư viện quản lý IC mở rộng chân giao tiếp I2C. Giúp robot tiết kiệm chân phần cứng của ESP32S3 mà vẫn bật/tắt và điều khiển được nhiều ngoại vi khác.
- **ESP32Encoder**: Thư viện cấu hình bộ đếm xung phần cứng (PCNT) của ESP32. Giúp ghi nhận chính xác từng "tick" quay của bánh xe ở tốc độ cao mà không làm nặng CPU, đảm bảo robot đo khoảng cách và góc rẽ không bị sai lệch.

## 2. BÀI TẬP
### Bài 1. LED
LED NeoPixel **dùng để debug trực quan** — biết robot đang làm gì mà không cần cáp hay màn hình. Ví dụ: xanh = đang chạy, đỏ = lỗi, vàng = chờ. Màu thể hiện điều gì là do bạn tự quyết định trong code.

LED NeoPixel là **loại LED RGB** — màu sắc được tạo từ 3 giá trị: Đỏ (Red), Xanh lá (Green), Xanh dương (Blue), mỗi giá trị từ 0–255. Trộn 3 màu này theo tỉ lệ khác nhau sẽ ra được hầu hết các màu:
```cpp
led.Color(255, 0, 0);   // Đỏ
led.Color(0, 255, 0);   // Xanh lá
led.Color(0, 0, 255);   // Xanh dương
led.Color(255, 255, 0); // Vàng (Đỏ + Xanh lá)
led.Color(0, 0, 0);     // Tắt (không màu nào)
```
---

```cpp
#include <Adafruit_NeoPixel.h>

#define LED_PIN 48
#define LED_COUNT 1

Adafruit_NeoPixel led(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  led.begin();
  led.setBrightness(50);
  led.show();
}

void loop() {
  // Đỏ
  led.setPixelColor(0, led.Color(255, 0, 0));
  led.show();
  vTaskDelay(500);

  // Xanh lá
  led.setPixelColor(0, led.Color(0, 255, 0));
  led.show();
  vTaskDelay(500);

  // Tắt
  led.setPixelColor(0, led.Color(0, 0, 0));
  led.show();
  vTaskDelay(500);
}
```
**Kết quả:** LED chớp đỏ → xanh lá → tắt → lặp lại.


### Bài 2. BUTTON BOOT
ESP32S3 có 2 nút vật lý:
- RESET - reset board, chạy lại code từ đầu
- BOOT (GPIO 0) - dùng như nút bấm thông thường trong code

Trong micromouse, nút BOOT thường dùng để bắt đầu chạy - robot đứng yên chờ, nhấn BOOT thì bắt đầu. 

Ngoài ra có thể dùng để reset trạng thái, chuyển chế độ, tùy bạn code. Ở bài này, ta dùng nút BOOT để bật/tắt LED.

```cpp
// 1. KHAI BÁO THƯ VIỆN
// Thêm thư viện Adafruit NeoPixel để có các hàm điều khiển LED RGB đặc biệt này
#include <Adafruit_NeoPixel.h>

// 2. ĐỊNH NGHĨA CÁC CHÂN CẮM (PHẦN CỨNG)
#define PIN_NEOPIXEL 48 // Khai báo chân dữ liệu kết nối với LED RGB trên board ESP32-S3 là chân số 48
#define PIN_BUTTON   0  // Khai báo chân kết nối với nút nhấn BOOT trên board là chân số 0
#define NUMPIXELS    1  // Khai báo số lượng LED NeoPixel có trên board (trên board chỉ có đúng 1 con LED)

// 3. KHỞI TẠO ĐỐI TƯỢNG ĐIỀU KHIỂN
// Tạo một đối tượng tên là "pixels" để quản lý LED với các tham số cấu hình:
// - NUMPIXELS: Số lượng LED (1)
// - PIN_NEOPIXEL: Chân điều khiển (48)
// - NEO_GRB + NEO_KHZ800: Kiểu sắp xếp màu (Green-Red-Blue) và tần số giao tiếp chuẩn của chip LED (800 Khz)
Adafruit_NeoPixel pixels(NUMPIXELS, PIN_NEOPIXEL, NEO_GRB + NEO_KHZ800);

// 4. HÀM CÀI ĐẶT BAN ĐẦU (Chạy duy nhất 1 lần khi cấp điện)
void setup() {
  // Cấu hình chân nút bấm BOOT làm cổng vào (INPUT) và bật điện trở kéo lên nội (PULLUP)
  // Điện trở PULLUP giữ cho chân này mặc định luôn ở mức CAO (1). Khi nhấn nút, chân sẽ bị nối đất xuống mức THẤP (0).
  pinMode(PIN_BUTTON, INPUT_PULLUP); 
  
  pixels.begin(); // Kích hoạt phần cứng và chuẩn bị thư viện NeoPixel sẵn sàng hoạt động
  pixels.clear(); // Xóa sạch dữ liệu màu cũ đang lưu trong bộ nhớ LED (đưa về trạng thái tắt hoàn toàn)
  pixels.show();  // Gửi lệnh xuất dữ liệu ra phần cứng để LED tắt ngay khi vừa khởi động
}

// 5. VÒNG LẶP CHÍNH (Trong ESP32, hàm này thực chất là một "Task" của FreeRTOS chạy liên tục)
void loop() {
  // BƯỚC 1: KIỂM TRA NÚT BẤM
  // digitalRead đọc trạng thái chân nút bấm. Nếu kết quả là LOW (mức 0) nghĩa là học sinh đang nhấn giữ nút BOOT.
  if (digitalRead(PIN_BUTTON) == LOW) {
    
    // --- LẦN 1: BẬT SÁNG LED ---
    // setPixelColor(vị trí LED, pixels.Color(Đỏ, Xanh Lá, Xanh Dương))
    // Tham số đầu tiên là 0 (vì đây là con LED đầu tiên và duy nhất, lập trình đếm từ 0)
    // Cài đặt màu Xanh Lá (Green = 255), các màu Đỏ và Xanh Dương bằng 0.
    pixels.setPixelColor(0, pixels.Color(0, 255, 0)); 
    pixels.show(); // Lệnh bắt buộc để đẩy dữ liệu màu vừa cài đặt hiển thị ra con LED thật
    
    // Thay vì dùng delay() truyền thống làm đơ chip, ta dùng vTaskDelay của RTOS
    // pdMS_TO_TICKS(200) đổi thời gian 200 mili-giây thành các nhịp (Ticks) của hệ điều hành
    // Lệnh này cho phép Task loop() "đi ngủ" 200ms, giải phóng CPU để làm việc hệ thống khác.
    vTaskDelay(pdMS_TO_TICKS(200));

    // --- LẦN 2: TẮT LED ---
    // Cài đặt cả 3 màu Đỏ, Xanh Lá, Xanh Dương bằng 0 để tạo ra màu đen (tức là TẮT LED)
    pixels.setPixelColor(0, pixels.Color(0, 0, 0));
    pixels.show(); // Cập nhật lệnh tắt ra con LED thật
    vTaskDelay(pdMS_TO_TICKS(200)); // Tiếp tục cho Task đi ngủ 200ms để tạo khoảng thời gian tắt.
    
  } 
  // BƯỚC 2: XỬ LÝ KHI KHÔNG NHẤN NÚT
  else {
    // Nếu không nhấn nút, ta luôn đảm bảo LED ở trạng thái TẮT
    pixels.setPixelColor(0, pixels.Color(0, 0, 0));
    pixels.show(); // Cập nhật lệnh tắt ra LED
    
    // >>> ĐOẠN QUAN TRỌNG NHẤT BÀI HỌC VỀ RTOS & WATCHDOG TIMER <
    // Khi không nhấn nút, chip chạy vào nhánh này. Nếu không có lệnh này, vòng lặp loop() sẽ chạy liên tục
    // với tốc độ hàng triệu lần/giây, chiếm 100% sức mạnh CPU và không cho các tác vụ ngầm hoạt động.
    // Lệnh vTaskDelay(1) ép hàm loop() dừng lại đúng 1 mili-giây.
    // Trong 1 mili-giây ngắn ngủi này, hệ thống sẽ tranh thủ "cho bộ đếm Watchdog Timer ăn".
    // Nhờ vậy, chip hiểu hệ thống vẫn an toàn, không bị treo và không kích hoạt lệnh tự động Reset (Crash).
    vTaskDelay(pdMS_TO_TICKS(1)); 
  }
}
```
**Kết quả:** Cấp điện cho board, mở Serial Monitor. Giữ nút BOOT → LED nhấp nháy xanh liên tục (200ms sáng, 200ms tắt). Thả nút → LED tắt hẳn.

### Bài 3. SERIAL
Serial dùng để in thông số ra màn hình trong lúc test và debug — ví dụ xem giá trị cảm biến, tick encoder, trạng thái robot. Mở Serial Monitor trên Arduino IDE để xem.

```cpp
void setup() {
  // 1. KHỞI TẠO CỔNG SERIAL
  // Mở cổng giao tiếp Serial với tốc độ truyền dữ liệu là 115200 baud (bits/giây).
  // Thầy/cô lưu ý nhắc học sinh phải chọn đúng số 115200 trên Serial Monitor thì mới đọc được chữ, chọn sai sẽ bị lỗi chữ loằng ngoằng.
  Serial.begin(115200); 
  // Đợi 1 giây (1000ms) bằng RTOS để chip ổn định cổng kết nối sau khi cắm cáp
  vTaskDelay(pdMS_TO_TICKS(1000)); 

  // 2. IN LỜI CHÀO BAN ĐẦU
  // Hàm Serial.println() dùng để in một dòng chữ từ chip lên màn hình máy tính và tự động xuống dòng.
  Serial.println("=========================================");
  Serial.println("XIN CHAO! Day la board ESP32-S3.");
  Serial.println("Hay nhap TEN cua em vao o phia tren roi an ENTER nhe:");
  Serial.println("=========================================");
}

void loop() {
  // 3. KIỂM TRA XEM CÓ DỮ LIỆU GỬI ĐẾN KHÔNG
  // Hàm Serial.available() giống như một người gác cửa. Nó kiểm tra xem học sinh đã gõ chữ gì và nhấn gửi chưa.
  // Nếu kết quả lớn hơn 0 (> 0), nghĩa là có ký tự đang nằm trong bộ nhớ đệm chờ chip xử lý.
  if (Serial.available() > 0) {
    
    // 4. ĐỌC DỮ LIỆU HỌC SINH VỪA NHẬP
    // Tạo một biến kiểu chuỗi chữ (String) tên là "tenHocSinh"
    // Hàm readStringUntil('\n') sẽ đọc toàn bộ các chữ cái học sinh gõ cho đến khi gặp ký tự xuống dòng (phím Enter).
    String tenHocSinh = Serial.readStringUntil('\n');
    
    // Hàm .trim() giúp loại bỏ các khoảng trắng thừa ở đầu hoặc cuối chuỗi (nếu học sinh lỡ tay gõ dấu cách)
    tenHocSinh.trim(); 

    // Kiểm tra nếu chuỗi nhận được không bị rỗng
    if (tenHocSinh.length() > 0) {
      // 5. PHẢN HỒI LẠI CHO HỌC SINH
      // In lời chào kết hợp với biến "tenHocSinh" vừa đọc được
      Serial.print("--> ESP32-S3 nói: Chào em ");
      Serial.print(tenHocSinh);
      Serial.println("! Chúc em có một bài học lập trình thật vui!");
      
      // In thêm độ dài của tên để tăng tính toán học/lập trình cho học sinh
      Serial.print("--> Bật mí: Tên của em có ");
      Serial.print(tenHocSinh.length());
      Serial.println(" ký tự đó nha.");
      Serial.println("-----------------------------------------");
    }
  }

  // Tránh treo Watchdog Timer khi không có dữ liệu gửi đến
  vTaskDelay(pdMS_TO_TICKS(1));
}
```
**Kết quả:**
- Mở Serial Monitor (chọn baudrate 115200)   
→ thấy lời chào "XIN CHAO! Day la board ESP32-S3...". Nhập tên vào ô phía trên rồi nhấn Enter  
→ board phản hồi lại "Chào em [tên]! Chúc em có một bài học lập trình thật vui!" kèm số ký tự trong tên. 

## 3. TÀI LIỆU SETUP MICROMOUSE
**Link tài liệu:** [Tài liệu Setup Micromouse](d1-setupmouse.md)
