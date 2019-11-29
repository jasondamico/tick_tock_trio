// ARDUINO 1: SIGNAL SENDER

// Library Inclusions 
#include <SoftwareSerial.h>
#include <Wire.h> // For master/slave
#include <SharpIR.h>

// Definitions for distance sensor
#define IR A0 // define signal pin
#define model 1080 // used 1080 because model GP2Y0A21YK0F is used
SharpIR SharpIR(IR, model);

/* BUTTON CONSTANTS */

const int BLUE_BUTTON = A1;
const int GREEN_BUTTON = A2;
const int YELLOW_BUTTON = A3;

// Distance at which the clock stops
const int DISTANCE = 13;

/* SIGNAL CONSTANTS */

const int TIME_SETTER = 1;
const int HOUR_UP = 2;
const int HOUR_TO_MIN = 3;
const int MINUTE_UP = 4;
const int EXIT_TIME_SETTER = 5;

const int ALARM_SETTER = 6;
const int ALARM_HOUR_UP = 7;
const int ALARM_HOUR_TO_MIN = 8;
const int ALARM_MINUTE_UP = 9;
const int EXIT_ALARM_SETTER = 10;

const int ALARM_TRIGGER = 11;
const int SNOOZE = 12;

const int MOTORS_OFF = 13;

const int UNIVERSAL_ON = 14;

/* GLOBAL VARIABLES */

// Used to detect that a button is being pushed: too much current for
// analogRead to be 0 when a button is pushed, there is some fluxuation
int analog_val = 50;

// Used to count how long a button is being held
int counter = 0;

boolean clockOn = false;



/* 
 *  Runs once when the Arduino is first powered
 */
void setup() {
  Wire.begin(); // join i2c bus (address optional for master)

  // Initializes terminal printer, which is used for testing and debugging
  Serial.begin(9600);
  Serial.println("TESTING ENVIRONMENT");
}

/*
 * Loops continuously while the Arduino is on
 */
void loop() {
  // Loops until the clock is turned on
  while(counter < 200 && !clockOn){
    counter = 0;

    while(analogRead(YELLOW_BUTTON) <= analog_val && counter < 200) {
      countUp();
    }

    // Signals that the alarm has been turned on
    if(counter == 200) {
      sendSignal(UNIVERSAL_ON);

      clockOn = true;
      delay(1000);
    }
  }

  // Resets the counter after the button is no longer being pushed 
  counter = 0;

  // Loop is triggered if the blue button is being pushed, counts up
  while(analogRead(BLUE_BUTTON) <= analog_val && counter < 200) {
    countUp();
  }

  if(counter == 200) {
    // Signals for the Slave Arduino to enter Time Setter mode
    sendSignal(TIME_SETTER);
      
    Serial.println("Message sent...");
    timeSetterMode();
  }

  // Resets the counter after the button is no longer being pushed 
  counter = 0;

  // Loop is triggered if the green button is being pushed, counts up
  while (analogRead(GREEN_BUTTON) <= analog_val && counter < 200) {
    countUp();
  }

  if(counter == 200) {
    // Signals fot the Slave Arduino to enter Alarm Setter mode
    sendSignal(ALARM_SETTER);
      
    Serial.println("Message sent...");
    alarmSetterMode();
  }

  // Resets the counter after the button is no longer being pushed 
  counter = 0;

  // Loop is triggered if the yellow button is being pushed, counts up
  while (analogRead(YELLOW_BUTTON) <= analog_val && counter < 200) {
    countUp();
  }

  // Sends "SNOOZE" signal if button is pushed for an intermediate amount of time,
  // sends "ALARM_TRIGGER" signal if held until the count is 200
  if(counter > 20 && counter < 200) {
    sendSignal(SNOOZE);
    delay(1000);
      
    Serial.println("Message sent...");
  } else if (counter == 200) {
    sendSignal(ALARM_TRIGGER);
    delay(1000);
      
    Serial.println("Message sent...");
  }

  // DEBUGGING CODE: Values of buttons and IR sensor are printed to monitor
  Serial.print("B:");
  Serial.print(analogRead(BLUE_BUTTON));
  Serial.print("    G:");
  Serial.print(analogRead(GREEN_BUTTON));
  Serial.print("    Y:");
  Serial.print(analogRead(YELLOW_BUTTON));
  Serial.print("           IR: ");
  Serial.println(SharpIR.distance());

  // Sends MOTORS_OFF signal when the distance sensor detects something in front of it
  if(SharpIR.distance() == DISTANCE){
    sendSignal(MOTORS_OFF);

    Serial.println("Object detected");
    delay(1000);    
  }
}


// SETTER MODES


/*
 * Puts the Master Arduino into Time Setter Mode, which means that the pushes of
 * the blue button coorespond to incrementing either the minutes or hours value by one.
 */
void timeSetterMode() {
  // Sets analog_val to 25 as the current is too great and thus the button would pick up phantom pushes if
  // the value was still 50 (i.e. the sitting analog read value of the button at this point dips below 50)
  analog_val = 25;
  
  boolean buttonPushed = false;
  boolean firstRun = true;
  boolean switchMethod = false;

  /* HOUR CHANGE */
  while(!switchMethod) {
    // Waits for the button to stop being pushed, then breaking out of the loop and itiating the code below
    while(firstRun) {
      if(analogRead(BLUE_BUTTON) >= analog_val) {
        firstRun = false;
        delay(1000);
      }
    }

    // Loop is executed if the button is being pushed and the method has not been switched
    while(analogRead(BLUE_BUTTON) <= analog_val && !switchMethod) {
      countUp();

      // Triggers increment of one hour once the loop is broken
      if(counter >= 80){
        buttonPushed = true;
      }

      // Triggers a breaking of the loop (and thus switching to minute change)
      if(counter >= 600) {
        switchMethod = true;
      }
    } 

    if (buttonPushed && counter < 600) {
      sendSignal(HOUR_UP);

      buttonPushed = false;
    }

    // Resets the counter after the button is no longer being pushed 
    counter = 0;
  } 

  // Signals a switch from the hour editor to the minutes editor
  sendSignal(HOUR_TO_MIN);

  switchMethod = false;
  firstRun = true;
  buttonPushed = false;

  /* MINUTE CHANGE */
  while(!switchMethod) {
    // Waits for the button to stop being pushed, then breaking out of the loop and itiating the code below
    while(firstRun) {
      if(analogRead(BLUE_BUTTON) >= analog_val) {
        firstRun = false;
        delay(1000);
      }
    }

    // Loop is executed if the button is being pushed and the method has not been switched
    while(analogRead(BLUE_BUTTON) <= analog_val && !switchMethod) {
      countUp();

      // Triggers increment of one minute once the loop is broken
      if(counter >= 80){
        buttonPushed = true;
      }

      // Triggers a breaking of the loop (and thus exiting Time Setter Mode)
      if(counter >= 600) {
        switchMethod = true;
      }
    } 

    if (buttonPushed && counter < 600) {
      sendSignal(MINUTE_UP);

      buttonPushed = false;
    }

    // Resets the counter after the button is no longer being pushed 
    counter = 0;
  }

  // Sends a signal to the Slave to exit Time Setter Mode
  sendSignal(EXIT_TIME_SETTER);

  // Resets analog_val to 50 when returning to the loop
  analog_val = 50;
}

/*
 * Puts the Master Arduino into Alarm Setter Mode, which means that the pushes of
 * the green button coorespond to incrementing either the alarm minutes or alarm hours value by one.
 */
void alarmSetterMode() {
  boolean buttonPushed = false;
  boolean firstRun = true;
  boolean switchMethod = false;

  /* ALARM HOUR CHANGE */
  while(!switchMethod) {
    while(firstRun) {
      if(analogRead(GREEN_BUTTON) >= analog_val) {
        firstRun = false;
        delay(1000);
      }
    }
    
    while(analogRead(GREEN_BUTTON) <= analog_val && !switchMethod) {
      countUp();
      if(counter >= 20){
        buttonPushed = true;
      }

      if(counter >= 600) {
        switchMethod = true;
      }
    } 

    if (buttonPushed && counter < 600) {
      sendSignal(ALARM_HOUR_UP);

      buttonPushed = false;
    }

    counter = 0;
  } 

  sendSignal(ALARM_HOUR_TO_MIN);

  switchMethod = false;
  firstRun = true;
  buttonPushed = false;

  /* ALARM MINUTE CHANGE */
  while(!switchMethod) {
    while(firstRun) {
      if(analogRead(GREEN_BUTTON) >= analog_val) {
        firstRun = false;
        delay(1000);
      }
    }

    while(analogRead(GREEN_BUTTON) <= analog_val && !switchMethod) {
      countUp();
      if(counter >= 20){
        buttonPushed = true;
      }

      if(counter >= 600) {
        switchMethod = true;
      }
    } 

    if (buttonPushed && counter < 600) {
      sendSignal(ALARM_MINUTE_UP);

      buttonPushed = false;
    }

    counter = 0;
  }

  sendSignal(EXIT_ALARM_SETTER); 
}


// UTILITY METHODS


/*
 * Coninually counts up, printing the value of the counter to the console
 */
void countUp() {
  counter++;
  Serial.println(counter);
}

/*
 * Sends a signal to the slave Arduino
 * 
 * @param signalConst The signal constant to be sent to the slave Arduino
 */
void sendSignal(int signalConst) {
  Wire.beginTransmission(2); // transmit to device #2
  Wire.write(signalConst);
  Wire.endTransmission(true);    // stop 
}
