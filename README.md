​Take Total Control: The Ultimate Bluetooth Arduino Controller
​Transform your smartphone into a powerful command center for your DIY robotics projects.
​Stop struggling with complex coding just to move your robot. Whether you are a student, a maker, or an robotics engineer, This app is designed to bridge the gap between your ideas and reality. Engineered for seamless performance and absolute flexibility, this is the only tool you need to bring your Arduino-based projects to life.
​Why choosing this app?
​100% Customizable Interface: Don't let your app dictate your workflow. You have full control to map commands, configure buttons, and tailor the interface to fit the specific needs of your robot, whether it’s a rover, a robotic arm, or a home automation system.
​Plug-and-Play Connectivity: Built on a robust Bluetooth protocol, our app ensures a stable, low-latency connection between your phone and your Arduino module (HC-05, HC-06, etc.).
​Lightweight & Efficient: We believe in simplicity. The app is optimized to run smoothly on any device, ensuring your focus remains on your project, not on app performance.
​Completely Free: No paywalls, no hidden features. We believe that innovation should be accessible to everyone.
​Built for the Maker Community
​We understand the challenges of robotics. That’s why we’ve built an app that adapts to you. With This app , you aren't just controlling a robot—you are customizing the entire experience to match your unique hardware setup.
​"The perfect companion for every Arduino enthusiast—simple, fast, and fully adaptable
 Here is the supported script For Arduino


{
 #include <SoftwareSerial.h>
#include <Servo.h>

// --- Pin Definitions ---
SoftwareSerial BT(2, 3); // Bluetooth RX, TX
const int trigPin = 12;
const int echoPin = 13;
const int in1 = 7; const int in2 = 8;
const int in3 = 9; const int in4 = 10;
const int enA = 5; const int enB = 6; // Must be connected because jumpers are removed
Servo neckServo;
const int servoPin = 11;

// --- Variables ---
char command;
const int motorSpeed = 255; // Locked at full speed
bool safeDriveMode = false;
bool autoPilotMode = false;
const int safeDistance = 15;

void setup() {
  Serial.begin(9600);
  BT.begin(9600);
  
  pinMode(in1, OUTPUT); pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT); pinMode(in4, OUTPUT);
  pinMode(enA, OUTPUT); pinMode(enB, OUTPUT);
  pinMode(trigPin, OUTPUT); pinMode(echoPin, INPUT);
  
  neckServo.attach(servoPin);
  neckServo.write(90); 
  stopRobot();
}

void loop() {
  if (BT.available()) {
    command = BT.read();
    
    // --- Movement Commands ---
    if (command == 'F') { safeDriveMode = false; autoPilotMode = false; moveForward(); }
    else if (command == 'B') { safeDriveMode = false; autoPilotMode = false; moveBackward(); }
    else if (command == 'L') { safeDriveMode = false; autoPilotMode = false; turnLeft(); }
    else if (command == 'R') { safeDriveMode = false; autoPilotMode = false; turnRight(); }
    else if (command == 'S') { safeDriveMode = false; autoPilotMode = false; stopRobot(); }
    
    // --- Manual Servo ---
    else if (command == 'X') { safeDriveMode = false; autoPilotMode = false; neckServo.write(180); }
    else if (command == 'Y') { safeDriveMode = false; autoPilotMode = false; neckServo.write(0); }
    
    // --- Modes ---
    else if (command == 'A') { autoPilotMode = false; safeDriveMode = true; neckServo.write(90); }
    else if (command == 'P') { safeDriveMode = false; autoPilotMode = true; neckServo.write(90); }
  }

  // --- Logic for Modes ---
  if (safeDriveMode) { runSafeDrive(); }
  if (autoPilotMode) { runAutoPilot(); }
}

// --- Helper Functions ---

void setSpeed() {
  // Always set to full speed since we removed the slider
  analogWrite(enA, motorSpeed);
  analogWrite(enB, motorSpeed);
}

void moveForward() { setSpeed(); digitalWrite(in1, HIGH); digitalWrite(in2, LOW); digitalWrite(in3, HIGH); digitalWrite(in4, LOW); }
void moveBackward() { setSpeed(); digitalWrite(in1, LOW); digitalWrite(in2, HIGH); digitalWrite(in3, LOW); digitalWrite(in4, HIGH); }
void turnLeft() { setSpeed(); digitalWrite(in1, LOW); digitalWrite(in2, HIGH); digitalWrite(in3, HIGH); digitalWrite(in4, LOW); }
void turnRight() { setSpeed(); digitalWrite(in1, HIGH); digitalWrite(in2, LOW); digitalWrite(in3, LOW); digitalWrite(in4, HIGH); }
void stopRobot() { analogWrite(enA, 0); analogWrite(enB, 0); digitalWrite(in1, LOW); digitalWrite(in2, LOW); digitalWrite(in3, LOW); digitalWrite(in4, LOW); }

int getDistance() {
  digitalWrite(trigPin, LOW); delayMicroseconds(2);
  digitalWrite(trigPin, HIGH); delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH, 30000);
  return duration * 0.034 / 2;
}

void runSafeDrive() {
  if (getDistance() > safeDistance || getDistance() == 0) { moveForward(); }
  else { stopRobot(); }
}

void runAutoPilot() {
  if (getDistance() > safeDistance || getDistance() == 0) { moveForward(); }
  else {
    stopRobot(); delay(300);
    neckServo.write(0); delay(500); int dR = getDistance();
    neckServo.write(180); delay(500); int dL = getDistance();
    neckServo.write(90); delay(300);
    if (dR > dL) { turnRight(); delay(400); }
    else { turnLeft(); delay(400); }
    stopRobot();
  }
}
}
