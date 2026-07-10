# HƯỚNG DẪN ĐIỀU KHIỂN MICROMOUSE: ENCODER, CẢM BIẾN TƯỜNG VÀ PID

## 1. Bản chất của Encoder trong Micromouse

**Encoder (Bộ mã hóa quay)** là mắt xích quan trọng giúp Robot xác định vị trí trong không gian hai chiều của mê cung mà không phụ thuộc hoàn toàn vào cảm biến ngoại vi.

* **Chức năng:** Đếm số xung (ticks) khi bánh xe quay, từ đó suy ra quãng đường và vận tốc thực tế.
* **Công thức quy đổi hình học:**
  * Chu vi bánh xe: $C = \pi \times d$ *(với $d$ là đường kính bánh xe)*
  * Độ dịch chuyển trên 1 tick: $D_{tick} = \frac{C}{N}$ *(với $N$ là tổng số xung trên 1 vòng quay)*
  * **Quãng đường thực tế:** $\text{Distance} = \text{Ticks} \times D_{tick}$

* **Cách tìm tổng số xung ticks khi bánh xe quay được 1 vòng trên thực tế:** Đánh dấu bánh xe, dùng tay xoay đúng 1 vòng trọn vẹn và đọc tổng số xung (ticks) hiển thị trên màn hình Serial.

---

## 2. Khái quát PID
### 2.1. Lý thuyết điều khiển PID tổng quan

Bộ điều khiển PID liên tục tính toán giá trị sai số $e(t)$ giữa **Giá trị đặt (SetPoint)** và **Giá trị phản hồi (Process Variable)** để tối ưu hóa công suất động cơ:

$$u(t) = K_p e(t) + K_i \int_{0}^{t} e(\tau)d\tau + K_d \frac{de(t)}{dt}$$

* **$e(t)$ (Error - Sai số):** Là độ lệch giữa vị trí mong muốn và vị trí thực tế tại thời điểm $t$.
  $$\text{Sai số } e(t) = \text{SetPoint (Đích)} - \text{Process Variable (Thực tế)}$$
  * *Ví dụ:* Bạn đặt robot đi 1000 ticks ($\text{SetPoint}$), hiện tại robot mới đi được 700 ticks ($\text{Process Variable}$) $\rightarrow e(t) = 1000 - 700 = 300$ ticks.
* **$u(t)$ (Control Signal - Tín hiệu điều khiển):** Là giá trị đầu ra sau khi tính toán PID (thường là tốc độ hoặc xung PWM cấp cho motor) tại thời điểm $t$, để motor tăng/giảm tốc độ để đạt được mục tiêu.
* **$K_p$ (Tỷ lệ):** Tạo lực đẩy tỉ lệ thuận với sai số hiện tại. Giúp đáp ứng nhanh nhưng dễ gây vọt lố.
* **$K_i$ (Tích phân):** Cộng dồn các sai số nhỏ trong quá khứ để triệt tiêu hoàn toàn sai số xác lập (dừng đúng đích).
* **$K_d$ (Vi phân):** Dự đoán xu hướng sai số dựa trên tốc độ thay đổi, đóng vai trò như một "hệ thống phanh" giảm dao động.

---

>  **LƯU Ý THỰC TẾ khi lập trình: TẦN SỐ LẤY MẪU (SAMPLING RATE)**
> * **Bản chất:** Công thức trên tính theo thời gian liên tục. Khi lập trình, PID chạy **rời rạc** nên khâu tích phân (I) và vi phân (D) phụ thuộc hoàn toàn vào khoảng thời gian $\Delta t$ giữa các lần tính.
> * **Hậu quả:** Nếu để hàm PID chạy tự do trong vòng lặp `while(true)`, thời gian mỗi vòng lặp sẽ bị trồi sụt, khiến bộ phanh $K_d$ bị nhiễu và làm robot chạy giật cục.
> * **Giải pháp:** Đặt toàn bộ logic tính PID vào trong một **Timer Interrupt (Ngắt bộ định thời)** để ép vi điều khiển tính toán theo một chu kỳ cố định.
> * **Tần số lý tưởng:** Từ **100Hz đến 500Hz** (gọi hàm PID đều đặn sau mỗi 10ms đến 2ms).

---

### 2.2. Hiện tượng thực tế khi Tuning sai hệ số ($K_p, K_i, K_d$)



##### Hệ số $K_p$ (Tỷ lệ - Lực đẩy chính)
* **Thấp quá:** Động cơ không đủ lực thắng ma sát. Robot chạy lờ đờ, chậm chạp hoặc đứng im luôn khi gần đến đích (do sai số nhỏ không đủ tạo ra PWM để quay bánh).
* **Cao quá:** Robot phóng đi rất nhanh nhưng bị **vọt lố (overshoot)** qua vạch đích. Sau đó nó phải giật lùi lại, tạo ra chu kỳ tiến-lùi liên tục (dao động mạnh) hoặc đâm sầm vào tường.

##### Hệ số $K_i$ (Tích phân - Khử sai số xác lập)
* **Thấp quá (hoặc bằng 0):** Robot chạy gần đến đích (ví dụ còn thiếu 5 ticks) thì dừng hẳn do lực $K_p$ lúc này quá yếu. Robot bị hiện tượng "early stopping".
* **Cao quá:** Sai số tích lũy quá nhanh khiến robot bị "quá tải tích phân" (Integral Windup). Robot sẽ bị lắc đuôi cực mạnh khi đi thẳng hoặc xoay quá vòng mất kiểm soát.

##### Hệ số $K_d$ (Vi phân - Hệ thống phanh)
* **Thấp quá:** Hệ thống không có phanh. Robot kiểm soát hướng lái kém, dễ bị đảo đuôi liên tục theo hình chữ chi (zigzag) khi cố canh giữa ô bằng cảm biến.
* **Cao quá:** "Phanh" quá gắt. Động cơ bị giật cục do $K_d$ cực kỳ nhạy cảm với nhiễu của cảm biến/encoder. Robot chạy không mượt, kêu rè rè và cực kỳ tốn pin.

---

## 3. Thiết kế các thuật toán PID cụ thể cho Micromouse

### 3.1. Bài toán 1: Chạy thẳng đúng khoảng cách (Đủ Ticks)

* **Sai số khoảng cách (Tốc độ chạy):** $E_{\text{dist}} = \text{Target Ticks} - \frac{\text{Left Ticks} + \text{Right Ticks}}{2}$
* **Sai số hướng tâm (Giữ thẳng):** $E_{\text{steer}} = \text{Left Ticks} - \text{Right Ticks}$

```c
void moveStraight(int target_ticks) {
    
    // --- PID KHOẢNG CÁCH (CHẠY TIẾN) ---
    int e_dist;          // Sai số hiện tại = Đích - Thực tế (Khâu P: Tạo lực chạy)
    int e_dist_prev = 0; // Sai số vòng trước (Khâu D: Hãm phanh khi gần đến đích)
    int e_dist_sum = 0;  // Tổng sai số tích lũy (Khâu I: Bù lực khi xe dừng non)
    
    resetEncoders(); // đặt lại số ticks của 2 motor về 0

    while (true) {
        int current_left = getLeftEncoder();
        int current_right = getRightEncoder();

        // 1. PID Khoảng cách
        e_dist = target_ticks - (current_left + current_right) / 2;
        e_dist_sum += e_dist;
        int base_speed = (Kp_dist * e_dist) + (Ki_dist * e_dist_sum) + (Kd_dist * (e_dist - e_dist_prev));
        e_dist_prev = e_dist;

        // Cấp nguồn động cơ
        setMotorSpeed(LEFT, base_speed - steer_correction);
        setMotorSpeed(RIGHT, base_speed + steer_correction);

        // Điều kiện dừng
        if (abs(e_dist) < DIST_THRESHOLD && abs(getVelocity()) < SPEED_THRESHOLD) break;
    }
    stopMotors(); // tắt motor/đưa speed về 0
}
```

---

### 3.2. Bài toán 2: Xoay tại chỗ chính xác (Xoay góc bằng Ticks)

Khi xoay tại chỗ, hai bánh quay ngược chiều.
* **Công thức quy đổi góc sang Ticks:**
  $$\text{Target Ticks}_{\text{turn}} = \frac{\theta^\circ \times \pi \times W}{360^\circ \times D_{tick}}$$
  *(W: Khoảng cách giữa 2 bánh xe)*
* **Giả định:** 
* **Sai số quay:** $E_{\text{turn}} = \text{Target Ticks}_{\text{turn}} - \frac{\text{Left Ticks} - \text{Right Ticks}}{2}$
    * **Giả định bắt buộc:** Hàm đọc Encoder phải trả về giá trị tăng (dương) khi quay tiến và giảm (âm) khi quay lùi để phép trừ (Left Ticks - Right Ticks) không bị triệt tiêu bằng 0.



```c
void turnInPlace(int target_turn_ticks) {
    int e_turn, e_turn_prev = 0, e_turn_sum = 0;
    resetEncoders(); // Reset xung về 0 trước khi xoay

    while (true) {
        int current_left = getLeftEncoder();
        int current_right = getRightEncoder(); 

        // Tính sai số: Mục tiêu - [(Trái - Phải) / 2] (Trừ của trừ thành cộng)
        e_turn = target_turn_ticks - (current_left - current_right) / 2;
        e_turn_sum += e_turn; // Tích lũy sai số cho khâu I
        
        // Tính toán PID để ra tốc độ xoay
        int turn_speed = (Kp_turn * e_turn) + (Ki_turn * e_turn_sum) + (Kd_turn * (e_turn - e_turn_prev));
        e_turn_prev = e_turn; // Lưu sai số vòng này cho vòng sau

        // Điều khiển động cơ: Xoay phải tại chỗ (Trái tiến, Phải lùi)
        setMotorSpeed(LEFT, turn_speed);
        setMotorSpeed(RIGHT, -turn_speed);

        // Đã xoay đủ góc (sai số nhỏ hơn ngưỡng) -> Thoát vòng lặp
        if (abs(e_turn) < TURN_THRESHOLD) break;
    }
    stopMotors(); // Dừng hẳn robot sau khi xoay xong
}
```

---

### 3.3. Bài toán 3: Canh giữa ô thông minh (Sử dụng Cảm biến tường)

Hiệu chỉnh sai số trượt bánh vi mô để tránh chạm vách.

* **Nguyên lý bù sai số cảm biến (thay cho $E_{\text{steer}}$):**
  * Có 2 tường: $E_{\text{wall}} = \text{Sensor Left} - \text{Sensor Right}$
  * Chỉ có tường trái: $E_{\text{wall}} = (\text{Sensor Left} - \text{Target Distance}) \times 2$
  * Chỉ có tường phải: $E_{\text{wall}} = (\text{Target Distance} - \text{Sensor Right}) \times 2$
  * Không có tường: $E_{\text{wall}} = \text{Left Ticks} - \text{Right Ticks}$

```c
// Đoạn logic này nằm trong hàm moveStraight
int e_wall = 0, e_wall_prev = 0;

if (hasWallLeft() && hasWallRight()) {
    e_wall = getLeftSensor() - getRightSensor();
} else if (hasWallLeft()) {
    e_wall = (getLeftSensor() - TARGET_WALL_DIST) * 2; 
} else if (hasWallRight()) {
    e_wall = (TARGET_WALL_DIST - getRightSensor()) * 2;
} else {
    e_wall = getLeftEncoder() - getRightEncoder(); 
}

int wall_correction = (Kp_wall * e_wall) + (Kd_wall * (e_wall - e_wall_prev));
e_wall_prev = e_wall;

setMotorSpeed(LEFT, base_speed - wall_correction);
setMotorSpeed(RIGHT, base_speed + wall_correction);
```

---

## 4. Bảng tổng hợp trạng thái điều khiển

| Hành động | Giá trị đích (Setpoint) | Cảm biến sơ cấp | Phản hồi thứ cấp (Sửa sai) |
| :--- | :--- | :--- | :--- |
| **Chạy thẳng 1 ô** | Ticks định mức 18cm | Encoder Trái + Phải | Cân bằng tích phân Ticks 2 bánh |
| **Xoay góc ($\pm90^\circ, 180^\circ$)** | Ticks quy đổi theo góc | Encoder (Đọc ngược dấu) | Vi phân vận tốc góc để giảm vọt lố |
| **Canh giữa dòng** | Điểm trung vị của lòng đường | Cảm biến khoảng cách IR | Thay đổi tỷ lệ tốc độ dựa trên độ lệch vách |

## 5. Hướng dẫn Tune PID


Quy trình chung cho mọi hành động là đặt các hệ số về 0, rồi tăng theo thứ tự: **$K_p \rightarrow K_d \rightarrow K_i$**.

* Gợi ý:
    *  Nên bắt đầu $K_p$ từ một số nguyên nhỏ (ví dụ 1.0 hoặc 0.1 tùy đơn vị). 
    *  $K_d$ thường lớn hơn $K_p$ nhiều lần (thử nghiệm ở mức $10 \times K_p$). 
    *  Khâu $K_i$ tích lũy rất nhanh nên bắt đầu từ một số cực nhỏ (ví dụ $0.001 \times K_p$).

---

##### Pha 1: Tune PID Xoay tại chỗ (Turn PID)
*Đây là hành động dễ tune nhất vì robot không bị ảnh hưởng bởi quán tính lao về phía trước.*

1. **Tìm $K_p$:** Tăng dần $K_p$ cho đến khi robot bắt đầu quay. Mục tiêu là khi truyền lệnh xoay $90^\circ$, robot quay nhanh, đạt gần đủ góc nhưng sẽ bị **vọt lố nhẹ (xoay quá $90^\circ$) rồi giật lại 1-2 nhịp**.
2. **Tìm $K_d$:** Giữ nguyên $K_p$, tăng mạnh $K_d$ (thường $K_d = 10 \times K_p$ đến $20 \times K_p$). Tăng cho đến khi robot quay một phát dứt khoát, **khựng lại ngay lập tức tại góc $90^\circ$** mà không bị lắc đuôi hay giật lùi.
3. **Tìm $K_i$:** Nếu robot dừng mượt nhưng góc quay luôn bị thiếu một chút (ví dụ chỉ được $88^\circ$ do ma sát lốp), tăng $K_i$ thật nhỏ để robot "nhích" nốt $2^\circ$ cuối. Nếu đã chuẩn, để $K_i = 0$.


---

##### Pha 2: Tune PID Đi thẳng thuần bằng Encoder (Straight PID)
*Mục đích là tìm bộ thông số giúp robot tịnh tiến ổn định theo trục dọc.*

**Tune $K_{p\_dist}$ và $K_{d\_dist}$ (PID khoảng cách tổng):** Cho robot chạy thẳng 1 ô (18cm). 
   * Tăng $K_{p\_dist}$ để robot lao đi nhanh và đạt gần đủ 18cm.
   * Tăng $K_{d\_dist}$ làm phanh để robot khựng lại chính xác ở vạch đích mà không bị dội ngược lại.
   * Nếu đi ổn định nhưng bị cộng dồn sai số và đi thiếu bước rõ rệt sau nhiều lần di chuyển, bắt đầu tăng $K_{i\_dist}$. Nếu dã chuẩn thì không cần đụng đến.
---

##### Pha 3: Tune PID Canh giữa bằng Cảm biến tường (Wall PID)
*Đặt robot vào đường thẳng có 2 bên vách.*

1. **Tune $K_{p\_wall}$:** Cho robot chạy thẳng với tốc độ cơ sở ổn định (đã tune ở Pha 2 - Straight PID). Lấy tay đẩy nhẹ robot lệch sang sát tường trái. 
   * Tăng dần $K_{p\_wall}$ cho đến khi robot tự động bẻ lái né tường trái để đi về lại giữa đường. 
   * Nếu $K_{p\_wall}$ quá cao, robot sẽ né tường trái quá mạnh rồi đâm sầm vào tường phải.
2. **Tune $K_{d\_wall}$:** Tăng $K_{d\_wall}$ để làm mượt quỹ đạo về giữa. Mục tiêu là khi robot bị lệch, nó sẽ **vuốt nhẹ một đường cong mượt mà để trở lại tâm ô**, thay vì chạy zigzag đập nhả liên tục giữa 2 bức tường.

---
