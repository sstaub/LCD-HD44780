# Arduino LCD library for HD47780

This library is for the HD47780 LCDs with 4-bit interface for use with Arduino.
The most difference to the original LiquidCrystal library from Arduino is that this library supports `printf()` functionality.
When using PlatformIO with AVR, to support `float` you must add following line in platformio.ini

```cpp
#build_flags = -Wl,-u,vfprintf -lprintf_flt -lm
```

## Version 2.x Breaking Changes
- Offset, set the coordinates to 1 as a start point for rows and columns, set `#define OFFSET 0` for old behavior
- New clear functions `clr()` and `clp()`
- Support for non AVR
- It also changes the order of the location inputs, first is the Row followed by the Column.

## Example
Here is a simple example which shows the capabilities of the display 
```cpp
#include "Arduino.h"
#include "LCD.h"

#define LCD_ROWS    2
#define LCD_COLUMNS 16

// for LCD Keypad Shield
#define LCD_RS     8
#define LCD_ENABLE 9
#define LCD_D4     4
#define LCD_D5     5
#define LCD_D6     6
#define LCD_D7     7

// special chars
uint8_t upArrow[8] = {  
    0b00100,
    0b01010,
    0b10001,
    0b00100,
    0b00100,
    0b00100,
    0b00000,
    };

uint8_t downArrow[8] = {
    0b00000,
    0b00100,
    0b00100,
    0b00100,
    0b10001,
    0b01010,
    0b00100,
    };

uint8_t rightArrow[8] = {
    0b00000,
    0b00100,
    0b00010,
    0b11001,
    0b00010,
    0b00100,
    0b00000,
    };

uint8_t leftArrow[8] = {
    0b00000,
    0b00100,
    0b01000,
    0b10011,
    0b01000,
    0b00100,
    0b00000,
    };
    
float data = 0.1f;

LCD lcd(LCD_RS, LCD_ENABLE, LCD_D4, LCD_D5, LCD_D6, LCD_D7); // rs, enable, d4, d5, d6, d7

void setup() {
    lcd.begin(LCD_ROWS, LCD_COLUMNS);
    lcd.create(0, downArrow);
    lcd.create(1, upArrow);
    lcd.create(2, rightArrow);
    lcd.create(3, leftArrow);
    }

void loop() {
    lcd.cls();
    lcd.printf("hello world %f", data);
    // print user chars
    lcd.character(2, 1, 0);
    lcd.character(2, 3, 1);
    lcd.character(2, 5, 2);
    lcd.character(2, 7, 3);

    delay(1000);
    lcd.display(DISPLAY_OFF);
    delay(1000);
    lcd.display(DISPLAY_ON);
    delay(1000);
    lcd.display(CURSOR_ON);
    delay(1000);
    lcd.display(BLINK_ON);
    delay(1000);
    lcd.display(BLINK_OFF);
    delay(1000);
    lcd.display(CURSOR_OFF);
    delay(1000);
    // scroll 16 positions to the right
    for (uint8_t pos = 0; pos < 16; pos++) {
        lcd.display(SCROLL_RIGHT); // scroll one position to right
        delay(500); // step time
        }
    // scroll 16 positions to the left
    for (uint8_t pos = 0; pos < 16; pos++) {
        lcd.display(SCROLL_LEFT); // scroll one position to left
        delay(500); // step time
        }
    delay(500);
    lcd.cls();
    lcd.locate(1, 1);
    lcd.printf("hello line 1");
    delay(500);
    lcd.locate(2, 1);
    lcd.printf("hello line 2");
    delay(500);
    // clear parts and rows
    lcd.clp(1, 1, 3);
    delay(500);
    lcd.clp(2, 4, 3);
    delay(500);
    lcd.clp(1, 6, 3);
    delay(500);
    lcd.clp(2, 10, 3);
    delay(500);
    lcd.clr(1);
    delay(500);
    lcd.clr(2);
    delay(500);
    }
```

# Documentation

## Constructor
```cpp
LCD(uint8_t rs, uint8_t enable, uint8_t d4, uint8_t d5, uint8_t d6, uint8_t d7)
```
- **rs** reset pin
- **enable** enable pin
- **d4** d4 pin
- **d5** d5 pin
- **d6** d6 pin
- **d7** d7 pin

Create a LCD object.

**Example**
```cpp
// for LCD Keypad Shield
#define LCD_RS     8
#define LCD_ENABLE 9
#define LCD_D4     4
#define LCD_D5     5
#define LCD_D6     6
#define LCD_D7     7
LCD lcd(LCD_RS, LCD_ENABLE, LCD_D4, LCD_D5, LCD_D6, LCD_D7); // rs, enable, d4, d5, d6, d7
```

## Methods

### **begin()**
```cpp
void begin(uint8_t rows = 2, uint8_t columns = 16, uint8_t dotsize = LCD_5x8DOTS)
```
- **rows** number of rows, default 2
- **columns** number of columns, default 16
- **dotsize** default LCD_5x8DOTS some displays have also LCD_5x10DOTS

Initialize the display, this must done in `setup()`.

**Example**
```cpp
lcd.begin(4, 20); // for 4x20 display
```

### **cls()**
```cpp
void cls()
```

Clears the display, set cursor to home.

**Example**
```cpp
lcd.cls();
```

### **clr()**
```cpp
void cls(uint8_t row)
```
- **row** set the row to clear

Clears a single row (line), set cursor position of the row to the first position.

**Example**
```cpp
lcd.clr(2); // clear row 2
```

### **clp()**
```cpp
void cls(uint8_t row, uint8_t column, uint8_t numbers)
```
- **row** set the row to clear
- **column** set the start column to clear
- **numbers** set the numbers of chars to clear

Clears a part of a row (line), set cursor position of the row to first column.

**Example**
```cpp
lcd.clp(1, 3, 4); // clears in row 1 four chars beginning with column 3
```

### **locate()**
```cpp
void locate(uint8_t row, uint8_t column)
```
- **row** starts with 1
- **column** starts with 1

Set position of the cursor, you have to set first row then the column<br> 
This different to the setCursor() methods of other Arduino libraries!!!

**Example**
```cpp
lcd.locate(2, 15); // move the cursor to the second row at column 15
```

### **home()**
```cpp
void home()
```

Set cursor to home position (1/1)

```cpp
lcd.home();
```

### **display()**
```cpp
void display(dispmode_t mode);
void display(dispmode_t mode, uint8_t value);
```

- **mode** display function you want to use
- **value** set additional value, e.g. for contrast

Set the modes and functions of the display.

- DISPLAY_ON Turn the display on
- DISPLAY_OFF Turn the display off
- CURSOR_ON Turns the underline cursor on
- CURSOR_OFF Turns the underline cursor off
- BLINK_ON Turn the blinking cursor on
- BLINK_OFF Turn the blinking cursor off
- SCROLL_LEFT These command scroll the display without changing the RAM
- SCROLL_RIGHT These commands scroll the display without changing the RAM
- LEFT_TO_RIGHT This is for text that flows Left to Right
- RIGHT_TO_LEFT This is for text that flows Right to Left
- AUTOSCROLL_ON This will 'right justify' text from the cursor
- AUTOSCROLL_OFF This will 'left justify' text from the cursor

**Example**
```cpp
lcd.display(CURSOR_ON); // show the cursor
lcd.display(BLINK_ON); // blink the cursor
```

### **character()**
```cpp
void character(uint8_t column, uint8_t row, uint8_t c)
```
- **row** row
- **column** column
- **c** single char

Draw a single character on given position, usefull for user chars.

**Example**
```cpp
lcd.character(1, 3, 'A'); // print char A on row 1, column 3
```

### **create()**
```cpp
void create(uint8_t location, uint8_t charmap[])
```
- **location** the location in the CGRAM (0 ... 7)
- **charmap[]** set additional value, e.g. for contrast

Create a custom character (glyph) for use on the LCD, this must done in `setup()`.<br>
Up to eight characters of 5x8 pixels are supported (numbered 0 ... 7).<br>
The appearance of each custom character is specified by an array of eight bytes, one for each row.<br>
The five least significant bits of each byte determine the pixels in that row.<br>
To display a custom character on the screen use write(n) or print('\n') its number.<br>

**Example**
```cpp
uint8_t smiley[8] = {
    0b00000,
    0b00000,
    0b01010,
    0b00000,
    0b00000,
    0b10001,
    0b01110,
    0b00000
    };

uint8_t frownie[8] = {
    0b00000,
    0b00000,
    0b01010,
    0b00000,
    0b00000,
    0b00000,
    0b01110,
    0b10001
    };

// in setup()
lcd.create(0, smiley);
lcd.create(1, frownie);

// to display the glyphs use print() or write()
lcd.locate(1, 1);
lcd.print('\0');
lcd.locate(1, 3);
lcd.write(1);
```

## **printf()**
```cpp
void printf(const char *format, ...)
```

Print  a formatted text, have a look to the C/C++ documentation.