# Power Flic



## Overview
The basic idea is to turn on/off a physical button. Our experiment turns out to be using Arduino yun, flic to control a power strip. 


### Step 1: Material 

- Arduino yun board
- Jumper wires
- Mini servo
- Power strip


### Step 2: Set up Arduino with mini servo


![Connect mini servo](https://raw.githubusercontent.com/Voxar/Power-Flic/master/Connect%20Arduino%20with%20the%20Mini%20Servo.JPG)


### Step 3: Connect with WiFi network 

- Connect Arduino Yun with WiFi network
- Connect phone with the same network
- Open Flic app in the phone and set up click with HTTP request

![Flic Setup](https://raw.githubusercontent.com/Voxar/Power-Flic/master/Flic%20Setup.PNG)



### Step 4: Write some code in Arduino  



	#include <Bridge.h>
    #include <BridgeServer.h>
    #include <BridgeClient.h>
    #include <Servo.h>
    
    // Listen to the default port 5555, the Yún webserver
    // will forward there all the HTTP requests you send
    BridgeServer server;
    Servo myservo;
    void setup() {
      // Bridge startup
      pinMode(13, OUTPUT);
      digitalWrite(13, LOW);
      Bridge.begin();
      digitalWrite(13, HIGH);
    
    
      // Listen for incoming connection only from localhost
      // (no one from the external network could connect)
      server.listenOnLocalhost();
      server.begin();
      myservo.attach(5);
    }
    
    void loop() {
      // Get clients coming from server
      BridgeClient client = server.accept();
    
      // There is a new client?
      if (client) {
        // Process request
        process(client);
    
        // Close connection and free resources.
        client.stop();
      }
    
      delay(50); // Poll every 50ms
    }
    
    void process(BridgeClient client) {
      // read the command
      String command = client.readStringUntil('/');
    
      // is "digital" command?
      if (command == "digital") {
        digitalCommand(client);
      }
    }
    
    void digitalCommand(BridgeClient client) {
      int pin, value;
    
      // Read pin number
      pin = client.parseInt();
    
      // If the next character is a '/' it means we have an URL
      // with a value like: "/digital/13/1"
      if (client.read() == '/') {
        value = client.parseInt();
        turnServo(value);
      } else {
        client.println("can not read value");
      }
    
      // Send feedback to client
      client.print(F("Pin D"));
      client.print(pin);
      client.print(F(" set to "));
      client.println(value);
    }
    
    
    void turnServo(int value) {
      //turn off is closewise
      if(value == 1) {
        myservo.write(180);
      } else {
        myservo.write(0);
      }
        delay(500);
        myservo.write(90);
        delay(500);
    }
    
	
	
### Step 5: Done and try out 


![The final result](https://raw.githubusercontent.com/Voxar/Power-Flic/master/Finished%20product.JPG)



### Youtube video  

[Servo controlled power strip with flic and arduino yun](https://youtu.be/XDSRUGdcWWA)