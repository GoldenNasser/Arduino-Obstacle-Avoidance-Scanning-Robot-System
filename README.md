# Arduino Obstacle Avoidance & Scanning Robot System

This repository contains a simulation project for a robot equipped with a smart obstacle avoidance system. The robot uses an Ultrasonic distance sensor mounted on a Servo Motor to continuously scan the area in front of it and makes movement decisions based on the detected distance. This project was built and simulated using Tinkercad.

## 🔗 Tinkercad Simulation Link
[https://www.tinkercad.com/things/4SPFT0xy3WH-arduino-obstacle-avoidance-amp-scanning-robot-system]

## 🎯 Task Objective
Program and build an electronic circuit that performs the following:
* **Continuous Scanning:** The servo motor rotates continuously to scan the area.
* **Normal Mode (Distance > 10 cm):** If the path is clear, the robot moves forward, and the servo scans the left side (between 90 degrees and 180 degrees).
* **Emergency Mode (Distance <= 10 cm):** If a close obstacle is detected, the motors stop immediately to avoid a collision, and the servo scans the right side (between 90 degrees and 0 degrees) searching for an alternative path.

## 🛠️ Hardware Components
* Arduino Uno
* L293D Motor Driver
* 4 DC Motors (representing the robot's wheels)
* Micro Servo Motor (to rotate the sensor)
* Ultrasonic Distance Sensor (HC-SR04)
* Breadboard and Jumper Wires
* 9V Battery (as an independent power source for the motors)

## 🚧 Challenges & Troubleshooting

While implementing this advanced project, I faced several programming and electronic challenges, and learned a lot from them:

1. **The `delay()` Function Issue and System Freezing:**
   * **Problem:** Initially, I used the `delay()` function to control the servo motor's speed. This temporarily "froze" the Arduino, causing the ultrasonic sensor to stop reading data continuously and delaying the robot's response to obstacles.
   * **Solution:** I removed the `delay()` function and used the `millis()` function to create a "Non-blocking Timer". This modification allowed the Arduino to smoothly update the servo's angle while simultaneously continuing to read the distance sensor without any interruption.

2. **Continuous Servo Rotation:**
   * **Problem:** I wanted the servo to rotate 360 degrees, but I discovered that standard servo motors are limited to only 180 degrees.
   * **Solution:** I programmed the servo to work in a back-and-forth "Scanning" system. I conditionally divided the scanning range: the left half (90 to 180) for a clear path, and the right half (90 to 0) when an obstacle is present.

3. **Dual Power Management:**
   * **Problem:** Powering 4 DC motors, a servo motor, and an ultrasonic sensor all from the Arduino's power (5V) led to high current draw, causing error messages and a simulated processor burnout.
   * **Solution:** I applied the concept of power isolation. I used an external 9V battery to supply the high-power motor pins (`Power 2`) on the L293D chip, while using the Arduino's power to run the chip's logic, the servo, and the sensor. I also ensured a **Common Ground** between the battery and the Arduino to maintain signal stability.

## 💻 Arduino Code
```cpp
#include <Servo.h>

// --- Motor Pins ---
const int IN1 = 4;
const int IN2 = 5;
const int IN3 = 6;
const int IN4 = 7;

// --- Distance Sensor Pins ---
const int trigPin = 8;
const int echoPin = 9;

// --- Servo ---
Servo neckServo;
const int servoPin = 10;

// Variables for smooth servo control
int servoAngle = 90;   // Current angle
int servoStep = 5;     // Movement step size
unsigned long lastServoMove = 0; 
const int servoDelay = 30; // Servo movement speed (milliseconds)

void setup() {
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  neckServo.attach(servoPin);
  neckServo.write(90); // Start at the center
  
  Serial.begin(9600);
}

void loop() {
  long distance = getDistance();
  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  unsigned long currentMillis = millis();

  // Update servo movement periodically 
  if (currentMillis - lastServoMove >= servoDelay) {
    lastServoMove = currentMillis;

    // --- If there is an obstacle (distance <= 10) ---
    if (distance > 0 && distance <= 10) {
      stopMotors(); // Stop motors
      
      // Servo movement between center (90) and right (0)
      servoAngle += servoStep;
      if (servoAngle >= 90) { // Reached center
        servoAngle = 90;
        servoStep = -5; // Reverse direction to right
      } else if (servoAngle <= 0) { // Reached max right
        servoAngle = 0;
        servoStep = 5;  // Reverse direction to center
      }
    } 
    // --- If the path is clear (distance > 10) ---
    else {
      moveForward(); // Move forward
      
      // Servo movement between center (90) and left (180)
      servoAngle += servoStep;
      if (servoAngle <= 90) { // Reached center
        servoAngle = 90;
        servoStep = 5;  // Reverse direction to left
      } else if (servoAngle >= 180) { // Reached max left
        servoAngle = 180;
        servoStep = -5; // Reverse direction to center
      }
    }
    
    neckServo.write(servoAngle); // Execute movement
  }
}

// Distance calculation function
long getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  long duration = pulseIn(echoPin, HIGH);
  return duration * 0.034 / 2;
}

// Movement functions
void moveForward() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}
