#include <ESP8266WiFi.h>
#include <espnow.h>
#include <LiquidCrystal.h>

// LCD wiring: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(D6, D5, D4, D3, D2, D1);

#define BUZZER D7   // You can change to a valid GPIO (example: GPIO15/D8 on NodeMCU)

// Must match TX struct
typedef struct struct_message {
  float temp1;
  float hum1;
  int gval1;
  int rval1;
  int vval1;
  int alert;
} struct_message;

struct_message incomingReadings;

void OnDataRecv(uint8_t *mac, uint8_t *incomingData, uint8_t len) {
  memcpy(&incomingReadings, incomingData, sizeof(incomingReadings));

  Serial.println("Data Received:");
  Serial.print("Temp: "); Serial.println(incomingReadings.temp1);
  Serial.print("Hum : "); Serial.println(incomingReadings.hum1);
  Serial.print("Gas : "); Serial.println(incomingReadings.gval1);
  Serial.print("Rain: "); Serial.println(incomingReadings.rval1);
  Serial.print("Vib : "); Serial.println(incomingReadings.vval1);
  Serial.print("Alert: "); Serial.println(incomingReadings.alert);

  // Display normal values
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("T:"); lcd.print(incomingReadings.temp1, 1);
  lcd.print(" H:"); lcd.print(incomingReadings.hum1, 0);

  lcd.setCursor(0, 1);
  lcd.print("G:"); lcd.print(incomingReadings.gval1);
  lcd.print(" R:"); lcd.print(incomingReadings.rval1);
  lcd.print(" V:"); lcd.print(incomingReadings.vval1);

  // Handle alerts
  if (incomingReadings.alert > 0) {
    delay(1000);
    lcd.clear();
    lcd.setCursor(0, 0);

    switch (incomingReadings.alert) {
      case 1: lcd.print("OVER TEMP/HUM");digitalWrite(BUZZER, 1); break;
      case 2: lcd.print("GAS LEAKED");digitalWrite(BUZZER, 1); break;
      case 3: lcd.print("RAIN DETECTED");digitalWrite(BUZZER, 1); break;
      case 4: lcd.print("VIB DETECTED");digitalWrite(BUZZER, 1); break;
    }

    // Buzzer ON for 2s
    
    delay(2000);
    digitalWrite(BUZZER, 0);
  }
}

void setup() {
  Serial.begin(115200);
  lcd.begin(16, 2);
  lcd.print("RX Init...");

  pinMode(BUZZER, OUTPUT);
  digitalWrite(BUZZER, 1);

  WiFi.mode(WIFI_STA);
  WiFi.disconnect();

  if (esp_now_init() != 0) {
    Serial.println("ESP-NOW Init Failed");
    lcd.setCursor(0, 1);
    lcd.print("ESP-NOW FAIL");
    return;
  }

  esp_now_set_self_role(ESP_NOW_ROLE_COMBO);
  esp_now_register_recv_cb(OnDataRecv);
}

void loop() {
  // Nothing required, callback handles data
}
