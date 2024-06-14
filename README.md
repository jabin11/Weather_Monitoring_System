#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>
#include <WiFi.h>
#include <ThingSpeak.h>

#define DHTPIN 2        // Define the pin connected to the DHT sensor
#define DHTTYPE DHT11    // Define the type of DHT sensor
int gas, co2lvl;

const char* ssid = "Sabbir";
const char* password = "19122918";
const unsigned long CHANNEL_ID = 2537820;
const char* WRITE_API_KEY = "BVX3CE9PHFHNT6V7";

DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(0x27, 20, 4);  // Set the LCD I2C address
WiFiClient client;  // Initialize WiFi client

void setup() {
  Serial.begin(115200);
  dht.begin();
  lcd.init();
  lcd.backlight();

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  ThingSpeak.begin(client);  // Pass the client object to ThingSpeak
}

void loop() {
int gas = analogRead(35);
  co2lvl = gas;
  
  Serial.println(co2lvl);

  lcd.setCursor (0,0);
  lcd.print("CO2 :");
  lcd.setCursor(6,0);
  lcd.print(co2lvl);
  lcd.print("ppm");
  
  
  
  if((co2lvl >= 0)&&(co2lvl <= 650))
  {
    lcd.print("-Good");
  }
  else if((co2lvl >= 650)&&(co2lvl <= 1200))
  {
    lcd.print("-Bad");
  }
  else
  {
    lcd.print(" Danger");
  }
  delay(1000);
  
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();
  int rainSensorValue = analogRead(34); // Assuming rain sensor connected to GPIO34 (ADC1_6)

  lcd.clear();
  lcd.setCursor(0, 1);
  lcd.print("T: ");
  lcd.print(temperature);
  lcd.print("C");
  lcd.setCursor(0, 2);
  lcd.print("H: ");
  lcd.print(humidity);
  lcd.print("%");
  lcd.setCursor(0, 3);
  lcd.print("Rain: ");
  lcd.print(rainSensorValue);

  delay(1000);

  // Update ThingSpeak channel
  ThingSpeak.writeField(CHANNEL_ID, 1, temperature, WRITE_API_KEY);
  ThingSpeak.writeField(CHANNEL_ID, 2, humidity, WRITE_API_KEY);
  ThingSpeak.writeField(CHANNEL_ID, 3, rainSensorValue, WRITE_API_KEY);
  ThingSpeak.writeField(CHANNEL_ID, 4, co2lvl , WRITE_API_KEY);
    // ThingSpeak updates every 15 seconds
}
