```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <Arduino_JSON.h>

const char* ssid = "PUBLICA";
const char* password = "";
//const char* ssid = "LABREDES";
//const char* password = "F0rmul4-1";

//Capa de aplicación
String BASE_URL = "http://54.227.168.241:8000/";

//Tomar un grupo usando Nyquist
void takeTest(){

}

void POSTRequest(String url , String data){
  HTTPClient http;
  http.begin(url.c_str()); //TCP handshake
  int httpResponseCode = http.POST(data); // Http request
  Serial.println(httpResponseCode);
  if(httpResponseCode == 200){
    String responseBody = http.getString();
    Serial.println(responseBody);
  }else{
      Serial.println("Error on HTTP request");
  }
}

String takeSingleSample(){
  int value = random(0,1024); //ADC 10-bit
  int timestamp = millis(); //uptime
  String deviceName = "HX711";
  String units = "ADC";
  JSONVar sample; //{}
  sample["value"] = value;
  sample["timestamp"] = timestamp;
  sample["deviceName"] = deviceName;
  sample["units"] = units;
  return JSON.stringify(sample);
}

void sendSingleSample(){
  String json = takeSingleSample();
  Serial.println(json);
  String url = "http://54.227.168.241:8000/readings";
  POSTRequest(url, json);
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
      sendSingleSample();
    }
  }
}
```
