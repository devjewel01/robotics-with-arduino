**Andriod Apps control Robot**
```cpp
// L298N Motor Driver Pins
#define LF 6     // Left Forward
#define LB 7     // Left Backward
#define RF 5     // Right Forward
#define RB 4     // Right Backward
#define ENA 9    // Enable A - Left Motor Speed
#define ENB 10   // Enable B - Right Motor Speed

// Additional Pin Definitions
#define FRONT_LIGHTS_PIN 11   
#define BACK_LIGHTS_PIN  12   
#define HORN_PIN        13    
#define EXTRA_PIN       8     

// Speed Settings
#define MAX_SPEED 255
#define TURN_SPEED_RATIO 0.8

// Variables
int currentSpeed = MAX_SPEED / 2;  // Default to 50% speed
bool frontLightsState = false;
bool backLightsState = false;
bool hornState = false;
bool extraState = false;

void setup() {
  Serial.begin(9600);    // HC-05 default speed
  
  // Initialize Motor Pins
  pinMode(LF, OUTPUT);
  pinMode(LB, OUTPUT);
  pinMode(RF, OUTPUT);
  pinMode(RB, OUTPUT);
  pinMode(ENA, OUTPUT);
  pinMode(ENB, OUTPUT);
  
  // Initialize additional pins
  pinMode(FRONT_LIGHTS_PIN, OUTPUT);
  pinMode(BACK_LIGHTS_PIN, OUTPUT);
  pinMode(HORN_PIN, OUTPUT);
  pinMode(EXTRA_PIN, OUTPUT);
}

void setSpeed(char speed_char) {
  if (speed_char >= '0' && speed_char <= '9') {
    currentSpeed = map(speed_char - '0', 0, 9, 0, MAX_SPEED);
  } else if (speed_char == 'q') {
    currentSpeed = MAX_SPEED;  // 100% speed
  }
}

void moveMotors(int leftSpeed, int rightSpeed) {
  // Apply current speed scaling
  leftSpeed = map(leftSpeed, -MAX_SPEED, MAX_SPEED, -currentSpeed, currentSpeed);
  rightSpeed = map(rightSpeed, -MAX_SPEED, MAX_SPEED, -currentSpeed, currentSpeed);
  
  // Left motor direction
  if (leftSpeed >= 0) {
    digitalWrite(LF, HIGH);
    digitalWrite(LB, LOW);
  } else {
    digitalWrite(LF, LOW);
    digitalWrite(LB, HIGH);
    leftSpeed = -leftSpeed;
  }
  
  // Right motor direction
  if (rightSpeed >= 0) {
    digitalWrite(RF, HIGH);
    digitalWrite(RB, LOW);
  } else {
    digitalWrite(RF, LOW);
    digitalWrite(RB, HIGH);
    rightSpeed = -rightSpeed;
  }
  
  // Apply PWM speed control
  analogWrite(ENA, leftSpeed);
  analogWrite(ENB, rightSpeed);
}

void executeCommand(char command) {
  int baseSpeed = currentSpeed;
  int turnSpeed = currentSpeed * TURN_SPEED_RATIO;
  
  switch (command) {
    // Movement commands
    case 'F': moveMotors(baseSpeed, baseSpeed); break;      // Forward
    case 'B': moveMotors(-baseSpeed, -baseSpeed); break;    // Backward
    case 'L': moveMotors(-turnSpeed, turnSpeed); break;     // Left
    case 'R': moveMotors(turnSpeed, -turnSpeed); break;     // Right
    case 'G': moveMotors(turnSpeed, baseSpeed); break;      // Forward Left
    case 'I': moveMotors(baseSpeed, turnSpeed); break;      // Forward Right
    case 'H': moveMotors(-turnSpeed, -baseSpeed); break;    // Back Left
    case 'J': moveMotors(-baseSpeed, -turnSpeed); break;    // Back Right
    case 'S':
    case 'D': moveMotors(0, 0); break;                      // Stop
    
    // Light controls
    case 'W': digitalWrite(FRONT_LIGHTS_PIN, HIGH); frontLightsState = true; break;
    case 'w': digitalWrite(FRONT_LIGHTS_PIN, LOW); frontLightsState = false; break;
    case 'U': digitalWrite(BACK_LIGHTS_PIN, HIGH); backLightsState = true; break;
    case 'u': digitalWrite(BACK_LIGHTS_PIN, LOW); backLightsState = false; break;
    
    // Horn control
    case 'V': digitalWrite(HORN_PIN, HIGH); hornState = true; break;
    case 'v': digitalWrite(HORN_PIN, LOW); hornState = false; break;
    
    // Extra feature control
    case 'X': digitalWrite(EXTRA_PIN, HIGH); extraState = true; break;
    case 'x': digitalWrite(EXTRA_PIN, LOW); extraState = false; break;
    
    // Speed control (0-9 and q for 100%)
    case '0'...'9':
    case 'q':
      setSpeed(command);
      break;
      
    default:
      // Invalid command - stop motors for safety
      moveMotors(0, 0);
      break;
  }
}

void loop() {
  if (Serial.available()) {
    char command = Serial.read();
    executeCommand(command);
  }
}
```
