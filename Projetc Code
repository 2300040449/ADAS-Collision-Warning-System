#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// ==========================================
// GPIO CONFIGURATION
// ==========================================
#define TRIG_PIN 5
#define ECHO_PIN 18
#define BUZZER_PIN 19

// LCD I2C configuration (Default ESP32 I2C pins: SDA=21, SCL=22)
// Change 0x27 to 0x3F if your LCD I2C address is different
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ==========================================
// CONSTANTS & THRESHOLDS
// ==========================================
const float WARNING_DISTANCE_CM = 20.0;
const unsigned long MEASUREMENT_TIMEOUT_US = 30000; // 30ms timeout (~500cm max distance)

// ==========================================
// SETUP
// ==========================================
void setup() {
  Serial.begin(115200);

  // Initialize GPIO pins
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  digitalWrite(TRIG_PIN, LOW);
  digitalWrite(BUZZER_PIN, LOW);

  // Initialize LCD
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("ADAS Collision");
  lcd.setCursor(0, 1);
  lcd.print("System Initializing");
  
  delay(2000);
  lcd.clear();
}

// ==========================================
// MAIN LOOP
// ==========================================
void loop() {
  float distance = readDistance();
  
  if (distance < 0) {
    handleSensorError();
  } else {
    updateDisplay(distance);
    handleWarning(distance);
  }

  // Small delay for stability
  delay(100);
}

// ==========================================
// SENSOR MEASUREMENT
// ==========================================
float readDistance() {
  // Ensure TRIG is low before pulsing
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  
  // Send 10us HIGH pulse to trigger
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  // Read echo pulse duration (with timeout to prevent freezing)
  long duration = pulseIn(ECHO_PIN, HIGH, MEASUREMENT_TIMEOUT_US);
  
  // If timeout occurred, pulseIn returns 0
  if (duration == 0) {
    return -1.0; 
  }
  
  // Calculate distance in cm (Speed of sound is ~343m/s or 0.0343 cm/us)
  float distanceCm = (duration * 0.0343) / 2.0;
  return distanceCm;
}

// ==========================================
// LCD FEEDBACK
// ==========================================
void updateDisplay(float distance) {
  lcd.setCursor(0, 0);
  lcd.print("Dist: ");
  lcd.print(distance, 1);
  lcd.print(" cm    "); // Trailing spaces to overwrite old characters
  
  lcd.setCursor(0, 1);
  if (distance <= WARNING_DISTANCE_CM) {
    lcd.print("WARNING!        ");
  } else {
    lcd.print("Status: SAFE    ");
  }
}

void handleSensorError() {
  lcd.setCursor(0, 0);
  lcd.print("Dist: OUT OF RNG");
  lcd.setCursor(0, 1);
  lcd.print("Status: SAFE    ");
  digitalWrite(BUZZER_PIN, LOW);
}

// ==========================================
// WARNING LOGIC
// ==========================================
void handleWarning(float distance) {
  if (distance <= WARNING_DISTANCE_CM) {
    digitalWrite(BUZZER_PIN, HIGH);
    Serial.println("WARNING: Obstacle detected!");
  } else {
    digitalWrite(BUZZER_PIN, LOW);
  }
}
