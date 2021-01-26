#include <BearSSLHelpers.h>
#include <CertStoreBearSSL.h>
#include <ESP8266WiFi.h>
#include <ESP8266WiFiAP.h>
#include <ESP8266WiFiGeneric.h>
#include <ESP8266WiFiMulti.h>
#include <ESP8266WiFiScan.h>
#include <ESP8266WiFiSTA.h>
#include <ESP8266WiFiType.h>
#include <WiFiClient.h>
#include <WiFiClientSecure.h>
#include <WiFiClientSecureAxTLS.h>
#include <WiFiClientSecureBearSSL.h>
#include <WiFiServer.h>
#include <WiFiServerSecure.h>
#include <WiFiServerSecureAxTLS.h>
#include <WiFiServerSecureBearSSL.h>
#include <WiFiUdp.h>

#include <ESP8266WiFi.h>
#include <WiFiClient.h>
#include <ESP8266WebServer.h>
const char INDEX_HTML[] =
  "<!DOCTYPE HTML>"
  "<html>"
  "<head>"
  "<meta name = \"viewport\" content = \"width = device-width, initial-scale = 1.0, maximum-scale = 1.0, user-scalable=0\">"
  "<title>Antennenrotor</title>"
  "<style>"
  "body { background-color: #d2f3eb; font-family: Arial, Helvetica, Sans-Serif; Color: #000000; }"
  "input { font-size:16pt; }"
  "</style>"
  "</head>"
  "<body>"
  "<h1>Antennenrotor</h1>"
  "<FORM action=\"/data\" method=\"post\">"
  "<P>"
  "<INPUT type=\"submit\" name=\"LINKS\" value=\"Links\">&nbsp;"
  "<INPUT type=\"submit\" name=\"STOP\" value=\"Stop\">&nbsp;"
  "<INPUT type=\"submit\" name=\"RECHTS\" value=\"Rechts\"><br><br>"
  "Geschwindigkeit (1 - 10):<br>"
  "<INPUT type=\"text\" value=\"%i\" name=\"SPEED\">"
  "</P>"
  "</FORM>"
  "</body>"
  "</html>";

//Globale Variablen
const char *ssid = "Antenne"; //Name des WLAN
const char *password = "womo12345"; //Passwort für das WLAN

const int stp = 12; //Pin Nummer für den Step Anschluss des Motor Treibers
const int dir = 13; //Pin Nummer für den Richtungs Anschluss des Motor Treibers
int richtung = 0; //aktuelle Richtung
int dly = 100; //Verzögerung für Motor Schritte
int spd = 1; //aktuelle Geschwindigkeit

ESP8266WebServer server(80); //Web-Server starten auf Port 80

//Diese Funktion wird aufgerufen wenn der Request an den Web-Server = "/" ist
void handleRoot() {
  char html[1000];//Buffer für Request
  //Sende die Homepage an den Browser wobei die Geschwindigkeitseingabe
  //mit derv aktuellen Geschwindigkeit vorbesetzt wird
  sprintf(html, INDEX_HTML, spd);
  server.send(200, "text/html", html );
}

//Diese Funktion wird aufgerufen wenn der Request "/left" ist
void handleLeft() {
  richtung = 1;//Richtung auf links und mit Homepage antworten
  handleRoot();
}

//Diese Funktion wird aufgerufen wenn der Request "/right" ist
void handleRight() {
  richtung = 2; //Richtung auf rechts und mit Homepage antworten
  handleRoot();
}

//Diese Funktion wird aufgerufen wenn der Request "/stop" ist
void handleStop() {
  richtung = 0; //Richtung auf stop und mit Homepage antworten
  handleRoot();

}

//Diese Funktion wird aufgerufen wenn der Request "/data?..." ist
void handleData() {

  String sspd;
  //Abhängig von den Argumenten des Requests werden Richtung und
  //Geschwindigkeit gesetzt. Geschwindigkeit zwischen 1 und 10
  if (server.hasArg("LINKS")) richtung = 1;
  if (server.hasArg("RECHTS")) richtung = 2;
  if (server.hasArg("STOP")) richtung = 0;
  if (server.hasArg("SPEED")) {
    sspd = server.arg("SPEED");
    spd = sspd.toInt();
    if (spd < 1) spd = 1;
    if (spd > 10) spd = 10;
    dly = (11 - spd) * 10; //Schrittabstand 100 bei Geschwindigkeit 1
    //und 10 bei Geschwindigkeit 10
  }
  handleRoot(); //Mit Homepage antworten
}

void setup() {
  //Pins vorbereiten
  pinMode(stp, OUTPUT);
  pinMode(dir, OUTPUT);
  digitalWrite(stp, HIGH);
  digitalWrite(dir, HIGH);
  delay(1000);
  Serial.begin(115200);
  Serial.println();
  Serial.print("Configuring access point...");

  WiFi.softAP(ssid, password);

  IPAddress myIP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(myIP);
  //ip ist immer 192.168.4.1
  //Definition der Antwortfunktionen
  server.on("/", handleRoot);
  server.on("/left", handleLeft);
  server.on("/right", handleRight);
  server.on("/stop", handleStop);
  server.on("/data", handleData);
  //Web Server starten
  server.begin();
  Serial.println("HTTP server started");
}

//Hauptschleife
void loop() {
  //Auf Request prüfen
  server.handleClient();
  //je nach Richtung den Richtungs Pin setzen
  //und einen Schritt am Step Pin ausgeben
  //Schrittlänge in ms ist durch dly bestimmt
  if (richtung == 1) {
    Serial.println("links");
    digitalWrite(dir, LOW);
    digitalWrite(stp, LOW);
    delay(dly);
    digitalWrite(stp, HIGH);
    delay(dly);
  }
  if (richtung == 2) {
    Serial.println("rechts");
    digitalWrite(dir, HIGH);
    digitalWrite(stp, LOW);
    delay(dly);
    digitalWrite(stp, HIGH);
    delay(dly);
  }
}
