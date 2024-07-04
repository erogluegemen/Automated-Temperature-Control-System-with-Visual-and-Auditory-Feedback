# Automated-Temperature-Control-System-with-Visual-and-Auditory-Feedback

## Introduction
The objective of this project is to create a temperature control system using an Arduino microcontroller that regulates a fan motor's speed based on the ambient temperature. Additionally, the system provides visual feedback using an RGB LED and auditory feedback using a buzzer.


## Project Overview
![embedded](https://github.com/erogluegemen/Automated-Temperature-Control-System-with-Visual-and-Auditory-Feedback/assets/30879498/b2e6746c-89aa-4c6e-9744-9667a96bf267)

## Project Code
```
#include <Adafruit_LiquidCrystal.h>
#define TEMPERATURE A0
#define MOTOR 3
#define BUZZER_PIN 4

Adafruit_LiquidCrystal lcd(0);

// define RGB LED pins
int redPin = 9;
int greenPin = 11;  
int bluePin = 10; 

// define button pin
int interruptPin = 2;

// define interrupt variables
volatile bool state = false;
volatile int seconds = 0;
unsigned long lastDebounceTime = 0;  

void setup() {
  Serial.begin(9600);
  lcd.begin(16, 2);
  pinMode(MOTOR, OUTPUT);
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  pinMode(interruptPin, INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(interruptPin), button_pressed, FALLING);
  pinMode(BUZZER_PIN, OUTPUT);
}

void loop() {
  if (state) {
    lcd.setCursor(0, 0);
    lcd.setBacklight(1); 
    lcd.print("Turned ON: ");
    lcd.setCursor(11, 0);
    lcd.print("   ");
    lcd.setCursor(11, 0);
    lcd.print(seconds);

    int rawTemp = analogRead(TEMPERATURE);
    int temperature = map(rawTemp, 20, 358, -40, 125);
    int speed = speedDecider(temperature);

    analogWrite(MOTOR, speed);
    setLight(temperature);
    
    lcd.setCursor(0, 1);
    lcd.print("T: ");
    lcd.setCursor(5, 1);
    lcd.print(" ");
    lcd.setCursor(3, 1);
    lcd.print(temperature);
    
    lcd.setCursor(7, 1);
    lcd.print("S: ");
    lcd.setCursor(12, 1);
    lcd.print(" ");
    lcd.setCursor(10, 1);
    lcd.print(speed);
  }
  else {
    analogWrite(MOTOR, 0);
    setColor(0, 0, 0);
    digitalWrite(BUZZER_PIN, LOW);
    lcd.setCursor(0, 0);
    lcd.print("Turned OFF: ");
    lcd.setCursor(12, 0);
    lcd.print("   ");
    lcd.setCursor(12, 0);
    lcd.print(seconds);
    lcd.setCursor(0, 1);
    lcd.print("               ");
  }
}

void button_pressed() {
  unsigned long currentTime = millis();
  if (currentTime - lastDebounceTime > 50) { 
    state = !state;
    seconds = millis() / 1000;
    Serial.println(seconds);
    lastDebounceTime = currentTime;
  }
}

int speedDecider(int temp) {
  if (temp < 10)
    return 0;
  else if (temp > 60)
    return 255;
  else
    return map(temp, 10, 60, 0, 255);
}

void setLight(int temp) {
  // green for cool
  if (temp <= 30)
    setColor(0, 255, 0);   
  else if (temp >= 80) {
    // blink red LED
    static bool toggle = false;
    if (toggle) {
      setColor(255, 0, 0);
      digitalWrite(BUZZER_PIN, HIGH);
    } else { // turn off the LED
      setColor(0, 0, 0); 
      digitalWrite(BUZZER_PIN, LOW);
    }
    toggle = !toggle;
  } else {
    setColor(255, 20, 0); // orange for moderate
    digitalWrite(BUZZER_PIN, LOW);
  }
}

void setColor(int redValue, int greenValue, int blueValue) {
  analogWrite(redPin, redValue);
  analogWrite(greenPin, greenValue);
  analogWrite(bluePin, blueValue);
}
```

## Contact
For any inquiries or feedback, please contact the project team at: <br>
[@Egemen Eroglu](https://github.com/erogluegemen) <br>
[@Ece Akdeniz](https://github.com/ece-akdeniz) 
