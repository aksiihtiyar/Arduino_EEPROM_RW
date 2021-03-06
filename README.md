_This manual is released by the author into the public domain._

#T24C02A EEPROM set-up:
- Connect Arduino GND to EEPROM GND.
- Connect Arduino SDA to EEPROM SDA.
- Connect Arduino SCL to EEPROM SCL.
- Connect Arduino 5V  to EEPROM Vcc.

#Arduino set-up:
1. Open the Arduino IDE.
2. Connect Arduino (I used USB).
3. Choose correct board and serial port in Arduino IDE.
4. Upload code to Arduino using the IDE.
5. Open serial console (Ctrl+Shift+M).
6. The board waits 4 seconds after Arduino reset (end of upload).
7. Then it promps for I²C address.
  * If you do not see this prompt, just enter the address.
  * If you don't know the address, enter the character 'P'.

#Getting started:
If you opened the serial console soon enough, you'll be prompted for an I²C address or a 'P'. If you did not open the serial console within four seconds after the Arduino reset, you will not see the prompt, but the Arduino will still patiently wait for your input.

Enter the address, or the character 'P'.

##Setting the I²C address:
__The Arduino wire library works with 7-bit I²C adresses.__

The Arduino Serial library can easily parse decimal values, not hex codes. Therefore the default T24C02A address 0xA0 is rendered thus:

1. Right-shift one bit to get 7-bit address: 0xA0 >> 1 → 0x50
2. Convert to decimal: 0x50 → 5×16 + 0 = 80

_If you have a T24C04A, T24C08A or T24C16A, you can change addresses later to write to other memory pages._

##Polling the I²C bus:
If you entered the character 'P', the Arduino will poll all 128 possible addresses on the I²C bus.
    It wil show you the address the devices that answered, and their answers.

#The main menu:
Once you have successfully entered an address, you will arrive in the main shell menu.

You will be asked for a command char:

    R — Read
    W — Write
    S — Set I²C address
    P — Poll the I²C bus.
S is useful if you set the I2C address wrong, have multiple EEPROMs or have a T24C04A, T24C08A or T24C16A IC.
##Reading:
1. Follow the instruction:
  - Enter the start address for the read.
  - Enter the number of bytes to be read.
2. The device will print all characters on a single line.
3. Followed by a line of statistics.
  - Optionally followed by a line indicating an error.

##Writing:
1. Follow the instruction:
2. Enter the start address for the write.
3. Enter the string to be written.
  - Escape numerical ASCII representations of characters by prefixing them with an @.
  - @64 is the @ character itself, in case you want to write an actual @.
4. The device will print each character submitted to the device.
5. The device will print a human readable error message for each unsuccessful write.
  - The device will retransmit (including reprint of) the character up to NUM_TRIES times.
  - `NUM_TRIES` is set to 10 in the code by default.

###After ten failed tries:
  - You get an error message indication.
  - The device skips this address.
  - It continues to write the next character to the next location in the EEPROM.

##Setting the I²C address:
The same procedure as detailed in the section "Getting Started", see above.

##Polling the I²C bus:
The same procedure as detailed in the section "Getting Started", see above.

_This polling is not done as part of a change of address. Therefore, the device will give information on the polled devices, and return to the menu immediately afterwards._

#Advanced command usage:
Due to the way the Arduino Serial library works, you need not wait for the prompt to enter the next command. It reads command characters and numerical characters separately as well thus:

`R11 13` will read thirteen bytes starting at EEPROM address 11.

- Note the space between 11 and 13, otherwise the start address will be set to 1113.
- Any non-numerical character should work between the numbers, it need not be a space.

`W25Hey!This is a test@64@` will write the string `Hey!This is a test@@` starting at EEPROM address 25.

- Note that the string is written directly after the start address.
- This method will not work if you want your string to start with a numerical character.
- The ending may be untuitive, but the first @ is treated as an escape character indication that the next character is the numerical 64 (an @ character). The next @ is the last character on the line. The system will check and see that the Serial buffer is empty, and conclude that it is not an escape character, since it is not followed by a number.
  - This only works at the end of the string! Any @ followed by a non-numerical character WILL mess up your write command.

