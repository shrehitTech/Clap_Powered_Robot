# Clap Robot

## Overview
This project is an Arduino-based robotics system designed to move a robot by listening to claps autonomously.   
I built this project to understand how sensors, motors, and control logic work together in real-world robotics applications.     

## Motivation
I am interested in robotics and engineering. This project helped me improve my understanding of embedded systems, problem-solving, and hands-on design. 
It also helped me learn how sound could be used to power a vehicle and how robotics have come so far.

## Hardware Used
- Arduino Uno / Nano
- Motor Driver (L298N / L293D)
- DC Motors
- Sensors (Sound Sensor)
- Chassis & Wheels
- Battery Pack
- Jumper Wires
- LED & Resistor

## Software & Tools
- Arduino IDE 
- Arduino C++ 
- GitHub 

## How It Works
1. Sound Sensor listens for the count of claps and the number of claps it heard it gives the information to the Arduino.   
2. Arduino processes the sensor data.   
3. If it heard 1 clap then it would move forward, 2 clap wil move backward, 3 clap would turn the bot left and 4 clap would turn the bot right.
4. Each clap powers the led so we can visually see how many claps it heard.
5. The robot performs its task from the instruction recieved from the Arduino and moves accordingly.  

## Challenges & Solutions
- **Problem:**  The Sound Sensor can't hear claps.          
  **Solution:** Adjust the potentiometer (blue color knob) to increase its sensitivity and check all the connections too.
- **Problem:**  Robot responds to multiple claps at once.       
  **Solution:** Implemented debouncing in code by ignoring additional signals for a short time after detecting a clap.   

## Improvements & Future Work
- Add speed control using PWM 
- Improve accuracy and responsiveness 
- Add more sensors or smarter logic
- Add a speak element

## Media
<img width="298" height="346" alt="image" src="https://github.com/user-attachments/assets/d50f3471-2614-4450-bb20-104f76015e89" />
<img width="257" height="332" alt="image" src="https://github.com/user-attachments/assets/451691d0-1ea7-429a-a580-f6847ea9d1a8" />
<img width="286" height="516" alt="image" src="https://github.com/user-attachments/assets/49651504-7611-49d0-b66a-d0d0ac242ca5" />
<img width="87" height="63" alt="image" src="https://github.com/user-attachments/assets/ded2312c-e055-4fa6-9432-dd3b4ea8c793" />


## What I Learned
- Sensor integration
- Understanding structured Arduino code
- Debugging hardware and software
- How can a bot be powered by claps.
- How to power a motor with motors driver.

## Author
Shrehit Dhiman        
Student interested in robotics and engineering

## Code
```cpp
// ---------------- Pin Definitions ----------------
const int soundSensor = 2;  // Digital output from CZN-153
const int IN1 = 8;
const int IN2 = 9;
const int IN3 = 10;
const int IN4 = 11;
const int EN1 = 5;
const int EN2 = 6;
const int ledPin = 13;      // Optional LED for clap feedback

// ---------------- Timing & Clap Variables ----------------
unsigned long lastClapTime = 0;
const unsigned long debounceDelay = 250; // Minimum time between claps (ms)
const unsigned long clapWindow = 1000;   // Time to wait after last clap to execute action (ms)
int clapCount = 0;

int lastSensorState = LOW;

// ---------------- Motor Timing ----------------
unsigned long actionStartTime = 0;
const unsigned long moveDuration = 1000; // Motor movement duration (ms)

// ---------------- Setup ----------------
void setup() {
  pinMode(soundSensor, INPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(EN1, OUTPUT);
  pinMode(EN2, OUTPUT);
  pinMode(ledPin, OUTPUT);
  Serial.begin(9600);
}

// ---------------- Loop ----------------
void loop() {
  int currentState = digitalRead(soundSensor); // converts the output from the sensor into int so it could be used later.

  // Rising edge detection + debounce
  if (currentState == HIGH && lastSensorState == LOW && (millis() - lastClapTime > debounceDelay)) {
    clapCount++;
    lastClapTime = millis();

    // Blink LED for feedback
    digitalWrite(ledPin, HIGH);
    delay(100);
    digitalWrite(ledPin, LOW);

    Serial.print("Clap detected: ");
    Serial.println(clapCount);
  }

  lastSensorState = currentState;

  // If enough time has passed since last clap, execute action
  if ((millis() - lastClapTime > clapWindow) && clapCount > 0) {
    performAction(clapCount);
    clapCount = 0;
  }

  // Stop motors after moveDuration
  if (actionStartTime > 0 && (millis() - actionStartTime > moveDuration)) {
    stopMotors();
    actionStartTime = 0;
  }
}

// ---------------- Actions ----------------
void performAction(int claps) {
  switch (claps) {
    case 1:
      moveForward();
      Serial.println("Moving Forward");
      break;
    case 2:
      moveBackward();
      Serial.println("Moving Backward");
      break;
    case 3:
      turnLeft();
      Serial.println("Turning Left");
      break;
    case 4:
      turnRight();
      Serial.println("Turning Right");
      break;
    default:
      stopMotors();
      Serial.println("Stopping");
      return;
  }
  actionStartTime = millis(); // Start motor timer
}

// ---------------- Motor Functions ----------------
//IN1 = Right Motor Forward Pin
//IN2 = Right Motor Backward Pin 
//IN3 = Left Motor Backward Pin
//IN4 = Left Motor Forward Pin
//ENA = Used to Power the Right Motor Pins
//ENB = Used to Power the Left Motor Pins
void moveForward() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(EN1, 128);
  analogWrite(EN2, 128);
}

void moveBackward() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(EN1, 255);
  analogWrite(EN2, 255);
}

void turnLeft() {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(EN1, 255);
  analogWrite(EN2, 255);
}

void turnRight() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(EN1, 255);
  analogWrite(EN2, 255);
}

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(EN1, 0);
  analogWrite(EN2, 0);
}

