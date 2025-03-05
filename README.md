// Define pin connections
const int sensorPinA = 2;  // Pin for sensor A
const int sensorPinB = 3;  // Pin for sensor B
const int buzzerPin = 4;   // Pin for the buzzer

// Variables for timing
unsigned long previousMillis = 0;
const long interval = 1000;  // Interval at which to measure flow (1 second)

// Variables for flow rate calculation
volatile int pulseCountA = 0;
volatile int pulseCountB = 0;
float flowRateA = 0.0;
float flowRateB = 0.0;

// Define tolerance for flow rate comparison
const float tolerance = 0.005; // Tolerance in ml/s, reduced for more sensitivity

void setup() {
  Serial.begin(9600);
  pinMode(sensorPinA, INPUT);
  pinMode(sensorPinB, INPUT);
  pinMode(buzzerPin, OUTPUT);
  attachInterrupt(digitalPinToInterrupt(sensorPinA), countPulseA, RISING);
  attachInterrupt(digitalPinToInterrupt(sensorPinB), countPulseB, RISING);
}

void loop() {
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;

    // Calculate the flow rate in milliliters per second
    flowRateA = (pulseCountA / 4500.0) * 1000.0; // Adjusted conversion factor for ml/s
    flowRateB = (pulseCountB / 4500.0) * 1000.0; // Adjusted conversion factor for ml/s

    // Reset the pulse counts for the next interval
    pulseCountA = 0;
    pulseCountB = 0;

    // Output the flow rates to the serial monitor
    Serial.print("Flow Rate A: ");
    Serial.print(flowRateA);
    Serial.print(" ml/s | Flow Rate B: ");
    Serial.println(flowRateB);
    
    // Check if there is a significant discrepancy between the two flow rates
    if (abs(flowRateA - flowRateB) > tolerance) {
      Serial.println("Significant discrepancy detected! Activating buzzer.");
      digitalWrite(buzzerPin, HIGH); // Turn on the buzzer
    } else {
      digitalWrite(buzzerPin, LOW); // Turn off the buzzer
      Serial.println("Flow rates are within tolerance.");
    }
  }
}

// Interrupt service routines
void countPulseA() {
  pulseCountA++;
}

void countPulseB() {
  pulseCountB++;
}
