# colorcode
color sensor code
// Libraries einbinden
#include "Servo.h"
#include "Wire.h"
#include "Adafruit_TCS34725.h"
 
// Servo-Positionen in Grad für Farben, bitte anpassen
const int redPos = 160;
const int orangePos = 130;
const int yellowPos = 100;
const int greenPos = 70;
const int bluePos = 30;
const int nonePos = 0; // Kein Objekt erkannt
 
// Servo-Objekt initialisieren
Servo myservo;
 
// Color Sensor-Objekt initialisieren
// Parameter siehe: https://learn.adafruit.com/adafruit-color-sensors/program-it
Adafruit_TCS34725 tcs = Adafruit_TCS34725(TCS34725_INTEGRATIONTIME_50MS, TCS34725_GAIN_1X);
 
// setup() wird einmal beim Start des Arduino ausgeführt
void setup() {
 
 // Serielle Kommunikation zur Ausgabe der Wert im seriellen Monitor
 Serial.begin(9600);
 Serial.println("Makerblog.at - MuMs Color Sensor");
 
 // Überprüfen, ob Color Sensor sich auch zurückmeldet
 if (tcs.begin()) {
 // Alles OK
 Serial.println("Sensor gefunden");
 } else {
 // Kein Sensor gefunden. Programm an dieser Stelle einfrieren
 Serial.println("TCS34725 nicht gefunden ... Ablauf gestoppt!");
 while (1); // Halt!
 } 
 
 // Der Servo hängt am PWM-Pin 3 
 myservo.attach(3);
 // Servo in Grundstellung fahren
 myservo.write(0);
 delay(1000);
 
}
 
// loop() wird wiederholt, solange der Arduino läuft
void loop() {
 
 // Der Sensor liefert Werte für R, G, B und einen Clear-Wert zurück
 uint16_t clearcol, red, green, blue;
 float average, r, g, b;
 delay(100); // Farbmessung dauert c. 50ms 
 tcs.getRawData(&red, &green, &blue, &clearcol);
 
 // Mein Versuch einer Farbbestimmung für 
 // die 5 M&M-Farben Rot, Grün, Blau, Orange und Gelb
 
 // Durchschnitt von RGB ermitteln
 average = (red+green+blue)/3;
 // Farbwerte durch Durchschnitt, 
 // alle Werte bewegen sich jetzt rund um 1 
 r = red/average;
 g = green/average;
 b = blue/average;
 
 // Clear-Wert und r,g,b seriell ausgeben zur Kontrolle
 // r,g und b sollten sich ca. zwischen 0,5 und 1,5 
 // bewegen. Sieht der Sensor rot, dann sollte r deutlich über 1.0
 // liegen, g und b zwischen 0.5 und 1.0 usw.
 Serial.print("\tClear:"); Serial.print(clearcol);
 Serial.print("\tRed:"); Serial.print(r);
 Serial.print("\tGreen:"); Serial.print(g);
 Serial.print("\tBlue:"); Serial.print(b);
 
 // Versuch der Farbfeststellung anhand der r,g,b-Werte.
 // Am besten mit Rot, Grün, Blau anfangen die die Schwellenwerte
 // mit der seriellen Ausgabe entsprechend anpassen
 if ((r > 1.4) && (g < 0.9) && (b < 0.9)) {
 Serial.print("\tROT");
 myservo.write(redPos);
 }
 else if ((r < 0.95) && (g > 1.4) && (b < 0.9)) {
 Serial.print("\tGRUEN");
 myservo.write(greenPos);
 }
 else if ((r < 0.8) && (g < 1.2) && (b > 1.2)) {
 Serial.print("\tBLAU");
 myservo.write(bluePos);
 }
 // Gelb und Orange sind etwas tricky, aber nach etwas
 // Herumprobieren haben sich bei mir diese Werte als
 // gut erwiesen
 else if ((r > 1.15) && (g > 1.15) && (b < 0.7)) {
 Serial.print("\tGELB");
 myservo.write(yellowPos);
 }
 else if ((r > 1.4) && (g < 1.0) && (b < 0.7)) {
 Serial.print("\tORANGE");
 myservo.write(orangePos);
 } 
 // Wenn keine Regel greift, dann ehrlich sein
 else {
 Serial.print("\tNICHT ERKANNT"); 
 // myservo.write(nonePos);
 }
 
 
 // Zeilenwechsel ausgeben
 Serial.println("");
 
 // Wartezeit anpassen für serielles Debugging
 delay(100);
 
}
