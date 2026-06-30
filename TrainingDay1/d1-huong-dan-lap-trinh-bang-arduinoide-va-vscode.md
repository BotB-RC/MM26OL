---
title: Hướng dẫn lập trình Arduino bằng Arduino IDE và VS Code

---

# I. Lập trình Arduino bằng Arduino IDE 

> Đây là cách lập trình truyền thống của Arduino. Nếu bạn mới bắt đầu học Arduino hoặc chỉ muốn thử nghiệm nhanh, Arduino IDE là một lựa chọn đơn giản và dễ sử dụng.

---
[TOC]

---
# Arduino ide

## 1. Cài đặt Arduino IDE

Tải Arduino IDE tại:

https://www.arduino.cc/en/software

Sau khi tải về, tiến hành cài đặt như các phần mềm thông thường.

---

## 2. Kết nối Arduino

1. Dùng cáp USB kết nối Arduino với máy tính.
2. Mở Arduino IDE.

---

## 3. Chọn loại bo mạch

Trên thanh menu chọn:

```text
Tools
    └── Board
```

Sau đó chọn đúng bo mạch.

Ví dụ:

- Arduino Uno
- Arduino Nano
- Arduino Mega 2560

---

## 4. Chọn cổng COM

Tiếp tục chọn:

```text
Tools
    └── Port
```

Ví dụ:

```text
COM3
COM5
COM8
```

Nếu không thấy cổng COM:

- Kiểm tra lại cáp USB.
- Kiểm tra Driver của bo mạch.
- Thử đổi cổng USB khác.

---

## 5. Viết chương trình

Ví dụ:

```cpp
void setup()
{
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop()
{
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);

    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
}
```

---

## 6. Kiểm tra chương trình (Verify)

Nhấn nút:

```text
✔ Verify
```

Arduino IDE sẽ biên dịch chương trình và thông báo nếu có lỗi.

---

## 7. Nạp chương trình

Nhấn nút:

```text
→ Upload
```

Chờ vài giây đến khi hiện thông báo:

```text
Done Uploading.
```

Lúc này chương trình đã được nạp lên Arduino.

---

## 8. Mở Serial Monitor

Nếu chương trình sử dụng:

```cpp
Serial.begin(9600);
```

Bạn có thể mở:

```text
Tools
    └── Serial Monitor
```

Hoặc nhấn:

```text
Ctrl + Shift + M
```

Đảm bảo tốc độ Baud ở góc dưới bên phải trùng với:

```cpp
Serial.begin(9600);
```

---

## 9. Cài đặt thư viện

Trong Arduino IDE chọn:

```text
Sketch
    └── Include Library
            └── Manage Libraries...
```

Tìm kiếm tên thư viện.

Ví dụ:

- Servo
- LiquidCrystal_I2C
- DHT sensor library
- Adafruit GFX

Sau đó nhấn **Install**.

---

## 10. Cấu trúc chương trình Arduino

Mọi chương trình Arduino đều gồm hai hàm chính:

```cpp
void setup()
{
    // Chạy một lần khi Arduino khởi động
}

void loop()
{
    // Lặp đi lặp lại liên tục
}
```

---

# Visual Studio Code

> Tài liệu này chỉ áp dụng cho người dùng sử dụng **PlatformIO trong Visual Studio Code (VS Code)** để lập trình Arduino.
>
> Không sử dụng Arduino IDE trong hướng dẫn này.

---

# 1. Cài đặt phần mềm cần thiết

## Bước 1: Cài đặt Visual Studio Code

Tải VS Code tại:

https://code.visualstudio.com/

Sau khi tải xong, tiến hành cài đặt như bình thường.

---

## Bước 2: Cài đặt PlatformIO

1. Mở VS Code.
2. Chọn **Extensions** (hoặc nhấn `Ctrl + Shift + X`).
3. Tìm kiếm:

```
PlatformIO IDE
```

4. Nhấn **Install**.
5. Khởi động lại VS Code nếu được yêu cầu.

Sau khi cài đặt thành công, bên trái màn hình sẽ xuất hiện biểu tượng PlatformIO.

---

# 2. Tạo dự án Arduino mới

1. Nhấn vào biểu tượng **PlatformIO**.
2. Chọn:

```
PIO Home → New Project
```

3. Điền thông tin dự án.

Ví dụ:

```
Project Name: Blink
Board: Arduino Uno
Framework: Arduino
```

4. Nhấn **Finish**.

PlatformIO sẽ tự động tạo cấu trúc thư mục dự án.

---

# 3. Cấu trúc dự án

Sau khi tạo xong, dự án sẽ có dạng:

```text
Project
│
├── src
│   └── main.cpp
│
├── lib
│
├── include
│
└── platformio.ini
```

Trong đó:

- `src/main.cpp`: Chứa mã nguồn chính.
- `lib/`: Chứa thư viện tự tạo.
- `include/`: Chứa file header.
- `platformio.ini`: File cấu hình dự án.

---

# 4. Viết chương trình Arduino

Mở file:

```text
src/main.cpp
```

Ví dụ chương trình nhấp nháy LED:

```cpp
#include <Arduino.h>

void setup()
{
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop()
{
    digitalWrite(LED_BUILTIN, HIGH);
    delay(1000);

    digitalWrite(LED_BUILTIN, LOW);
    delay(1000);
}
```

---

# 5. Biên dịch chương trình

Có hai cách:

### Cách 1

Nhấn biểu tượng:

```text
✓ Build
```

ở thanh công cụ phía dưới.

### Cách 2

Nhấn tổ hợp phím:

```text
Ctrl + Alt + B
```

Nếu không có lỗi, chương trình sẽ được biên dịch thành công.

---

# 6. Nạp chương trình vào Arduino

1. Kết nối Arduino với máy tính bằng cáp USB.
2. Nhấn biểu tượng:

```text
→ Upload
```

hoặc:

```text
Ctrl + Alt + U
```

PlatformIO sẽ tự động biên dịch và nạp chương trình lên bo mạch.

---

# 7. Sử dụng Serial Monitor

Ví dụ:

```cpp
void setup()
{
    Serial.begin(9600);
}

void loop()
{
    Serial.println("Hello Arduino");
    delay(1000);
}
```

Mở Serial Monitor bằng:

```text
Ctrl + Alt + S
```

hoặc:

```text
PlatformIO → Monitor
```

---

# 8. Cài đặt thư viện

Không nên tự sao chép thư viện vào dự án.

Thay vào đó:

```text
PIO Home
→ Libraries
```

Tìm kiếm và cài đặt thư viện cần thiết.

Ví dụ:

- Servo
- LiquidCrystal_I2C
- Adafruit GFX
- DHT Sensor Library

---

# 9. File cấu hình platformio.ini

Ví dụ cho esp32:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200

lib_deps =
    adafruit/Adafruit VL53L0X
    adafruit/Adafruit NeoPixel
    robtillaart/PCF8574
```

---

# 10. Các lỗi thường gặp

## Không upload được chương trình

Kiểm tra:

- Cáp USB có hỗ trợ truyền dữ liệu không.
- Đã chọn đúng loại bo mạch chưa.
- Cổng COM có bị chương trình khác chiếm dụng không.

---

## Không thấy dữ liệu trên Serial Monitor

Kiểm tra:

```cpp
Serial.begin(9600);
```

và đảm bảo:

```ini
monitor_speed = 9600
```

có cùng tốc độ baud.

---

## Thiếu thư viện

Nếu xuất hiện lỗi:

```text
No such file or directory
```

Hãy cài thư viện thông qua:

```text
PIO Home → Libraries
```

---


# Quy trình làm việc được khuyến nghị

```text
Tạo dự án
    ↓
Viết chương trình
    ↓
Build
    ↓
Sửa lỗi
    ↓
Upload
    ↓
Kiểm tra hoạt động
    ↓
Nộp dự án
```

# Các thư viện, board cần cài đặt khi lập trình micromouse

## 1. Cài board esp32
File → Preferences

Thêm vào Additional Boards Manager URLs:

https://espressif.github.io/arduino-esp32/package_esp32_index.json

Sau đó

Tools Board → Boards Manager

* Tìm esp32 cài: esp32 by Espressif Systems
---
## 2. Cài thư viện
* Adafruit VL53L0X
* Adafruit NeoPixel
* PCF8574
---