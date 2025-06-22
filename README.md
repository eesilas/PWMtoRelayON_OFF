const int pwmInputPin = 2;    // PWM input (D2, Interrupt 0)
const int relayPin = 7;       // Relay control (D7)
volatile unsigned long riseTime = 0;
volatile unsigned long pulseWidth = 0;

void setup() {
  Serial.begin(115200);
  pinMode(pwmInputPin, INPUT);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW); // Start with relay open
  
  // Attach interrupt for PWM detection
  attachInterrupt(digitalPinToInterrupt(pwmInputPin), risingEdge, RISING);
}

void loop() {
  if (pulseWidth > 0) { // Valid pulse detected
    Serial.print("Pulse Width: ");
    Serial.print(pulseWidth);
    Serial.println(" µs");

    // Check if pulse is in 1.1ms - 1.3ms range (Close Relay)
    if (pulseWidth >= 1100 && pulseWidth <= 1300) {
      digitalWrite(relayPin, HIGH); // Close relay
      Serial.println("Relay CLOSED");
    }
    // Check if pulse is in 1.7ms - 1.9ms range (Open Relay)
    else if (pulseWidth >= 1700 && pulseWidth <= 1900) {
      digitalWrite(relayPin, LOW); // Open relay
      Serial.println("Relay OPENED");
    }
    
    pulseWidth = 0; // Reset for next measurement
  }
  delay(20); // Small delay to avoid flooding serial monitor
}

// Interrupt Service Routine (ISR) for rising edge
void risingEdge() {
  riseTime = micros();
  detachInterrupt(digitalPinToInterrupt(pwmInputPin));
  attachInterrupt(digitalPinToInterrupt(pwmInputPin), fallingEdge, FALLING);
}

// ISR for falling edge
void fallingEdge() {
  pulseWidth = micros() - riseTime;
  detachInterrupt(digitalPinToInterrupt(pwmInputPin));
  attachInterrupt(digitalPinToInterrupt(pwmInputPin), risingEdge, RISING);
}
