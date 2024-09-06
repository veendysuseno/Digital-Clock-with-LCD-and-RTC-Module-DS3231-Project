# Digital Clock with LCD and RTC Module DS3231 Project

## Description

This Arduino project integrates a LiquidCrystal display and a DS3231 real-time clock (RTC) module to display the current time and temperature. It also features a time adjustment function via serial communication.

## Components

- **LiquidCrystal LCD**: Displays time and temperature.
- **DS3231 RTC Module**: Keeps track of time and provides temperature readings.

## Features

- **Time Adjustment**: Adjust date and time through serial input.
- **Custom Symbols**: Displays temperature with a custom symbol.

## Code Overview

1. **Setup**:

   - Initializes the LCD and RTC.
   - Defines custom symbols for temperature display.

2. **Loop**:

   - Listens for time adjustment commands.
   - Updates the LCD with the current time and temperature.

3. **Functions**:
   - `adjust_time()`: Prompts user to input new time values.
   - `update_lcd()`: Refreshes the display with the current time and temperature.

## Usage

1. **Adjust Time**: Send `"ADJTIME"` via the serial monitor to start the time adjustment process.
2. **View Data**: The LCD will display the current time and temperature.

For additional details on setup and usage, refer to the inline comments in the code.

## Code

```cpp
#include <LiquidCrystal.h>
#include <DS3231.h>

const int PIN_RS = 2;
const int PIN_E = 3;
const int PIN_D4 = 4;
const int PIN_D5 = 5;
const int PIN_D6 = 6;
const int PIN_D7 = 7;

const String REQ_ADJUST_TIME = "ADJTIME";
const String TIME_ADJUST_STRING[] = { "tanggal", "bulan", "tahun","jam", "menit", "detik" };
const int TIME_BOTTOM_THRESHOLD[] = { 1, 1, 2000, 0, 0, 0 };
const int TIME_UPPER_THRESHOLD[] = { 31, 12, 2099, 23, 59, 59 };

byte temperature_symbol[8] = {
   0b00100,
   0b01010,
   0b01010,
   0b01110,
   0b01110,
   0b11111,
   0b11111,
   0b01110
};

byte degree_symbol[8] = {
   0b01110,
   0b01010,
   0b01110,
   0b00000,
   0b00000,
   0b00000,
   0b00000,
   0b00000
};

const int TEMPERATURE_SYMBOL = 0;
const int DEGREE_SYMBOL = 1;

LiquidCrystal lcd( PIN_RS, PIN_E, PIN_D4, PIN_D5, PIN_D6, PIN_D7 );
DS3231 rtc( SDA, SCL );

void setup(){
   lcd.begin(16,2);
   rtc.begin();
   lcd.createChar( TEMPERATURE_SYMBOL, temperature_symbol );
   lcd.createChar( DEGREE_SYMBOL, degree_symbol );
   Serial.begin(9600);
}

void loop(){
   if( Serial.available() ) {
      String recv = Serial.readString();
      if( recv == REQ_ADJUST_TIME ) {
         adjust_time();
      }
   }

   update_lcd();
}

void adjust_time(){
   int waktu[6];

   for( int i = 0; i < 6; i++ ) {
      while(true)
      {
         String dialog = "Input " + TIME_ADJUST_STRING[i] + " (" +
                        String(TIME_BOTTOM_THRESHOLD[i]) + "~" +
                        String(TIME_UPPER_THRESHOLD[i]) + ") : ";

         Serial.print( dialog );

         while( !Serial.available() ) {
            update_lcd();
         }

         int recv = Serial.parseInt();
         Serial.println( recv );
         if( recv < TIME_BOTTOM_THRESHOLD[i] || recv > TIME_UPPER_THRESHOLD[i] ) {
            Serial.println("[ERROR]: Cek kembali input waktu");
         } else {
            waktu[i] = recv;
            break;
         }
      }
   }

   rtc.setDate( waktu[0], waktu[1], waktu[2] );
   rtc.setTime( waktu[3], waktu[4], waktu[5] );
   Serial.println("> Update waktu berhasil!");
}

void update_lcd(){
   lcd.setCursor( 0, 0 );
   String t = "[" + String(rtc.getTimeStr(FORMAT_LONG)) + "] ";
   lcd.print( t );
   lcd.write( (byte) TEMPERATURE_SYMBOL );
   lcd.print( (int) rtc.getTemp() );
   lcd.write( (byte) DEGREE_SYMBOL );
   lcd.print("C");
   lcd.setCursor( 0, 1 );
   lcd.print("Date: ");
   lcd.print( String(rtc.getDateStr(FORMAT_LONG,FORMAT_LITTLEENDIAN,'-')) );
}
```
