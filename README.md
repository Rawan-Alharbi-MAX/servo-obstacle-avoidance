## مشروع التحكم بمحرك DC وسيرفو باستخدام شيلد L293D وحساس Ultrasonic

---

### الوصف:
هذا المشروع يستخدم حساس Ultrasonic لقياس المسافة أمام الروبوت.  
عندما يكتشف الحساس عائقاً على بعد 10 سم أو أقل:  
- يتوقف محرك DC ثم يعكس اتجاه حركته للخلف لمدة ثانيتين.  
- محرك السيرفو يتحرك فقط إلى اليسار (زاوية 90 درجة جهة اليسار، على سبيل المثال زاوية 180 درجة في الكود) لتغيير اتجاه الروبوت.  
- بعد تجاوز العائق، يعود السيرفو للوضع المستقيم (90 درجة) والمحرك يستمر بالسير للأمام.  
- إذا لم يكن هناك عائق، المحرك يستمر في الحركة للأمام، والسيرفو في وضع 90 درجة.

---

### المكونات المستخدمة:

- Arduino Uno  
- حساس Ultrasonic (3 أطراف: Power, Ground, Signal)  
- محرك DC  
- محرك سيرفو (microServo)  
- شيلد L293D  
- بطارية 9V  

---

### التوصيلات:

| المكون          | الطرف في Arduino/Shield            | التوصيل              |
|-----------------|----------------------------------|----------------------|
| Ultrasonic Sensor | Power                           | 5V                   |
|                 | Ground                           | GND                  |
|                 | Signal                          | Arduino Pin 9        |
| Servo Motor     | VCC                             | 5V                   |
|                 | GND                             | GND                  |
|                 | Signal                          | Arduino Pin 6        |
| L293D Shield    | EN1                             | Arduino Pin 5 (PWM)   |
|                 | IN1                             | Arduino Pin 3        |
|                 | IN2                             | Arduino Pin 4        |
| DC Motor        | M1 Outputs (OUT1 & OUT2)         | محرك DC              |
| Battery 9V      | + (Power1 & Power2 على الشيلد)  | موجب البطارية         |
|                 | - (GND)                        | سالب البطارية مرتبط مع GND الأردوينو والشيلد |

---

### ملاحظات الطاقة:

- بطارية 9V متصلة بمداخل الطاقة الخاصة بشيلد L293D لتزويد المحرك DC بالطاقة المناسبة.  
- سالب البطارية (GND) مشترك مع أردوينو وشيلد لضمان إشارة أرض موحدة.

---

### الكود البرمجي:

```c++
#include <Servo.h>

Servo myservo;

const int enA = 5;  // PWM سرعة محرك DC
const int in1 = 3;
const int in2 = 4;

const int sensorPin = 9;
const int servoPin = 6;

long duration;
int distance;

void setup() {
  Serial.begin(9600);

  // إعداد محرك DC
  pinMode(enA, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);

  // إعداد حساس Ultrasonic
  pinMode(sensorPin, INPUT);

  // إعداد السيرفو
  myservo.attach(servoPin);
  myservo.write(90);  // الوضع المستقيم (منتصف زاوية السيرفو)

  // تشغيل محرك DC للأمام
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  analogWrite(enA, 150);
}

void loop() {
  duration = pulseIn(sensorPin, HIGH);
  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  if (distance <= 10 && distance > 0) {
    // توقف محرك DC
    analogWrite(enA, 0);

    // غير اتجاه محرك DC للخلف
    digitalWrite(in1, LOW);
    digitalWrite(in2, HIGH);
    analogWrite(enA, 150);

    // تحرك السيرفو إلى اليسار (زاوية 180 درجة)
    myservo.write(180);

    delay(2000);

    // رجع السيرفو للوضع المستقيم (90 درجة)
    myservo.write(90);

    // رجع المحرك للسير للأمام
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    analogWrite(enA, 150);
  }
  else {
    // استمر للأمام
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
    analogWrite(enA, 150);

    myservo.write(90);
  }

  delay(200);
}

