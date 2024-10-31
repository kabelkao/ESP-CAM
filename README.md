# ESP-CAM


**V části ExternalTemperatureSensor.h je třeba doplnit:**
  // TMEP
  #include <WiFi.h>
  #include <HTTPClient.h>
  
  // tempMCU
  #include "esp_system.h"

**V části ExternalTemperatureSensor.cpp je třeba doplnit:**

  Do hlavičky:
  // TMEP, změnit DOMENA na svoji hodnotu dle TMEP
  String serverName = "http://DOMENA.tmep.cz/index.php?";

  Do: void ExternalSensor::ReadSensorData() {
  
    // Přidání čtení hodnoty teploty MCU
        float tempMCU = temperatureRead();

        /* ODESLANI DATA NA TMEP.cz */
        if(WiFi.status()== WL_CONNECTED){
            HTTPClient http;

            float teplota = DhtSensor.getTemperature();
            float vlhkost = DhtSensor.getHumidity();

            String serverPath = serverName + "teplota=" + teplota + "&vlhkost=" + vlhkost + "&tempMCU=" + tempMCU;
            
            // zacatek http spojeni
            http.begin(serverPath.c_str());
            
            // http get request
            int httpResponseCode = http.GET();
            
            if (httpResponseCode>0) {
              Serial.print("HTTP Response code: ");
              Serial.println(httpResponseCode);
              String payload = http.getString();
              Serial.println(payload);
            }
            else {
              Serial.print("Error code: ");
              Serial.println(httpResponseCode);
            }
            // Uvolneni
            http.end();
          }
          else {
            Serial.println("Wi-Fi odpojeno");
          }
          /*----------------------------*/

Celé se to pak odehraje za běhu hlavního kódu ESP32_PrusaConnectCam.ino který vše udělá v setup() a v loop() udělá jen reset.
Na TMEP se odesílají data proměnných: "teplota", "vlhkost" a "tempMCU". Tyto je třeba na TMEP nastavit.
