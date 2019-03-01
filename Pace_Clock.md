## LED Pace Clock
As a personal project, I wanted to combine my passions; sports and technology. This was a great way to put the two together.

A little background on the project, I swam competitively for several years and I enjoyed having immediate feedback during training. So I came up with this idea of a LED strip pace clock. The strip is placed at the bottom of the pool and you have to race the light. The point of this is to keep pace and it provides immediate and visual feedback. This really helps when you tweak your stroke technique. This LED strip can also be used for track as well since both sports consist of an athlete covering a certain amount of distance over time just in a different medium(water/air).

The LED strip is connected to an Arduino Mega. The Arduino has code on it that allows it to interface with a GUI that I made using Visual Studio. Below is a wiring diagram of how the LED strip is connected to the Arduino.
![Schematic]
This diagram is not completely accurate as the software I used to design this diagram wouldn’t allow me to place external power supplies. Instead of power and ground wires connecting directly to the Arduino. They are connected to a 5V power supply. The serial connection from the Arduino to the data in pin on the strip is the same.

Code is shown below:
`#include <FastLED.h>

<div>#define LEDPIN = 7;<\div>

String inputString = "";
bool stringComplete = false;
String commandString = "";

unsigned int NUM_LEDS = 0;
unsigned int NUM_TIMES = 0;
unsigned int DIST = 0;
double INTERVAL = 0;
double REST = 0;
bool start = false;
unsigned long startMillis;
unsigned long currentMillis;
unsigned long period;
boolean isConnected = false;

int dot = 0;
int k = 0;
int i = 0;

CRGB leds[750];

void setup() {
 Serial.begin(9600);
 FastLED.addLeds<WS2812B, 7>(leds,750);
 startMillis = millis();
}

void loop() {
  if(stringComplete) {
    stringComplete = false;
    getCommand();

  if(commandString.startsWith("STAR")) {
    Serial.print("Signal Started");
  }
  if(commandString.startsWith("STOP")) {
    Serial.print("Signal Stopped");
  }
  if(commandString.startsWith("LEN0")) {
      //# of leds for 25 yard course
      //NUM_LEDS = 686;
      NUM_LEDS = 150;
  }
  if(commandString.startsWith("LEN1")) {
      //# of leds for 25 meter course
      NUM_LEDS = 750;
  }
  if(commandString.startsWith("1TIM")) {
      //determines number of times LED runs down strip
      NUM_TIMES = (int)commandString[4];
  }
  if(commandString.startsWith("2TIM")) {
      NUM_TIMES = commandString.substring(4,6).toInt();
  }
  if(commandString.startsWith("PACE")) {
    //Parse data xxxx.xx
    //Removes trailing 0's
    String temp0 = commandString.substring(4,12);
    for(int i = 0; i < 12; i++) {
      if(temp0[i] == '.') {
        temp0 = commandString.substring(4,i+2);
      }
    }
    INTERVAL = temp0.toDouble();
  }
  if(commandString.startsWith("REST")) {
    //Remove trailing 0's
    String temp = commandString.substring(4,12);
    for(int i = 0; i < 12; i++) {
      if(temp[i] == '.') {
        temp = commandString.substring(4,i+2);
      }
    }
    //Take total interval and subtract pace to get amount of rest before starting LED movement
    REST = temp.toDouble() - INTERVAL;
  }
  if(commandString.startsWith("1DIS")) {
    DIST = (int)commandString[4];
  }
  if(commandString.startsWith("2DIS")) {
    DIST = commandString.substring(4,6).toInt();
  }
  if(commandString.startsWith("GO")) {
    start = true;
    period = (INTERVAL*1000)/(NUM_LEDS*DIST);
  }
    inputString = "";
  }

  if(start) {
    currentMillis = millis();
    if(currentMillis - startMillis >= period && dot < NUM_LEDS - 9 /*&& i%2 == 0 && i < DIST && k < NUM_TIMES*/) {
        leds[dot] = CRGB::Green;
        leds[dot+1] = CRGB::Green;
        leds[dot+2] = CRGB::Green;
        leds[dot+3] = CRGB::Green;
        leds[dot+4] = CRGB::Green;
        leds[dot+5] = CRGB::Green;
        leds[dot+6] = CRGB::Green;
        leds[dot+7] = CRGB::Green;
        leds[dot+8] = CRGB::Green;
        leds[dot+9] = CRGB::Green;
        FastLED.show();
        leds[dot] = CRGB::Black;
        leds[dot+1] = CRGB::Black;
        leds[dot+2] = CRGB::Black;
        leds[dot+3] = CRGB::Black;
        leds[dot+4] = CRGB::Black;
        leds[dot+5] = CRGB::Black;
        leds[dot+6] = CRGB::Black;
        leds[dot+7] = CRGB::Black;
        leds[dot+8] = CRGB::Black;
        leds[dot+9] = CRGB::Black;
        dot++;
        startMillis = currentMillis;
    }
    if(dot == NUM_LEDS - 9) {
      dot = 0;
      start = false;
    }
  }

}
void getCommand() {
  if(inputString.length()>0) {
     commandString = inputString.substring(1,12);
  }
}

void serialEvent() {
  while (Serial.available()) {
    // get the new byte:
    char inChar = (char)Serial.read();
    // add it to the inputString:
    inputString += inChar;
    // if the incoming character is a newline, set a flag
    // so the main loop can do something about it:
    if (inChar == '\n') {
      stringComplete = true;
    }
  }
}`