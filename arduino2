// ARDUINO 2 CODE
// SIGNAL RECEIVER

// Library Inclusions
#include <SoftwareSerial.h>
#include <Wire.h> // For master/slave

/* PORT CONSTANTS */

const int SPEAKER_1 = 11;

// Motor values
const int MA1=2;
const int MA2=4;
const int PWMA=3;

const int LCD_PORT = 5;

const int GREEN_LIGHT = 7;
const int YELLOW_LIGHT = 8;
const int RED_LIGHT = 9;
const int BLUE_LIGHT = 10;

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

int alarmH = 12;
int alarmM = 1;
String alarmTimeOfDay = "am";
boolean alarmOn = true;
boolean alarmPlaying = false;
int alarmNum = 1;

int h = 0;
int m = 0;
int s = 51;
String timeOfDay = "am";

int methodSignal = 0;

// Initialization of LCD
SoftwareSerial LCD(6, LCD_PORT); // Arduino SS_RX = pin 6 (unused), Arduino SS_TX = pin 5 



/* 
 *  Runs once when the Arduino is first powered
 */
void setup() {
  // Initiates Slave Arduino
  Wire.begin(2);              
  Wire.onReceive(receiveEvent); // register event
  Serial.begin(9600);           // start serial for output

  // Initializes LED outputs
  pinMode(GREEN_LIGHT, OUTPUT);
  pinMode(YELLOW_LIGHT, OUTPUT);
  pinMode(RED_LIGHT, OUTPUT);
  pinMode(BLUE_LIGHT, OUTPUT);

  // Initializes motor outputs
  pinMode(MA1,OUTPUT);
  pinMode(MA2,OUTPUT);
  pinMode(PWMA,OUTPUT);

  // Initializes LCD
  LCD.begin(9600); // set up serial port for 9600 baud
  delay(500); // wait for display to boot 

  // Clears LCD screen
  LCD.write("                ");
  LCD.write("                ");

  // Stays in loop until universal on signal is given
  while(methodSignal != UNIVERSAL_ON) {
    receiveEvent(1);
    Serial.print("STUCK!   ");
    Serial.println(methodSignal);
  }

  // Sets the clock time to start
  setTime(12, 0);
}

/*
 * Loops continuously while the Arduino is on
 */
void loop() {
  receiveEvent(1);

  // DEBUGGING: Prints the most recent signal received
  Serial.println(methodSignal);

  // Waits for Time/Alarm Setter signals
  if(methodSignal == TIME_SETTER && !alarmPlaying) {
    timeSetterMode();
  } else if (methodSignal == ALARM_SETTER) {
    alarmSetterMode();
  }

  // Continually increments the time and checks to see if the alarm should play
  timeTeller();
  alarm();
}


/* UTILITY METHODS */


/*
 * Receieves values sent from the Master Ardunio and subsequently assigns them to the
 * methodSignal variable. Also checks for certain signals and responds accordingly.
 * 
 * @param howMany How many events are to be receieved, needed to designate receieveEvent as
 * the Slave Arduino's processor of signals.
 */
void receiveEvent(int howMany) {
  // Loops while there are still signals available
  while(0 < Wire.available()) {
      methodSignal = Wire.read();
      
      if(methodSignal == HOUR_UP && !alarmPlaying) {
        h++;

        // Checks to see if the time of day (am/pm) needs to be changed
        checkTimeOfDay();
      } else if (methodSignal == MINUTE_UP && !alarmPlaying) {
        m++;
      } else if (methodSignal == ALARM_HOUR_UP && !alarmPlaying) {
        alarmH++;
        
        // Checks to see if the time of day (am/pm) needs to be changed
        checkAlarmTimeOfDay();
      } else if (methodSignal == ALARM_MINUTE_UP && !alarmPlaying) {
        alarmM++;
      } else if (methodSignal == ALARM_TRIGGER && !alarmPlaying) {
        if (alarmOn) {
          alarmOn = false;
        } else {
          alarmOn = true;
        }
      // Turns the alarm off both if "SNOOZE" is receieved and the "ALARM TRIGGER" signal is received
      // while the alarm is playing (which is sent by the same button as the snooze).
      } else if (methodSignal == SNOOZE || (methodSignal == ALARM_TRIGGER && alarmPlaying)) {
        alarmOff();
      } else if (methodSignal == MOTORS_OFF) {
        motorsOff();
      }
  }

  // Always checks to see if the time values need to be updated after receiving a signal
  checkTime();
  checkAlarmTime();
}


/* SETTER MODES */


/*
 * The mode in which the Arduino is awaiting signals specifically for time setting purposes. 
 */
void timeSetterMode() {
  // Seconds automatically reset when the time is changed
  s = 0;
  
  boolean switchMethod = false;
  Serial.println("WOOHOO!");

  /* HOUR CHANGE */
  
  while(!switchMethod) {
    
    receiveEvent(1);
    displayTime("timeSetterMode");

    if(methodSignal == HOUR_TO_MIN) {
      resetLCD();
      switchMethod = true;
    }
  }

  switchMethod = false;

  /* MINUTE CHANGE */

  while(!switchMethod) {
    receiveEvent(1);

    displayTime("timeSetterMode");

    if(methodSignal == EXIT_TIME_SETTER){
      resetLCD();
      switchMethod = true;
    }
  }
}

/*
 * The mode in which the Arduino is awaiting signals specifically for alarm time setting purposes. 
 */
void alarmSetterMode() {;
  int startTime = millis();
  Serial.println(startTime);
  
  boolean switchMethod = false;
  Serial.println("WOOHOO!");

  /* HOUR CHANGE */  

  while(!switchMethod) {
    
    receiveEvent(1);
    
    displayAlarm("alarmSetterMode");

    if(methodSignal == ALARM_HOUR_TO_MIN) {
      resetLCD();
      switchMethod = true;
    }
  }

  switchMethod = false;

  /* MINUTE CHANGE */

  while(!switchMethod) {
    receiveEvent(1);

    displayAlarm("alarmSetterMode");

    if(methodSignal == EXIT_ALARM_SETTER){
      resetLCD();
      switchMethod = true;
    }
  }

  int endTime = millis();
  Serial.println(endTime);  

  s = s + (endTime - startTime) / 1000;
  if(s >= 60) {
    s -= 60;
    m++; 
    checkTime();
  }
}


/* TIME METHODS */


/*
 * THE TELLER OF TIME. Universal incrementor of the seconds variable every second of time passed.
 */
void timeTeller() {
  // DEBUGGING: Prints the time to the console
  Serial.print("Hours: ");
  Serial.print(h);
  Serial.print(" Minutes: ");
  Serial.print(m);
  Serial.print(" Seconds: ");
  Serial.println(s);
  displayTime("");

  // Waits 1 second before incrementing the seconds variable. Processing the rest of the code takes additional
  // time, which is accounted for in the 950ms delay (rather than 1000ms)
  delay(950);
  s++;

  // Checks to update the time if necessary
  checkTime();
}

/* 
 *  Checks seconds, minutes and hours variables to see if they have reached their 
 *  maximum values and subsequently updates them while incrementing their cooresponding 
 *  values (e.g., if there are 60 seconds, minutes increment by one and seconds is reset
 *  to zero)
 */
void checkTime() {
  if(s == 60) {
    m++;
    s = 0;
  }

  if(m == 60) {
    h++;

    // Checks to see if the time of day (am/pm) needs to be changed
    checkTimeOfDay();
    m = 0;
  }

  if(h == 13) {
    h = 1;
  }
}

/*
 * Sets the hours and minutes global variables to the parameter values
 * 
 * @param hours Current hours value
 * @param minutes Current minutes value
 */
void setTime(int hours, int mins) {
  h = hours;
  m = mins;
}

/*
 * Checks to see if the hours variable is equal to twelve, and if so, the time is switched
 * from "am" to "pm" or vice versa. Always called after the hours are incremented.
 */
void checkTimeOfDay() {
  if(h == 12) {
    if (timeOfDay == "am") {
      timeOfDay = "pm";
    } else {
      timeOfDay = "am";
    }
  }
}


/* ALARM METHODS */


/*
 * Checks to see if the time is equal to the alarm time, activates the alarm if so.
 */
void alarm() {
  // Checks if the alarm is on and all the time variables are equivalent
  if(alarmOn && alarmH == h && alarmM == m && s == 0) {
    resetLCD();
    alarmPlaying = true;
    motorsOn();

    // LED system cooresponding to what number alarm is playing
    if(alarmNum == 1) {
      digitalWrite(GREEN_LIGHT, HIGH);
    } else if (alarmNum == 2) {
      digitalWrite(YELLOW_LIGHT, HIGH);
    } else if (alarmNum == 3) {
      digitalWrite(RED_LIGHT, HIGH);
    }
  } 

  // Plays the speaker every other second
  if(alarmPlaying) {
    if(s % 2 == 0) {
      tone(SPEAKER_1, 2000, 1000);
    } 
  }
}

/* 
 *  Checks alarm seconds, minutes and hours variables to see if they have reached their 
 *  maximum values and subsequently updates them while incrementing their cooresponding 
 *  values (e.g., if there are 60 seconds, minutes increment by one and seconds is reset
 *  to zero).
 */
void checkAlarmTime() {
  if(alarmM == 60) {
    alarmH++;

    // Checks to see if the time of day (am/pm) needs to be changed
    checkAlarmTimeOfDay();
    alarmM = 0;
  }

  if(alarmH == 13) {
    alarmH = 1;
  }
}

/*
 * Checks to see if the alarm hours variable is equal to twelve, and if so, the alarm time is 
 * switched from "am" to "pm" or vice versa. Always called after the alarm hours are incremented.
 */
void checkAlarmTimeOfDay() {
  if(alarmH == 12) {
    if (alarmTimeOfDay == "am") {
      alarmTimeOfDay = "pm";
    } else {
      alarmTimeOfDay = "am";
    }
  }
}

/* 
 *  Turns all the functions of the alarm off.
 */
void alarmOff() {
  if(alarmPlaying) {
    // SNOOZE: resets the alarm for one minute later
    alarmM++;
    if(alarmM >= 60) {
      alarmM = alarmM - 60;
      alarmH++;
      checkAlarmTimeOfDay();
  
      if(alarmH == 13) {
        alarmH = 1;
      }
    }
    
    alarmPlaying = false;
    motorsOff();

    // Turns off any LEDs that were turned on
    digitalWrite(RED_LIGHT, LOW);
    digitalWrite(YELLOW_LIGHT, LOW);
    digitalWrite(GREEN_LIGHT, LOW);
    digitalWrite(BLUE_LIGHT, LOW);

    // Resets the alarm number back to one if the alarm cycle has run through all three options
    if(alarmNum == 3) {
      alarmNum = 1;
    } else {
      alarmNum++;
    }
  }
}


/* MOTOR METHODS */


/* 
 *  Turns on the necessary ports to move the motors.
 */
void motorsOn() {
  digitalWrite(MA1,HIGH);
  digitalWrite(MA2,LOW);
  analogWrite(PWMA,255);
}

/* 
 *  Turns off the necessary ports to stop the motors.
 */
void motorsOff() {
  digitalWrite(MA1,LOW);
  analogWrite(PWMA,0);  
}


/* LCD METHODS */


/*
 * Clears the LCD display by writing blank lines to the display
 */
void clearLCD() {
  LCD.write("                ");
  LCD.write("                ");
}

/*
 * Moves the LCD cursor to the first position of the first line
 */
void toFirstLine() {
  LCD.write(254); 
  LCD.write(128);
}

/*
 * Moves the LCD cursor to the first position of the second line
 */
void toSecondLine() {
  LCD.write(254);
  LCD.write(192);
}

/*
 * Resets the LCD by clearing the display and moving the cursor to the first spot of the first line
 */
void resetLCD() {
  clearLCD();
  toFirstLine();
}

/**
 * Writes the current time on the LCD display. 
 * 
 * @param method The method calling the displayTime method
 */
void displayTime(String method) {
  char hourString[10], minString[10], secString[10], todString[10];

  // Each integer time value must be converted to a string to display on the LCD screen.
  sprintf(hourString,"%2d",h);
  sprintf(minString,"%2d",m);
  sprintf(secString,"%2d",s);
  sprintf(todString,"%2d",timeOfDay);

  resetLCD();
  LCD.write(hourString);
  LCD.write(":");
  LCD.write(minString);
  LCD.write(":");
  LCD.write(secString);
  if(timeOfDay == "am") {
    LCD.write(" am");
  } else {
    LCD.write(" pm");
  }

  // Special display for Time Setter Mode, helps the user identify whether they
  // are editing the hours or the minutes of the time. 
  if(method == "timeSetterMode") {
    toSecondLine();
    if(methodSignal == 1 || methodSignal == 2) {
      LCD.write("Editing Hours...");
    } else if (methodSignal == 3 || methodSignal == 4) {
      LCD.write("Editing Minutes...");
    }
  }

  // Special display for when the alarm if the alarm is on
  if(alarmOn && method != "timeSetterMode") {
    toSecondLine();
    if(alarmPlaying) {
      LCD.write("ALARM PLAYING");
    } else {
      LCD.write("           ");
      if(alarmOn) {
        LCD.write("*");
      } else {
        LCD.write(" ");
      }

      displayAlarm("displayTime");
    }
  }
}

/* 
 * Writes the current alarm time on the LCD display. 
 * 
 * @param method The method calling the displayAlarm method
 */
void displayAlarm(String method) {
  char hourString[10], minString[10], secString[10];

  // Each integer alarm time value must be converted to a string to display on the LCD screen.
  sprintf(hourString,"%2d",alarmH); 
  sprintf(minString,"%2d",alarmM);

  if(method != "displayTime") {
    resetLCD();
  }
  LCD.write(hourString);
  LCD.write(":");
  LCD.write(minString);
  if(alarmTimeOfDay == "am") {
    LCD.write(" am");
  } else {
    LCD.write(" pm");
  }
  LCD.write("    ");

  // Special display for if the alarm time is being edited in Alarm Setter mode
  if(method == "alarmSetterMode") {
    toSecondLine();
    if(methodSignal == 6 || methodSignal == 7) {
      LCD.write("Editing Hours...");
    } else if (methodSignal == 8 || methodSignal == 9) {
      LCD.write("Editing Minutes...");
    }
  }
}
