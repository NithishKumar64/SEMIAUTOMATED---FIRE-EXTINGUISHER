# SEMIAUTOMATED---FIRE-EXTINGUISHER
#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include <Servo.h>

// Libraries for SMS using Twilio
#include <HTTPClient.h>
#include <HTTPClient.h>
#include <HTTPClient.h>

char auth[] = "YourAuthToken"; // You should get Auth Token in the Blynk App.
char ssid[] = "YourNetworkName"; // Your WiFi SSID
char pass[] = "YourPassword"; // Your WiFi Password

// Pins for Servos
#define SERVO1_PIN 14 // Pin for Servo 1
#define SERVO2_PIN 12 // Pin for Servo 2

// Pin for LED
#define LED_PIN 2 // Pin for LED

// Pin for Flame Sensor
#define FLAME_SENSOR_PIN 27

// Pin for Smoke Sensor
#define SMOKE_SENSOR_PIN 26

Servo servo1;
Servo servo2;

// Flame and Smoke sensor variables
int flameSensorValue = 0;
int smokeSensorValue = 0;

void setup()
{
  Serial.begin(9600);
  Blynk.begin(auth, ssid, pass);

  // Initialize servo motors
  servo1.attach(SERVO1_PIN);
  servo2.attach(SERVO2_PIN);

  // Initialize LED pin
  pinMode(LED_PIN, OUTPUT);

  // Initialize flame sensor pin
  pinMode(FLAME_SENSOR_PIN, INPUT);

  // Initialize smoke sensor pin
  pinMode(SMOKE_SENSOR_PIN, INPUT);
}

void loop()
{
  Blynk.run();

  // Read sensor values
  flameSensorValue = digitalRead(FLAME_SENSOR_PIN);
  smokeSensorValue = digitalRead(SMOKE_SENSOR_PIN);

  // Check if flame is detected
  if (flameSensorValue == HIGH)
  {
    // Turn on LED
    digitalWrite(LED_PIN, HIGH);

    // Send SMS using Twilio
    sendSMS("Fire Detected!");
  }
  else
  {
    // Turn off LED
    digitalWrite(LED_PIN, LOW);
  }
}

// Function to control Servo 1
BLYNK_WRITE(V1)
{
  int pos = param.asInt();
  servo1.write(pos);
}

// Function to control Servo 2
BLYNK_WRITE(V2)
{
  int pos = param.asInt();
  servo2.write(pos);
}

// Function to send SMS using Twilio
void sendSMS(String message)
{
  HTTPClient http;

  // Your Twilio Account SID and Auth Token
  String accountSID = "YourTwilioAccountSID";
  String authToken = "YourTwilioAuthToken";

  // Your Twilio phone number
  String fromNumber = "+1234567890";

  // Recipient phone number
  String toNumber = "+1234567890";

  // Twilio API URL
  String url = "https://api.twilio.com/2010-04-01/Accounts/" + accountSID + "/Messages.json";

  // Twilio message body
  String body = "To=" + toNumber + "&From=" + fromNumber + "&Body=" + message;

  http.begin(url.c_str());
  http.setAuthorization(accountSID.c_str(), authToken.c_str());
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");

  int httpResponseCode = http.POST(body);

  if (httpResponseCode > 0)
  {
    String response = http.getString();
    Serial.println(httpResponseCode);
    Serial.println(response);
  }
  else
  {
    Serial.println("Error on sending message.");
  }

  http.end();
}
