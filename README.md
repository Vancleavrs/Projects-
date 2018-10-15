////////////////////////////////////////////////////////////////////////
// Arduino Bluetooth Interface with Brainsense
// 
// This is example code provided by Pantech Prolabs. and is provided
// license free.
////////////////////////////////////////////////////////////////////////
#include <FastLED.h>

#define BAUDRATE 57600
#define DEBUGOUTPUT 0
#define ACTIVATE_PIN 2 
#define DATA_PIN    3
#define LED_TYPE    WS2811
#define COLOR_ORDER GRB
#define NUM_LEDS 48
#define BRIGHTNESS          160
#define FRAMES_PER_SECOND  300
CRGB leds[NUM_LEDS];

#define powercontrol 10



// checksum variables
byte generatedChecksum = 0;
byte checksum = 0; 
int payloadLength = 0;
byte payloadData[64] = {0};
byte attention = 0;

// system variables
long lastReceivedPacket = 0;
boolean bigPacket = false;
boolean UP = true; 
int prev_pos = 0;
int current_pos = 0;
uint8_t gHue = 0; // rotating "base color" used by many of the patterns
int whackamoleStarted = 0;
//////////////////////////
// Microprocessor Setup //
//////////////////////////
void setup() {
  pinMode(ACTIVATE_PIN, INPUT_PULLUP);
  pinMode(4, OUTPUT);
  digitalWrite(4, LOW);
  digitalWrite(ACTIVATE_PIN, LOW);
  Serial.begin(BAUDRATE);           // USB
  FastLED.addLeds<NEOPIXEL, DATA_PIN>(leds, NUM_LEDS);
  
  delay(3000); // 3 second delay for recovery  
  // tell FastLED about the LED strip configuration
  FastLED.addLeds<LED_TYPE,DATA_PIN,COLOR_ORDER>(leds, NUM_LEDS).setCorrection(TypicalLEDStrip);
  FastLED.setBrightness(BRIGHTNESS);
  initalizeLED();
  clearLED();
  pinMode(5, OUTPUT);
  digitalWrite(5, LOW);
  pinMode(6, OUTPUT);
  digitalWrite(6, HIGH);

}

////////////////////////////////
// Read data from Serial UART //
////////////////////////////////
byte ReadOneByte() {
  int ByteRead;
  while(!Serial.available());
  ByteRead = Serial.read();

#if DEBUGOUTPUT  
  Serial.print((char)ByteRead);   // echo the same byte out the USB serial (for debug purposes)
#endif

  return ByteRead;
}
/////////////
//MAIN LOOP//
/////////////
void loop() {
  prev_pos = 0;
  if((digitalRead(ACTIVATE_PIN) == HIGH)){
    whackamoleStarted +=1;
    delay(2000);
  }
  clearLEDS();
  while((whackamoleStarted > 0) && (digitalRead(ACTIVATE_PIN) == HIGH) ){
  // Look for sync bytes
  if(ReadOneByte() == 170) 
  {
    //header consists of 3 bytes, 2 sync bytes and 1 payload length byte. 
    //first 2 (sync) bytes are 0xAA which equals 170 in decimal, these two sync bytes
    // signal the beginning of the first packet 
    if(ReadOneByte() == 170) 
    {
        payloadLength = ReadOneByte();
      
        if(payloadLength > 169)                      //Payload length can not be greater than 169
        return;
        generatedChecksum = 0;        
        for(int i = 0; i < payloadLength; i++) 
        {  
        payloadData[i] = ReadOneByte();            //Read payload into memory
        generatedChecksum += payloadData[i];
        }   

        checksum = ReadOneByte();                      //Read checksum byte from stream      
        generatedChecksum = 255 - generatedChecksum;   //Take one's compliment of generated checksum

        if(checksum == generatedChecksum) 
        {    
          attention = 0;

          for(int i = 0; i < payloadLength; i++) 
          {                                          // Parse the payload
          switch (payloadData[i]) 
          {
          case 2:
            i++;            
            bigPacket = true;            
            break;
          case 4:
            i++;
            attention = payloadData[i];                        
            break;
          case 5:
            i++;
            break;
          case 0x80:
            i = i + 3;
            break;
          case 0x83:
            i = i + 25;      
            break;
          default:
            break;
          } // switch
        } // for loop

#if !DEBUGOUTPUT
        // *** Add your code here ***
        if(bigPacket) 
        {
          switch(attention / 10) 
          {
          case 0:
            current_pos = 0;   
            manageLEDS();     
            break;
          case 1:
            current_pos = 1; 
            manageLEDS();     
            break;
          case 2:
            current_pos = 2; 
            manageLEDS();  
            break;
          case 3:                 
            current_pos = 3; 
            manageLEDS();           
            break;
          case 4:  
            current_pos = 4;  
            manageLEDS();           
            break;
          case 5:  
            current_pos = 5; 
            manageLEDS();                
            break;
          case 6:             
            current_pos = 6; 
            manageLEDS();                  
            break;
          case 7: 
            current_pos = 7;  
            manageLEDS();               
            break;    
          case (8):
            current_pos = 8;  
            manageLEDS();     
            for(int j = 0; j< NUM_LEDS; j++){
              leds[j] = CRGB:: Black;
              FastLED.show();
            }
            
            delay(2000);
            for(int k = 0; k< NUM_LEDS; k++){
              leds[k] = CRGB:: Green;
              FastLED.show();
            }
            
            delay(2000);
            for(int h = 0; h< NUM_LEDS; h++){
              leds[h] = CRGB:: Black;
              FastLED.show();
            } 
            digitalWrite(4, HIGH);
            digitalWrite(5, HIGH);
            digitalWrite(6, LOW);
              while(1){
                //do nada  
               
              }
              break;
           }                        
        }
#endif        
        bigPacket = false;        
      }
      else {
        // Checksum Error
      }  // end if else for checksum
    } // end if read 0xAA byte
  } // end if read 0xAA byte
 }
}
void clearLED() {
  for (int i = 0; i < NUM_LEDS; i++) {
    leds[i] = CRGB::Black;
    FastLED.show();
  }
  return;
}
void initalizeLED() {
  for (int i = 0; i < NUM_LEDS; i++) {
    leds[i] = CRGB::Red;
    
    FastLED.show();
  }
  delay(1000);
  return;
}
void manageLEDS(){
  
  if(digitalRead(ACTIVATE_PIN) ==LOW){
    prev_pos = 0;
    return;
  }
  if (prev_pos < current_pos){
    if(digitalRead(ACTIVATE_PIN) ==LOW){
    prev_pos = 0;
    return;
  }
    UP = true;
  }
  else if (prev_pos > current_pos){
    if(digitalRead(ACTIVATE_PIN) ==LOW){
    prev_pos = 0;
    return;
  }
    UP = false; 
  }
  if (UP){
    for(int j = prev_pos*7; j < (current_pos*7)-6; j+=7){
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      
      leds[j] = CRGB::Red;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+1] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+2] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+3] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+4] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+5] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+6] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j+7] = CRGB:: Red; 
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
    }
  }
  else{
    for(int j = (prev_pos*7); j >= (current_pos*7)+6; j-=7){
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      leds[j] = CRGB::Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-1] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-2] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-3] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-4] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-5] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-6] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
      leds[j-7] = CRGB:: Black;
      delay(400);
      if(digitalRead(ACTIVATE_PIN) ==LOW){
        prev_pos = 0;
        return;
      }
      FastLED.show();
    }
  }
  prev_pos = current_pos;
  return;            
}
void clearLEDS(){
  for(int i = 0; i< NUM_LEDS; i++){
    leds[i] = CRGB::Black;
  }
  FastLED.show();
  delay(500);
}
