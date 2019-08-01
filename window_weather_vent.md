## Window Weather Vent

This project was pitched to me and a few other colleagues. The purpose of this project was to demonstrate a proof of concept with future hopes of manufacturing this product and going to production.
One of the notable outcomes of this project was filing for a **provisional patent**.

The Window Weather Vent is a window insert that automatically opens and closes leuvers based on the temperature and humidity of the room you are in and the atmosphere outside the room. The temperature and humidity was sampled periodically from two different locations. A sample was collected from the exterior of the vent which faced the atmosphere, and another sample was collected in the room from a portable sensor using a RF antenna.

The following materials were used to make this project:
- Arduino Uno
- Touchscreen LCD Shield for Arduino Uno
- 2x temperature sensors
- Servo Motor
- Foam
- Plywood
- Filter
- RF Antenna
- RF Reciever

The Arduino Uno recieved temperature and humidity data from 2 different data pins. One was wired to a RF reciever and the other was connected directly to a temperature sensor. The Uno had a output that was wired to the servo motor to control the leuvers. The remaining data pins were connected to the LCD touchscreen and allowed the user to set what temperature they want the room to be. A custom algorithm would then determine how much to open the luevers.

The frame was made using plywood, and the leuvers were made out of foam for this prototype. The servo motor had custom 3-D printed brackets that allowed the motor to move the leuvers.

An infomercial for this project can be found [Here](https://www.youtube.com/watch?v=iSsPUcMq7kE).
