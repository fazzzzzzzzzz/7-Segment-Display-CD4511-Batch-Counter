// Digit 1 (hundreds)
const int h1 = 2, h2 = 3, h3 = 4, h4 = 5;
// Digit 2 (tens) BCD inputs
const int t1 = 6, t2 = 7, t3 = 8, t4 = 9;
// Digit 3 (ones)
const int o1 = 10, o2 = 11, o3 = 12, o4 = 13;

// Buttons
const int decButtonPin = A0;
const int incButtonPin = A1;

// Potentiometer
const int potPin = A2;

int count = 0; // 0-999
int lastPotMapped = -1;

// debounce state
bool incLastReading = HIGH, decLastReading = HIGH;
bool incStableState  = HIGH, decStableState  = HIGH;
unsigned long incLastDebounceTime = 0, decLastDebounceTime = 0;
const unsigned long debounceDelay = 25; // ms

// Hold-to-repeat state
unsigned long incPressStart = 0, decPressStart = 0;
unsigned long incLastRepeat = 0, decLastRepeat = 0;
const unsigned long holdThreshold = 400; 
const unsigned long repeatInterval = 120; 

void setup() {
  pinMode(h1, OUTPUT); pinMode(h2, OUTPUT); pinMode(h3, OUTPUT); pinMode(h4, OUTPUT);
  pinMode(t1, OUTPUT); pinMode(t2, OUTPUT); pinMode(t3, OUTPUT); pinMode(t4, OUTPUT);
  pinMode(o1, OUTPUT); pinMode(o2, OUTPUT); pinMode(o3, OUTPUT); pinMode(o4, OUTPUT);

  pinMode(decButtonPin, INPUT_PULLUP);
  pinMode(incButtonPin, INPUT_PULLUP);

  Serial.begin(9600);
  updateDisplays();
}

void loop() {
  handlePotentiometer();
  handleButton(incButtonPin, incLastReading, incStableState, incLastDebounceTime,
               incPressStart, incLastRepeat, true);
  handleButton(decButtonPin, decLastReading, decStableState, decLastDebounceTime,
               decPressStart, decLastRepeat, false);
}

void handlePotentiometer() {
  int potValue = analogRead(potPin); // 0-1023
  int mapped = map(potValue, 0, 1023, 0, 999);

  if (abs(mapped - lastPotMapped) >= 2) {
    count = mapped;
    lastPotMapped = mapped;
    updateDisplays();
    Serial.println(count);
  }
}

void changeCount(bool isIncrement) {
  if (isIncrement) {
    if (count < 999) count++;
  } else {
    if (count > 0) count--;
  }
  lastPotMapped = map(analogRead(potPin), 0, 1023, 0, 999); // resync pot and it won't snap back
  updateDisplays();
  Serial.println(count);
}

void handleButton(int pin, bool &lastReading, bool &stableState,
                   unsigned long &lastDebounceTime,
                   unsigned long &pressStart, unsigned long &lastRepeat,
                   bool isIncrement) {
  bool reading = digitalRead(pin);

  if (reading != lastReading) {
    lastDebounceTime = millis();
  }

  if ((millis() - lastDebounceTime) > debounceDelay) {
    if (reading != stableState) {
      stableState = reading;
      if (stableState == LOW) {
        changeCount(isIncrement);
        pressStart = millis();
        lastRepeat = millis();
      }
    }
  }

  if (stableState == LOW) {
    unsigned long heldFor = millis() - pressStart;
    if (heldFor > holdThreshold) {
      if (millis() - lastRepeat > repeatInterval) {
        changeCount(isIncrement);
        lastRepeat = millis();
      }
    }
  }

  lastReading = reading;
}

void setBCD(int pinA, int pinB, int pinC, int pinD, int digit) {
  digitalWrite(pinA, digit & 0x01);
  digitalWrite(pinB, (digit >> 1) & 0x01);
  digitalWrite(pinC, (digit >> 2) & 0x01);
  digitalWrite(pinD, (digit >> 3) & 0x01);
}

void updateDisplays() {
  int hundreds = count / 100;
  int tens = (count / 10) % 10;
  int ones = count % 10;

  setBCD(h1, h2, h3, h4, hundreds);
  setBCD(t1, t2, t3, t4, tens);
  setBCD(o1, o2, o3, o4, ones);
} 
