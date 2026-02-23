code:
   // Pin Definitions
int redPins[] = {2, 5, 8, 11};
int yellowPins[] = {3, 6, 9, 12};
int greenPins[] = {4, 7, 10, 13};
int trigPins[] = {22, 24, 26, 28};
int echoPins[] = {23, 25, 27, 29};

// Timing Constraints
const unsigned long MAX_GREEN = 120000; // 120 Seconds Max
const unsigned long EXTENSION = 5000;   // 5 Seconds per car
const unsigned long YELLOW_TIME = 3000; // 3 Seconds Yellow

void setup() {
  for (int i = 0; i < 4; i++) {
    pinMode(redPins[i], OUTPUT);
    pinMode(yellowPins[i], OUTPUT);
    pinMode(greenPins[i], OUTPUT);
    pinMode(trigPins[i], OUTPUT);
    pinMode(echoPins[i], INPUT);
    digitalWrite(redPins[i], HIGH); // All roads start at Red
  }
  Serial.begin(9600);
}

long getDistance(int index) {
  digitalWrite(trigPins[index], LOW);
  delayMicroseconds(2);
  digitalWrite(trigPins[index], HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPins[index], LOW);
  long duration = pulseIn(echoPins[index], HIGH, 30000); // 30ms timeout
  return duration * 0.034 / 2;
}

void handleTraffic(int lane) {
  unsigned long startTime = millis();
  unsigned long clearTime = millis() + EXTENSION; // Initial 5s "window"

  // Turn Light Green
  digitalWrite(redPins[lane], LOW);
  digitalWrite(greenPins[lane], HIGH);
  Serial.print("Road "); Serial.print(lane + 1); Serial.println(" is GREEN.");

  // Impulse Loop
  while (millis() - startTime < MAX_GREEN) {
    long dist = getDistance(lane);
    
    // Check for "Impulse" (Car detected)
    if (dist > 0 && dist < 15) { 
      clearTime = millis() + EXTENSION; // Reset the gap timer
      Serial.println("Impulse detected: Extending Green...");
    }

    // GAP-OUT: If no cars seen within the extension window
    if (millis() > clearTime) {
      Serial.println("Gap detected: Ending early.");
      break; 
    }
    delay(100); 
  }

  // Yellow Transition
  digitalWrite(greenPins[lane], LOW);
  digitalWrite(yellowPins[lane], HIGH);
  delay(YELLOW_TIME);
  digitalWrite(yellowPins[lane], LOW);
  digitalWrite(redPins[lane], HIGH);
}

void loop() {
  for (int i = 0; i < 4; i++) {
    long checkDist = getDistance(i);
    // If a car is waiting at the junction
    if (checkDist > 0 && checkDist < 15) {
      handleTraffic(i);
    }
  }
}
