#include <SoftwareSerial.h>
#include <LiquidCrystal_I2C.h>
#include <DFRobotDFPlayerMini.h>
 
SoftwareSerial mySoftwareSerial(12, 10); // RX, TX
DFRobotDFPlayerMini myDFPlayer;
LiquidCrystal_I2C lcd(0x27, 16, 2);
 
int poten = A3;
int ocupado = 6;
int reproducir = 5;
int parar = 4;
int proxima = 3;
int anterior = 2;
 
int contseg = 0;
int contmin = 0;
int VOL, VOL_OLD;
int flag = 0;
int currentTrack = 1; // Para rastrear la pista actual
 
void setup() {
    pinMode(poten, INPUT);
    pinMode(reproducir, INPUT);
    pinMode(parar, INPUT);
    pinMode(proxima, INPUT);
    pinMode(anterior, INPUT);
    pinMode(ocupado, INPUT);
    
    Serial.begin(9600);
    mySoftwareSerial.begin(9600);
    myDFPlayer.begin(mySoftwareSerial);
    
    lcd.init();
    lcd.backlight();
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Santos MP3");
    lcd.setCursor(0, 1);
    lcd.print("** MP3 Player **");
    delay(3000);
    
    myDFPlayer.setTimeOut(500);
    myDFPlayer.volume(30); // 0 - 30
    myDFPlayer.EQ(DFPLAYER_EQ_BASS);
    myDFPlayer.outputDevice(DFPLAYER_DEVICE_SD);
myDFPlayer.play(currentTrack); // Reproducir la primera canción al iniciar
}
 
void loop() {
    int ant = digitalRead(anterior);
    int pro = digitalRead(proxima);
    int par = digitalRead(parar);
    int rep = digitalRead(reproducir);
    int ocu = digitalRead(ocupado);
    int pot = analogRead(poten);
 
    // Reproducción de la canción
    if (rep == LOW) {
        delay(50); // debounce
myDFPlayer.play(currentTrack);
        reprod();
    }
 
    // Canción anterior
    if (ant == LOW) {
        delay(50); // debounce
        currentTrack = currentTrack > 1 ? currentTrack - 1 : 1; // No bajar de 1
        myDFPlayer.previous();
        contseg = 0;
        contmin = 0;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Anterior...");
        delay(500);
        reprod();
    }
 
    // Siguiente canción
    if (pro == LOW) {
        delay(50); // debounce
        currentTrack = currentTrack < 7 ? currentTrack + 1 : 7; // No subir de 7
        myDFPlayer.next();
        contseg = 0;
        contmin = 0;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Siguiente...");
        delay(500);
        reprod();
    }
 
    // Pausa y reanudar
    if (par == LOW) {
        delay(50); // debounce
        if (flag == 0) {
            flag = 1;
            myDFPlayer.pause();
            lcd.setCursor(0, 0);
            lcd.print("Pause  ");
        } else {
            flag = 0;
            myDFPlayer.start();
            reprod();
        }
    }
 
    // Conteo de tiempo
    if (ocu == 0) { // Reproduciendo
        contseg++;
        lcd.setCursor(11, 1);
        lcd.print(contmin);
        lcd.print(":");
        if (contseg < 10) {
            lcd.print("0");
        }
        lcd.print(contseg);
        delay(1000);
        if (contseg >= 60) {
            contmin++;
            contseg = 0;
        }
    }
 
    // Ajuste de volumen
    VOL = map(pot, 0, 1023, 0, 30);
    if (VOL != VOL_OLD) {
        VOL_OLD = VOL;
        myDFPlayer.volume(VOL);
        lcd.setCursor(0, 1);
        lcd.print("Vol:");
        lcd.print(VOL);
    }
 
    // Pantalla de espera
    if ((ocu == 1) && (flag == 0)) {
        lcd.setCursor(0, 0);
        lcd.print("Santos MP3");
        lcd.setCursor(0, 1);
        lcd.print("** MP3 Player **");
        delay(20);
    }
}
 
void reprod() {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Playing");
    delay(1000);
    lcd.setCursor(9, 0);
    int currentFileNumber = myDFPlayer.readCurrentFileNumber();
    if (currentFileNumber != -1) {
        lcd.print(currentFileNumber); // Muestra la canción actual
    } else {
        lcd.print("Error");
    }
    delay(1000);
}
