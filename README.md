# Arduino_code
Códigos e projeto com Arduino.

<h1>Criando um monitoramento de Temperatura e Humidade com Json</h1>

````
#include <DHT.h>
#include <SPI.h>
#include <Ethernet.h>
#include <ArduinoJson.h>

#define DHT_PIN 2   // Substitua pelo pino conectado ao DHT22
#define DHT_TYPE DHT22
#define NUM_READINGS 5

byte mac[] = {0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED};
IPAddress ip(192, 168, 1, 177);
EthernetServer server(80);

DHT dht(DHT_PIN, DHT_TYPE);

void setup() {
  Serial.begin(9600);
  Ethernet.begin(mac, ip);
  server.begin();
  dht.begin();
}

void loop() {
  EthernetClient client = server.available();

  if (client) {
    while (client.connected()) {
      float temperatureReadings[NUM_READINGS];
      float humidityReadings[NUM_READINGS];

      for (int i = 0; i < NUM_READINGS; i++) {
        temperatureReadings[i] = dht.readTemperature();
        humidityReadings[i] = dht.readHumidity();
        delay(2000);  // Intervalo de 2 segundos entre leituras
      }

      // Ordena as leituras em ordem crescente manualmente
      for (int i = 0; i < NUM_READINGS - 1; i++) {
        for (int j = i + 1; j < NUM_READINGS; j++) {
          if (temperatureReadings[i] > temperatureReadings[j]) {
            float temp = temperatureReadings[i];
            temperatureReadings[i] = temperatureReadings[j];
            temperatureReadings[j] = temp;
          }
          if (humidityReadings[i] > humidityReadings[j]) {
            float temp = humidityReadings[i];
            humidityReadings[i] = humidityReadings[j];
            humidityReadings[j] = temp;
          }
        }
      }

      // Calcula a média dos valores do meio
      float averageTemperature = 0;
      float averageHumidity = 0;
      for (int i = 1; i < NUM_READINGS - 1; i++) {
        averageTemperature += temperatureReadings[i];
        averageHumidity += humidityReadings[i];
      }
      averageTemperature /= (NUM_READINGS - 2);
      averageHumidity /= (NUM_READINGS - 2);

      // Cria o JSON de resposta com valores formatados
      StaticJsonBuffer<200> jsonBuffer;
      JsonObject& jsonDocument = jsonBuffer.createObject();
      jsonDocument["temperature"] = String(averageTemperature, 2);
      jsonDocument["humidity"] = String(averageHumidity, 2);

      String jsonString;
      jsonDocument.printTo(jsonString);

      client.println("HTTP/1.1 200 OK");
      client.println("Content-Type: application/json");
      client.println();
      client.println(jsonString);

      break;
    }
    client.stop();
  }
}
````
