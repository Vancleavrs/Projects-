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
