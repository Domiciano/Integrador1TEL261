```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "PUBLICA";
const char* password = "";

//Capa de aplicación
String url = "https://facelogprueba.firebaseio.com/data.json";

void POSTRequest(){
  HTTPClient http;
  http.begin(url.c_str()); //TCP handshake
  int httpResponseCode = http.POST("{\"message\":\"Hello World from Domi\"}"); // Http request
  Serial.println(httpResponseCode);
  if(httpResponseCode == 200){
    String responseBody = http.getString();
    Serial.println(responseBody);
  }else{
      Serial.println("Error on HTTP request");
  }
}

void GETRequest(){
    HTTPClient http;
    http.begin(url.c_str()); //TCP handshake
    int httpResponseCode = http.GET(); // Http request
    Serial.println(httpResponseCode);
    if(httpResponseCode == 200){
      String responseBody = http.getString();
      Serial.println(responseBody);
    }else{
      Serial.println("Error on HTTP request");
    }
}

void connectToWiFi(){
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password); //Intento para conectarse al WiFi
  Serial.print("Connecting to WiFi ..");

  //While
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }

  Serial.println("Connected!!");
  Serial.println(WiFi.localIP());
}

void setup() {
  Serial.begin(115200);
}

void loop() {
  
}

void serialEvent() {
  if (Serial.available() > 0) {
    String data = Serial.readStringUntil('\n');
    Serial.println(data);
    if(data == "wifi"){
      connectToWiFi();
    }else if(data == "get"){
      GETRequest();
    }else if(data == "post"){
      POSTRequest();
    }
  }
}
```
