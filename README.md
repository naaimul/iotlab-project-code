#include <Servo.h> #include <SoftwareSerial.h>

// Servo and SIM800L setup Servo myservo; int pos = 0; SoftwareSerial sim800l(7, 6); // RX from SIM800L to pin 7, TX from Arduino to pin 6

int motor_speed = 150; #define pump 5 // Relay module

// Flame sensor pins #define RightSensor 7 #define FrontSensor 8

// Motor driver pins #define LM1 2 #define LM2 13 #define ENA 3 #define RM1 4 #define RM2 12 #define ENB 10

bool fireAlertSent = false;

void setup() { myservo.attach(11); myservo.write(0);

pinMode(LM1, OUTPUT); pinMode(LM2, OUTPUT); pinMode(RM1, OUTPUT); pinMode(RM2, OUTPUT); pinMode(ENA, OUTPUT); pinMode(ENB, OUTPUT);

pinMode(pump, OUTPUT); pinMode(RightSensor, INPUT); pinMode(FrontSensor, INPUT);

Serial.begin(9600); sim800l.begin(9600);

analogWrite(ENA, motor_speed); analogWrite(ENB, motor_speed); }

void loop() { int rightFlame = digitalRead(RightSensor); int frontFlame = digitalRead(FrontSensor);

Serial.print("Right: "); Serial.print(rightFlame); Serial.print(" | Front: "); Serial.println(frontFlame);

if (frontFlame == LOW) { moveForward(); fireAlertSent = false; } else if (rightFlame == LOW) { turnRight(); fireAlertSent = false; } else if (frontFlame == LOW && rightFlame == LOW) { moveForward(); fireAlertSent = false; } else if (rightFlame == LOW || frontFlame == LOW) { stopMotors(); digitalWrite(pump, LOW); // Turn ON water pump

if (!fireAlertSent) {
  sendSMS("01816872153", "Warning! Fire");
  Serial.println("SMS sent!");
  fireAlertSent = true;
}

for (int i = 0; i < 3; i++) {
  for (pos = 0; pos <= 125; pos++) {
    myservo.write(pos);
    delay(15);
  }
  for (pos = 125; pos >= 0; pos--) {
    myservo.write(pos);
    delay(15);
  }
}

digitalWrite(pump, HIGH);  // Turn off water pump
myservo.write(0);
} else { stopMotors(); fireAlertSent = false; digitalWrite(pump, HIGH); }

delay(200); }

// Movement Functions void moveForward() { digitalWrite(LM1, HIGH); digitalWrite(LM2, LOW); digitalWrite(RM1, HIGH); digitalWrite(RM2, LOW); }

void turnRight() { digitalWrite(LM1, HIGH); digitalWrite(LM2, LOW); digitalWrite(RM1, LOW); digitalWrite(RM2, LOW); }

void stopMotors() { digitalWrite(LM1, LOW); digitalWrite(LM2, LOW); digitalWrite(RM1, LOW); digitalWrite(RM2, LOW); }

// SMS Function void sendSMS(String phoneNumber, String message) { sim800l.println("AT+CMGF=1"); delay(500); sim800l.print("AT+CMGS=""); sim800l.print(phoneNumber); sim800l.println("""); delay(500); sim800l.print(message); delay(500); sim800l.write(26); // Ctrl+Z delay(5000); }
