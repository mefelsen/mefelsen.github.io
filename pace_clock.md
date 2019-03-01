## LED Pace Clock
As a personal project, I wanted to combine my passions; sports and technology. This was a great way to put the two together.

A little background on the project, I swam competitively for several years and I enjoyed having immediate feedback during training. So I came up with this idea of a LED strip pace clock. The strip is placed at the bottom of the pool and you have to race the light. The point of this is to keep pace and it provides immediate and visual feedback. This really helps when you tweak your stroke technique. This LED strip can also be used for track as well since both sports consist of an athlete covering a certain amount of distance over time just in a different medium(water/air).

The LED strip is connected to an Arduino Mega. The Arduino has code on it that allows it to interface with a GUI that I made using Visual Studio. Below is a wiring diagram of how the LED strip is connected to the Arduino.
![Schematic](/images/pace_clock_schematic.png)
This diagram is not completely accurate as the software I used to design this diagram wouldn’t allow me to place external power supplies. Instead of power and ground wires connecting directly to the Arduino. They are connected to a 5V power supply. The serial connection from the Arduino to the data in pin on the strip is the same.

The most difficult part was parsing the serial data and converting it into data the Arduino could use.
```c
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
}
```
Below is the GUI that I made using Windows Forms. This is written in C#.
![GUI](/images/pace_clock_gui.png)
The code below is how I take the inputted by the user via API and convert them to serial data for the Arduino to use.

'''c
void getAvailableComPorts()
        {
            ports = SerialPort.GetPortNames();
        }
void sendtoArduino()
        {
            //
            //Length
            //
            if (comboBox2.SelectedIndex == 0)
            {
                port.Write("#LEN00000000\n");
            }
            else
            {
                port.Write("#LEN10000000\n");
            }
            //
            //Number of times
            //
            string val = numericUpDown1.Value.ToString();
            if (numericUpDown1.Value > 9)
            {
                port.Write("#2TIM" + val + "00000\n");
                Debug.WriteLine("# of times: " + val);
            }
            else
            {
                port.Write("#1TIM" + val + "000000\n");
                Debug.WriteLine("# of times: " + val);
            }
            //
            //Pace
            //
            string total_seconds = convert(maskedTextBox1.Text);
            total_seconds = format(total_seconds);
            port.Write("#PACE" + total_seconds + "\n");
            Debug.WriteLine("Pace: " + total_seconds);
            //
            //Interval
            //
            string interval = convert(maskedTextBox2.Text);
            interval = format(interval);
            port.Write("#REST" + interval + "\n");
            Debug.WriteLine("Interval: " + interval);
            //
            //Distance
            //
            string s_dist = comboBox3.SelectedItem.ToString();
            int dist = Int32.Parse(s_dist) / 25;
            s_dist = dist.ToString();
            if (dist > 9)
            {
                port.Write("#2DIS" + s_dist + "00000\n");
                Debug.WriteLine("Distance: " + s_dist);
            }
            else
            {
                port.Write("#1DIS" + s_dist + "000000\n");
                Debug.WriteLine("Distance: " + s_dist);
            }
        }

        private void connectToArduino()
        {
            isConnected = true;
            string selectedPort = comboBox1.GetItemText(comboBox1.SelectedItem);
            try
            {
                port = new SerialPort(selectedPort, 9600, Parity.None, 8, StopBits.One);
            }
            catch (ManagementException e)
            {
                //Do Nothing
            }
            port.Open();
            port.Write("#STAR0000000\n");
            button1.Text = "Disconnect";
        }

        private void disconnectFromArduino()
        {
            isConnected = false;
            port.Write("#STOP0000000\n");
            port.Close();
            button1.Text = "Connect";
        }

        private string AutodetectArduinoPort()
        {
            //Set Arduino port as default
            ManagementScope connectionScope = new ManagementScope();
            SelectQuery serialQuery = new SelectQuery("SELECT * FROM Win32_SerialPort");
            ManagementObjectSearcher searcher = new ManagementObjectSearcher(connectionScope, serialQuery);

            try
            {
                foreach (ManagementObject item in searcher.Get())
                {
                    string desc = item["Description"].ToString();
                    string deviceId = item["DeviceID"].ToString();

                    if (desc.Contains("Arduino"))
                    {
                        return deviceId;
                    }
                }
            }
            catch (ManagementException e)
            {
                //Do Nothing
            }

            return null;
        }

        string convert(string s)
        {
            /*convert string from masked textbox to number for second conversion
            and then back to string to send to arduino*/
            int minutes;
            int seconds;
            double milliseconds;
            double total_seconds;
            string s_total_seconds;
            try
            {
                minutes = Int32.Parse(s.Substring(0, 2));
                seconds = Int32.Parse(s.Substring(3, 2));
                milliseconds = Double.Parse(s.Substring(6, 2));
                total_seconds = (minutes * 60) + seconds + (milliseconds * .01);
                s_total_seconds = total_seconds.ToString();
                if (milliseconds == 0)
                {
                    s_total_seconds += ".00";
                }
                return s_total_seconds;
            }
            catch (FormatException)
            {
                //Do Nothing
            }
            return "";
        }

        string format(string s)
        {
            //Make sure all strings are the same # of characters
            for (int i = s.Length; i < 7; i++)
            {
                s += "0";
            }
            return s;
        }
'''
[Here](https://www.youtube.com/watch?v=GyDT5p9yfjY) is a video of the LED Pace Clock in action.