
<h1>The Prototype</h1>

![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591629659546_image.png)

![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591629612971_image.png)

![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591629877367_image.png)


<h1>The Circuit</h1>

https://www.tinkercad.com/things/h1esqGi3XCH-final-project/editel

![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591624458797_image.png)

The components that form my circuit are:

- x1 Arduino Uno 
- x1 BreadBoard
- x1 100 Ω Resistor
- x1 4.7 kΩ Resistor
- x1 PhotoResistor
- x1 Red LED (to replace the laser transmitter)
- x3 Servo Motors
- JumpWires

The Arduino is the most important piece for this project, is the microcontroller which will be connected to all the other components, and is what will read the inputs and activate the different components.

The BreadBoard is simply the construction base, I will use this piece to connect the components with each other.

The two resistors I am using have an important work to do: they reduce the current flow, to avoid burning the components with too much electricity.

The Photoresistor will interact with the environment, turning on the prototype (laser and servo motors) when the surrounding light is less than a pre-decided intensity.

The Laser will project a light onto the closest object/wall.

 The Servo Motors are what will move the mechanic arm and, subsequently, the laser.
 
 The jumpWires connect all the above together.
 

What is a Laser Transmitter?

The origin of the word laser comes from the acronym Light Amplification by Stimulated Emission of Radiation.
As the acronym says a laser transmitter emits a beam of laser light through a process of optical amplification based on the stimulated emission of electromagnetic radiation


![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591622038307_Laser+1.png)
![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591622038186_laser+2.png)




What is a Servo Motor?

A servo Motoris an actuator that allows for precise control of angular position, velocity and acceleration.
It can rotate up to 180 degrees and it consists of a small gearing system, a coiled motor and a control circuit, as well as a pot for rotation control.


![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591622375839_servo+motor+2.jpg)


The Servo motors have 3 pins to connect: one will connect to ground, the second one to power and the last one will connect to a PWM input on the arduino.

What is a Photoresistor?

A photoresistor includes a first electrode layer, a photosensitive material layer, and a second electrode layer which are stacked with each other. The first electrode layer is located on a first surface of the photosensitive material and the second electrode layer is located on a second surface, it’s also important to know that they are opposite to each other. Between the first and second layers there is a carbon nanotube film structure consisting of a plurality of carbon nanotubes substantially aligned along a single preferred direction.

![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591622544021_photoresistor+1.png)


The photoresistor in addition to being connected to power and ground, it needs to be connected to an Analog input pin of the Arduino (A0) for the arduino to read the information.


<h1>The Code</h1>


    //**************************************************************
    #include <Servo.h>
    
    float min_x = 5;
    float max_x = 355;
    float min_y = 5;
    float max_y = 90;
    float min_z = 5;
    float max_z = 90;
    int min_freeze = 200;
    int max_freeze = 3000;
    float minimal_movement = 5;
    
    int random_delay;
    float x_position = min_x + (max_x - min_x)/2;
    float y_position = min_y + (max_y - min_y)/2;
    float z_position = min_z + (max_z - min_z)/2;
    float x_old_position = x_position;
    float y_old_position = y_position;
    float z_old_position = z_position;
    float x_new_position;
    float y_new_position;
    float z_new_position;
    float x_speed;
    float y_speed;
    float z_speed;
    int movement_time;
    
    Servo x_servo;  
    Servo y_servo;
    Servo z_servo;
    int pos = 0;
    
    const int LIGHT_PIN = A0; // Pin connected to voltage divider output
    const int LED_PIN = 13; // This replace the Laser
    const float VCC = 4.98; // Measured voltage of Ardunio 5V line
    const float R_DIV = 4660.0; // Measured resistance of 3.3k resistor
    
    // Set this to the minimum resistance require to turn an LED on:
    const float DARK_THRESHOLD = 10000.0;
    
    void setup() 
    {
      z_servo.attach(11);  // attaches the z servo on pin 11 to the servo object
      y_servo.attach(6);  // attaches the y servo on pin 6 to the servo object
      x_servo.attach(9);  // attaches the x servo on pin 9 to the servo object
      
      
      //Place the servos in the center at the beginning 
      z_servo.write(z_position);
      y_servo.write(y_position); 
      x_servo.write(x_position);
      
      Serial.begin(9600);
      pinMode(LIGHT_PIN, INPUT);
      pinMode(LED_PIN, OUTPUT);
    }
    
    void loop() 
    {
      // Read the ADC, and calculate voltage and resistance from it
      int lightADC = analogRead(LIGHT_PIN);
      if (lightADC > 0){
       
        // Use the ADC reading to calculate voltage and resistance
        float lightV = lightADC * VCC / 1023.0;
        float lightR = R_DIV * (VCC / lightV - 1.0);
        Serial.println("Voltage: " + String(lightV) + " V");
        Serial.println("Resistance: " + String(lightR) + " ohms");
    
        // If resistance of photocell is greater than the dark
        // threshold setting, turn the LED on.
      if (lightR >= DARK_THRESHOLD){
          digitalWrite(LED_PIN, HIGH);
                      //SERVOS
                movement_time = random(10,40);
                      random_delay = random(min_freeze, max_freeze);
                      x_new_position = random(min_x+minimal_movement, max_x-minimal_movement);
                      y_new_position = random(min_y+minimal_movement, max_y-minimal_movement);
                      z_new_position = random(min_z+minimal_movement, max_z-minimal_movement);
      
                      if( (z_new_position > z_old_position) && (abs(z_new_position - z_old_position) < 5 )) {
                               z_new_position = z_new_position + minimal_movement;
                      }  else if ( (z_new_position < z_old_position) && (abs(z_new_position - z_old_position) < 5 )) {
                        z_new_position = z_new_position - minimal_movement;
                      }
      
                      if( (y_new_position > y_old_position) && (abs(y_new_position - y_old_position) < 5 )) {
                               y_new_position = y_new_position + minimal_movement;
                      }  else if ( (y_new_position < y_old_position) && (abs(y_new_position - y_old_position) < 5 )) {
                        y_new_position = y_new_position - minimal_movement;
                      }
      
                      if( (x_new_position > x_old_position) && (abs(x_new_position - x_old_position) < 5 )) {
                        x_new_position = x_new_position + minimal_movement;
                      }  else if ( (x_new_position < x_old_position) && (abs(x_new_position - x_old_position) < 5 )) {
                        x_new_position = x_new_position - minimal_movement;
                      }
      
                      x_speed = (x_new_position - x_old_position)/movement_time;
                      y_speed = (y_new_position - y_old_position)/movement_time; 
                      z_speed = (z_new_position - z_old_position)/movement_time;
                      for (pos = 0; pos < movement_time; pos += 1) { 
                          x_position = x_position + x_speed;
                          y_position = y_position + y_speed;
                          z_position = z_position + z_speed;
                          x_servo.write(x_position);  
                          y_servo.write(y_position);  
                          z_servo.write(z_position);
                        delay(10); 
                      }
                      x_old_position = x_new_position;
                      y_old_position = y_new_position;
                      z_old_position = z_new_position;
                      delay(random_delay);
      }else{
          digitalWrite(LED_PIN, LOW);
    
        Serial.println();
        delay(500);
      }
    }
    }



<h1>3D Components</h1>

https://www.tinkercad.com/things/bAgprTEKalG-final-project/edit

![](https://paper-attachments.dropbox.com/s_F8F1AC22731BD4F2B939A7FBCCF7EA0E0B980977A3D58F674B5B9E09ADFCD575_1591626062134_image.png)

