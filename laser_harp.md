## Laser Harp

A laser harp is a electronic instrument that uses laser diodes as strings to play music. The laser harp uses an Arduino Uno that had MIDI controller firmware flashed to it. 

This build required:
- Arduino Uno
- 6x Laser Diodes
- 6x Photoresistors
- 6x 100 Ohm Resistors
- MIDI Emulator (such as garage band)

The diodes, photoresistors, and resistors were wired in parallel to the Arduino as shown below:
![Schematic](/images/laser_harp_schematic.jpg)

The code for this is always monitoring the current going through each photoresistor by using analogRead(), when the laser is blocked and no light is entering the photoresistor, no current is read and the Arduino knows to play a note in which it calls midiMessage() and writes the data to the serial port.

''' c
void loop() { 
  for(int i=0;i<notes;i++){   
    
    if((analogRead(i)<threshold)&&(notePlaying[i]==false)){        //note on & CC messages
      int note=beginNote+scale[i];
      midiMessage(0x90, 1, note, 100);                          
      notePlaying[i]=true;                                               
    }
    
    if((analogRead(i)>threshold)&&(notePlaying[i]==true)){        //note off messages
      int note=beginNote+scale[i];
      midiMessage(0x80, 1, note, 0x00);
      notePlaying[i]=false;                                         
    }
  }
}

void midiMessage(int commandByte, int channelByte, int data1Byte, int data2Byte) {
  if(midiMode){
    Serial.write(commandByte);
    Serial.write(channelByte);
    Serial.write(data1Byte);
    Serial.write(data2Byte);
  }else{
    if(commandByte==0x90){
      Serial.print("Note On message: note ");
    }else if(commandByte==0x80){
      Serial.print("Note Off message: note ");
    }
    Serial.print(data1Byte);
    Serial.print(", velocity ");
    Serial.println(data2Byte);
  }
}
'''

The final product is shown below:
![Harp](/images/laser_harp.jpg)
(This was the only picture I took of it, unfortunately it was in my messy garage, I was using fog so the light would refract off the particles and make the laser beam visible)
