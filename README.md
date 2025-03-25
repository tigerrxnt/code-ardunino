# code-ardunino

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

// WiFi Credentials
const char* ssid = "YOUR_SSID";           // Replace with your WiFi SSID
const char* password = "YOUR_PASSWORD";   // Replace with your WiFi Password

// LINE Notify Token
const String LINE_TOKEN = "YOUR_LINE_TOKEN";  // Replace with your LINE Notify Token

int speedSensorPin_1 = D2;  // GPIO 4 on Wemos D1 R1
int speedSensorPin_2 = D3;  // GPIO 0
int speedSensorPin_3 = D4;  // GPIO 2
int speedSensorPin_4 = D5;  // GPIO 14

int statusSpeed_1 = 0;
int statusSpeed_2 = 0;
int statusSpeed_3 = 0;
int statusSpeed_4 = 0;

int coin_1 = 0;
int coin_2 = 0;
int coin_5 = 0;
int coin_10 = 0;
int coin_total = 0;  // Variable to store total amount of coins

int previous_coin_total = 0;  // To track previous total amount

int oneClink_1 = 0;
int oneClink_2 = 0;
int oneClink_3 = 0;
int oneClink_4 = 0;

void setup() {
  Serial.begin(9600);
  
  // Initialize LCD
  lcd.begin();
  lcd.print("Hello");
  lcd.setCursor(0, 1);
  lcd.print("Micro Projects th");
  delay(1000);
  lcd.clear();

  // Initialize pins
  pinMode(speedSensorPin_1, INPUT);
  pinMode(speedSensorPin_2, INPUT);
  pinMode(speedSensorPin_3, INPUT);
  pinMode(speedSensorPin_4, INPUT);

  // Connect to WiFi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi!");
}

void loop() {
  statusSpeed_1 = digitalRead(speedSensorPin_1);
  statusSpeed_2 = digitalRead(speedSensorPin_2);
  statusSpeed_3 = digitalRead(speedSensorPin_3);
  statusSpeed_4 = digitalRead(speedSensorPin_4);

  bool coinAdded = false;  // Track if any coin is added

  // Check sensor 1
  if (statusSpeed_1 == 1) {
    if (oneClink_1 == 0) {
      oneClink_1 = 1;
      coin_1++;
      coinAdded = true;
    }
  } else {
    oneClink_1 = 0;
  }

  // Check sensor 2
  if (statusSpeed_2 == 1) {
    if (oneClink_2 == 0) {
      oneClink_2 = 1;
      coin_2++;
      coinAdded = true;
    }
  } else {
    oneClink_2 = 0;
  }

  // Check sensor 3
  if (statusSpeed_3 == 1) {
    if (oneClink_3 == 0) {
      oneClink_3 = 1;
      coin_5++;
      coinAdded = true;
    }
  } else {
    oneClink_3 = 0;
  }

  // Check sensor 4
  if (statusSpeed_4 == 1) {
    if (oneClink_4 == 0) {
      oneClink_4 = 1;
      coin_10++;
      coinAdded = true;
    }
  } else {
    oneClink_4 = 0;
  }

  // Calculate total coins
  coin_total = coin_1 + coin_2 + coin_5 + coin_10;

  // If any coin was added and total has changed, send notification
  if (coinAdded && coin_total != previous_coin_total) {
    sendLineNotify("Coin total updated: " + String(coin_total) + 
                   " (c_1: " + String(coin_1) + 
                   ", c_2: " + String(coin_2) + 
                   ", c_5: " + String(coin_5) + 
                   ", c_10: " + String(coin_10) + ")");
    
    previous_coin_total = coin_total;  // Update previous total
  }

  // Update LCD display
  lcd.setCursor(0, 0);
  lcd.print("c_1 :");
  lcd.print(coin_1);
  lcd.setCursor(9, 0);
  lcd.print("c_2: ");
  lcd.print(coin_2);

  lcd.setCursor(0, 1);
  lcd.print("c_5: ");
  lcd.print(coin_5);
  lcd.setCursor(9, 1);
  lcd.print("c_10: ");
  lcd.print(coin_10);

  lcd.setCursor(0, 2);
  lcd.print("Total: ");
  lcd.print(coin_total);

  delay(1000);
}

// Function to send LINE Notify message
void sendLineNotify(String message) {
  WiFiClientSecure client;
  if (!client.connect("notify-api.line.me", 443)) {
    Serial.println("Connection failed");
    return;
  }

  String request = String("POST /api/notify HTTP/1.1\r\n") +
                   "Host: notify-api.line.me\r\n" +
                   "Authorization: Bearer " + LINE_TOKEN + "\r\n" +
                   "Content-Type: application/x-www-form-urlencoded\r\n" +
                   "Content-Length: " + String(String("message=").length() + message.length()) + "\r\n\r\n" +
                   "message=" + message;

  client.print(request);

  Serial.println("Message sent: " + message);
}
