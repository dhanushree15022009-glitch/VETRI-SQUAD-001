# VETRI-SQUAD-001#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// ---------- OLED ----------
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// ---------- PINS ----------
#define ACS_PIN A0

#define RELAY1 D5
#define RELAY2 D6
#define RELAY3 D7
#define RELAY4 D8

#define BUZZER D0

// ---------- SETTINGS ----------
float peakLimit = 1.50;   // Ampere - change for your prototype

// ACS712 sensitivity
// 5A version  = 185 mV/A
// 20A version = 100 mV/A
// 30A version = 66 mV/A
float sensitivity = 0.185;

// ESP8266 ADC reference
float adcReference = 3.3;

// ACS712 zero-current voltage
float zeroVoltage = 2.5;


// ---------- SETUP ----------
void setup() {

  Serial.begin(115200);

  // OLED
  Wire.begin(D2, D1);

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("OLED NOT FOUND");
    while (true);
  }

  // Relay
  pinMode(RELAY1, OUTPUT);
  pinMode(RELAY2, OUTPUT);
  pinMode(RELAY3, OUTPUT);
  pinMode(RELAY4, OUTPUT);

  // Buzzer
  pinMode(BUZZER, OUTPUT);

  // Relay OFF
  digitalWrite(RELAY1, HIGH);
  digitalWrite(RELAY2, HIGH);
  digitalWrite(RELAY3, HIGH);
  digitalWrite(RELAY4, HIGH);

  digitalWrite(BUZZER, LOW);

  // Startup display
  display.clearDisplay();
  display.setTextColor(WHITE);

  display.setTextSize(2);
  display.setCursor(10, 5);
  display.println("PEAK");

  display.setCursor(10, 30);
  display.println("SHAVING");

  display.display();

  delay(2000);
}


// ---------- CURRENT READING ----------
float readCurrent() {

  long total = 0;

  // Take multiple samples
  for (int i = 0; i < 200; i++) {
    total += analogRead(ACS_PIN);
    delayMicroseconds(200);
  }

  float averageADC = total / 200.0;

  float voltage = (averageADC / 1023.0) * adcReference;

  float current = (voltage - zeroVoltage) / sensitivity;

  // Negative value -> make positive
  current = abs(current);

  return current;
}


// ---------- DISPLAY ----------
void showDisplay(float current, float power, String status) {

  display.clearDisplay();

  display.setTextColor(WHITE);

  display.setTextSize(1);
  display.setCursor(0, 0);
  display.println("PEAK ENERGY SHAVING");

  display.setTextSize(2);
  display.setCursor(0, 16);
  display.print("I:");
  display.print(current, 2);
  display.println(" A");

  display.setTextSize(1);
  display.setCursor(0, 40);
  display.print("Power: ");
  display.print(power, 1);
  display.println(" W");

  display.setCursor(0, 54);
  display.print(status);

  display.display();
}


// ---------- LOOP ----------
void loop() {

  float current = readCurrent();

  // Example prototype voltage
  float voltage = 12.0;

  float power = voltage * current;

  Serial.print("Current: ");
  Serial.print(current);
  Serial.print(" A   Power: ");
  Serial.print(power);
  Serial.println(" W");


  // ---------- PEAK DETECTED ----------
  if (current > peakLimit) {

    Serial.println("PEAK DETECTED!");
    Serial.println("Shaving load...");

    // Buzzer
    digitalWrite(BUZZER, HIGH);
    delay(200);
    digitalWrite(BUZZER, LOW);

    // Turn OFF non-essential loads
    digitalWrite(RELAY4, HIGH);
    delay(300);

    digitalWrite(RELAY3, HIGH);
    delay(300);

    showDisplay(current, power, "PEAK
