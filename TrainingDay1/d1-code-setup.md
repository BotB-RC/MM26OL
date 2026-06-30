---
title: CODE SETUP

---

```cpp
/// Noodle code
/// Important Lib
#include <Arduino.h>
#include <Adafruit_VL53L0X.h>
#include <Adafruit_NeoPixel.h>
#include <Wire.h>
#include <PCF8574.h>

/// STL Lib
#include <utility>
#include <vector>

#define print_sensor sensor_print
#define print Serial.print
#define println Serial.println
using ui8 = uint8_t;
using ui16 = uint16_t;
using i32 = int32_t;
using ui32 = uint32_t;
using ui64 = uint64_t;

/// ================================================= CONFIG =================================================

// PWM
#define PWM_FREQ 20000
#define PWM_RES  8

#define PWM_CH_L 0
#define PWM_CH_R 1

// ENCODER PIN (B dùng trong ISR)
#define ENC_L_A 1
#define ENC_L_B 2

#define ENC_R_A 12
#define ENC_R_B 13

// MOTOR PIN
#define L_IN1 6
#define L_IN2 5
#define L_PWM 4

#define R_IN1 11
#define R_IN2 10
#define R_PWM 7

/// ================================================= ENCODER =================================================

// ticks
volatile int32_t ticksL = 0;
volatile int32_t ticksR = 0;

volatile i32 timeL;
volatile i32 timeR;
volatile i32 pastTimeL;
volatile i32 pastTimeR;
volatile i32 speedL;
volatile i32 speedR;

// nếu quay ngược thì đổi true/false
bool invertEncL = true;
bool invertEncR = true;

// ISR trái
void IRAM_ATTR isrEncL(){
  bool B = gpio_get_level((gpio_num_t)ENC_L_B);
  int step = (B ? +1 : -1);
  if(invertEncL) step = -step;
  ticksL += step;

  pastTimeL = timeL;
  timeL = micros();
}

// ISR phải
void IRAM_ATTR isrEncR(){
  bool B = gpio_get_level((gpio_num_t)ENC_R_B);
  int step = (B ? +1 : -1);
  if(invertEncR) step = -step;
  ticksR += step;

  pastTimeR = timeR;
  timeR = micros();
}

/// ================================================= MOTOR STRUCT =================================================

struct Motor {
  ui8 in1, in2, pwm;
  ui8 pwmCh;
  volatile int32_t* ticks;
  volatile i32* speed;

  void begin(){
    pinMode(in1, OUTPUT);
    pinMode(in2, OUTPUT);

    pinMode(pwm, OUTPUT);

    // encoder
    pinMode(ENC_L_A, INPUT_PULLUP);
    pinMode(ENC_L_B, INPUT_PULLUP);
    pinMode(ENC_R_A, INPUT_PULLUP);
    pinMode(ENC_R_B, INPUT_PULLUP);

    ledcSetup(pwmCh, PWM_FREQ, PWM_RES);
    ledcAttachPin(pwm, pwmCh);
  }


  void stop(){
    digitalWrite(in1, LOW);
    digitalWrite(in2, LOW);
    ledcWrite(pwmCh, 0);
  }

  int32_t getTicks(){
    return *ticks;
  }

  i32 getSpeed(){
    return *speed;
  }

  void resetTicks(){
    *ticks = 0;
  }

  void forwardPWM(i32 speed){
    if(speed > 255) speed = 255;
    if(speed < 0) speed = 0;

    digitalWrite(in1, LOW);
    digitalWrite(in2, HIGH);
    ledcWrite(pwmCh, speed);
  }

  void backwardPWM(i32 speed){
    if(speed > 255) speed = 255;
    if(speed < 0) speed = 0;

    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    ledcWrite(pwmCh, speed);
  }

  void moveToTick(int targetTick){
    const float Kp = 1;
    i32 error = targetTick - getTicks();

    if(error > 0){
      forwardPWM(Kp * error);
    }
    else if(error < 0){ieu khien toc do dong co: -255 -> 255
      backwardPWM(Kp * error);
    }
  }

  void movePWM(i32 speed){ /// dieu khien toc do dong co: -255 -> 255
    if(speed < 0){
      backwardPWM(abs(speed));
    }else{
      forwardPWM(speed);
    }
  }
};

/// ================================================= MOTOR INSTANCE =================================================
Motor motorL = {
  .in1 = L_IN2, // swaped to reverse H-bridge
  .in2 = L_IN1, // swaped to reverse H-bridge
  .pwm = L_PWM,
  .pwmCh = PWM_CH_L,
  .ticks = &ticksL,
  .speed = &speedL
};

Motor motorR = {
  .in1 = R_IN1,
  .in2 = R_IN2,
  .pwm = R_PWM,
  .pwmCh = PWM_CH_R,
  .ticks = &ticksR,
  .speed = &speedR
};



/// ================================================= ToF SENSOR =================================================
#define SDA_PIN 8
#define SCL_PIN 9

// PCF8574 address
#define PCF_ADDR 0x20

// XSHUT mapping trên PCF (P0 → P3)
#define XSHUT_L   1
#define XSHUT_M1  0
#define XSHUT_M2  3
#define XSHUT_R   2

PCF8574 pcf(PCF_ADDR);

// Sensors
Adafruit_VL53L0X sensorL;
Adafruit_VL53L0X sensorM1;
Adafruit_VL53L0X sensorM2;
Adafruit_VL53L0X sensorR;

// Distance
volatile ui16 distL = 0;
volatile uint16_t distM1 = 0;
volatile uint16_t distM2 = 0;
volatile uint16_t distR = 0;

// Fiter
uint16_t lastL1=0, lastL2=0;
uint16_t lastM11=0, lastM12=0;
uint16_t lastM21=0, lastM22=0;
uint16_t lastR1=0, lastR2=0;

uint16_t median3(uint16_t a, uint16_t b, uint16_t c){
  if (a > b) { uint16_t t=a; a=b; b=t; }
  if (b > c) { uint16_t t=b; b=c; c=t; }
  if (a > b) { uint16_t t=a; a=b; b=t; }
  return b;
}

// ================================================= PCF CONTROL =================================================
void xshut_write(uint8_t pin, bool level){
  pcf.write(pin, level ? HIGH : LOW);
}

// ================================================= INIT =================================================
bool init_sensor(Adafruit_VL53L0X &sensor, uint8_t xshutPin, uint8_t addr, const char* name){
  xshut_write(xshutPin, HIGH);
  vTaskDelay(50); // ⚠️ quan trọng với PCF

  while(!sensor.begin()){
    print("Retry ");
    println(name);
    vTaskDelay(50);
  }

  sensor.setAddress(addr);

  print("[OK] ");
  print(name);
  print(" @ 0x");
  println(addr, HEX);

  return true;
}

void sensor_init(){
  println("=== Sensor init (PCF8574) ===");

  Wire.begin(SDA_PIN, SCL_PIN);
  vTaskDelay(10);

  pcf.begin();
  vTaskDelay(10);

  // Tắt hết
  for(int i = 0; i < 4; i++){
    xshut_write(i, LOW);
  }
  vTaskDelay(100);

  // Init từng sensor
  init_sensor(sensorL,  XSHUT_L,  0x30, "L");
  init_sensor(sensorM1, XSHUT_M1, 0x31, "M1");
  init_sensor(sensorM2, XSHUT_M2, 0x32, "M2");
  init_sensor(sensorR,  XSHUT_R,  0x33, "R");

  // Timing budget
  const ui32 budget = 5000;
  sensorL.setMeasurementTimingBudgetMicroSeconds(budget);
  sensorM1.setMeasurementTimingBudgetMicroSeconds(budget);
  sensorM2.setMeasurementTimingBudgetMicroSeconds(budget);
  sensorR.setMeasurementTimingBudgetMicroSeconds(budget);

  vTaskDelay(10);

  // Continuous mode
  sensorL.startRangeContinuous(5);
  sensorM1.startRangeContinuous(5);
  sensorM2.startRangeContinuous(5);
  sensorR.startRangeContinuous(5);

  println("=== Sensor READY ===");
}

// ================================================= READ =================================================
void sensor_read(){
  const float aL  = 0.98, bL  = -5.0;
  const float aM1 = 0.99, bM1 = -3.0;
  const float aM2 = 1.00, bM2 =  0.0;
  const float aR  = 0.96, bR  = -12.0;
  // ===== L =====
  if(sensorL.isRangeComplete()){
    float raw = aL * sensorL.readRangeResult() + bL;

    uint16_t med = median3(lastL2, lastL1, (uint16_t)raw);
    distL = distL * 0.7 + med * 0.3;

    lastL2 = lastL1;
    lastL1 = raw;
  }

  // ===== M1 =====
  if(sensorM1.isRangeComplete()){
    float raw = aM1 * sensorM1.readRangeResult() + bM1;

    uint16_t med = median3(lastM12, lastM11, (uint16_t)raw);
    distM1 = distM1 * 0.7 + med * 0.3;

    lastM12 = lastM11;
    lastM11 = raw;
  }

  // ===== M2 =====
  if(sensorM2.isRangeComplete()){
    float raw = aM2 * sensorM2.readRangeResult() + bM2;

    uint16_t med = median3(lastM22, lastM21, (uint16_t)raw);
    distM2 = distM2 * 0.7 + med * 0.3;

    lastM22 = lastM21;
    lastM21 = raw;
  }

  // ===== R =====
  if(sensorR.isRangeComplete()){
    float raw = aR * sensorR.readRangeResult() + bR;

    uint16_t med = median3(lastR2, lastR1, (uint16_t)raw);
    distR = distR * 0.7 + med * 0.3;

    lastR2 = lastR1;
    lastR1 = raw;
  }
}

void read_sensor_raw(){
  const float aL  = 0.98, bL  = -5.0;
  const float aM1 = 0.99, bM1 = -3.0;
  const float aM2 = 1.00, bM2 =  0.0;
  const float aR  = 0.96, bR  = -12.0;
  // ===== L =====
  if(sensorL.isRangeComplete()){
    distL = (uint16_t)(aL * sensorL.readRangeResult() + bL);
  }

  // ===== M1 =====
  if(sensorM1.isRangeComplete()){
    distM1 = (uint16_t)(aM1 * sensorM1.readRangeResult() + bM1);
  }

  // ===== M2 =====
  if(sensorM2.isRangeComplete()){
    distM2 = (uint16_t)(aM2 * sensorM2.readRangeResult() + bM2);
  }

  // ===== R =====
  if(sensorR.isRangeComplete()){
    distR = (uint16_t)(aR * sensorR.readRangeResult() + bR);
  }
}

// ================= DEBUG =================
void sensor_print(){
  print(">SensorL:"); print(distL); print(",");
  print("SensorM1:"); print(distM1); print(",");
  print("SensorM2:"); print(distM2); print(",");
  print("SensorR:"); print(distR); println(",");
}

/// ================================================= ToF SENSOR END =================================================

void setup(){
  Serial.begin(115200);
  vTaskDelay(1000);

  /// Example
  // Khoi tao 2 motor
  motorL.begin();
  motorR.begin();
  attachInterrupt(digitalPinToInterrupt(ENC_L_A), isrEncL, RISING);
  attachInterrupt(digitalPinToInterrupt(ENC_R_A), isrEncR, RISING);

  // Khoi tao Sensor
  sensor_init();
  vTaskDelay(500);

  // vi du
  read_sensor_raw();
  print(">sensorL:"); print(distL); print(",");
  print("sensorM1:"); print(distM1); print(",");
  print("sensorM2:"); print(distM2); print(",");
  print("sensorR:"); print(distR); print(",");

  print("motorL_tick:"); print(motorL.getTicks()); print(",");
  print("motorR_tick:"); print(motorR.getTicks()); print(",");

  motorL.movePWM(255);
  motorR.movePWM(255);
  vTaskDelay(5000);
  motorL.movePWM(-255);
  motorR.movePWM(-255);
  vTaskDelay(5000);
  motorL.movePWM(0);
  motorR.movePWM(0);

}


void loop() {

}
```