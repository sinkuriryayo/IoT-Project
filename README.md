# IoT-Project
This is IoT project to turn on/off electrical lamp by using smartphone/pc or other network device.
Here there are codes to make successfull project 

#include <WiFi.h>

const char* ssid = "ESP32_LED_SiAli";
const char* password = "12345678";

const int ledPin = 25;
WiFiServer server(80);

String header;
bool ledState = false;

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  WiFi.softAP(ssid, password);
  Serial.print("AP IP address: ");
  Serial.println(WiFi.softAPIP());

  server.begin();
}

void loop() {
  WiFiClient client = server.available();

  if (client) {
    Serial.println("New Client.");
    String currentLine = "";

    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        header += c;

        if (c == '\n') {
          if (currentLine.length() == 0) {
            // Handle requests
            if (header.indexOf("GET /on") >= 0) {
              ledState = true;
              digitalWrite(ledPin, HIGH);
            } else if (header.indexOf("GET /off") >= 0) {
              ledState = false;
              digitalWrite(ledPin, LOW);
            }

            // HTML & CSS
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            client.println("<!DOCTYPE html><html><head>");
            client.println("<meta name='viewport' content='width=device-width, initial-scale=1'>");
            client.println("<style>");
            client.println("body { font-family: Arial; background: #f0f2f5; text-align: center; padding: 20px; }");
            client.println(".card { background: white; max-width: 400px; margin: auto; padding: 20px; border-radius: 15px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }");
            client.println("h2 { margin-bottom: 10px; }");
            client.println("p { font-size: 1.2rem; }");
            client.println(".btn { display: inline-block; margin: 10px; padding: 12px 24px; font-size: 16px; border: none; border-radius: 8px; cursor: pointer; }");
            client.println(".on { background-color: #4CAF50; color: white; }");
            client.println(".off { background-color: #f44336; color: white; }");
            client.println(".btn:hover { opacity: 0.9; }");
            client.println("</style></head><body>");
            client.println("<div class='card'>");
            client.println("<h2>ESP32 LED Controller</h2>");
            client.print("<p>Status: <strong>");
            client.print(ledState ? "ON" : "OFF");
            client.println("</strong></p>");
            client.println("<a href='/on'><button class='btn on'>Turn ON</button></a>");
            client.println("<a href='/off'><button class='btn off'>Turn OFF</button></a>");
            client.println("</div>");
            client.println("</body></html>");

            client.println();
            break;
          } else {
            currentLine = "";
          }
        } else if (c != '\r') {
          currentLine += c;
        }
      }
    }

    header = "";
    client.stop();
    Serial.println("Client disconnected.");
  }
}
