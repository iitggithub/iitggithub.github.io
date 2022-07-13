## Logging into my computer with an RFID card

On the list of stupid things to do, this is probably up there. We usually have fairly restrictive password policies in corporate environments and it gets difficult to remember which password I've currently set. It's also quite frustrating when you accidentally type it wrong and get your account locked.

I decided that there had to be a better a way and so I decided to use a spare RFID card as a means to authenticate myself and enter my password. By doing so, I could use a randomly generated password which conforms to any password policy requirements and because the password is compiled into code it remains somewhat secure.

Seriously, don't do this unless it's in a development environment. I am not responsible if someone figures out how to bypass your access control system and gain unauthorised access to your computer.

### Items Needed

1. RFID Scanner such as the [https://www.adafruit.com/product/789](Adafruit PN532 NFC/RFID Controller Shield)
2. [https://core-electronics.com.au/arduino-leonardo.html](Arduino Leonardo with Headers) (or equivalent such as Esplora, Zero, Due and MKR Family boards)
3. [https://core-electronics.com.au/shield-stacking-headers-for-arduino-r3-compatible.html](Stacking header pins) for Arduino
4. [https://core-electronics.com.au/micro-usb-cable.html](Micro USB to USB A cable) (for connecting the Leonardo to the computer)

### Setup

Start by installing the Arduino IDE from [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software).

Download the [](RFID_keyboard.ino) file and open it in the Arduino IDE.

In the Tools menu, choose Board -> Arduino AVR Boards -> Arduino Leonardo. You can also choose a different board if you're not using the Leonardo. Setting this makes sure we're compiling the code to run on the correct microcontroller.

Next, you'll need to select the correct serial port to use to talk to the Leonardo. In the Tools menu, choose Ports and you'll see a list of serial ports. Now plug in your Arduino using the USB cable and check the ports menu again. You see how there's a new option in the Ports menu? That's your Arduino so choose it ( Mine was "/dev/cu.usbmodem2401 (Arduino Leonardo)" ) and let's continue.

Finally, depending on which RFID scanner you end up going with, you'll need to install some additional libraries. For the Adafruit board, you'll need download the [https://github.com/adafruit/Adafruit-PN532](Adafruit PN532 library from github). Uncompress the folder and rename the folder Adafruit\_PN532. Inside the folder you should see the Adafruit\_PN532.cpp and Adafruit\_PN532.h files. Install the Adafruit_PN532 library foler by placing it in your arduinosketchfolder/libraries folder. By default this was in Documents/Arduino/libraries so if the Arduino folder and/or libraries sub folder do not exist, you'll need to create them yourself.

### Generate A Random Password

You can use a random password generator such as the [https://www.lastpass.com/features/password-generator](LastPass Password Generator) to generate a random string which will eventually become your password. Note that you'll want to avoid using a randomly generated password that includes double quotes ( " ) to make the next step easier. You can use them, but you'll need to escape them in the code. I find it's just easier to generate another random password until there's no double quotes in the password. For the purpose of this tutorial, we'll use the random string "2$70$Q^yqZT^Cv#f" (without quotes).

In the example below, we can see the randomString variable has been updated with the randomly generated string generated earlier.

```
const char randomString[] = "2$70$Q^yqZT^Cv#f";    // Your randomly generated string is stored here
const char cardUUIDList[] = {"YOUR_CARD_UUID_HERE"}; // a comma delimeted list of card UUIDs that are permitted
```

NOTE: DO NOT CHANGE YOUR SYSTEM PASSWORD YET.

### Program The Micro Controller

In the Arduino IDE, edit the randomString variable and change the string YOUR\_PASSWORD\_HERE to the randomly generated password you created in the previous step.

Once you've done that, click the Upload button to verify and compile the code and copy it to the arduino. If everything was successful we can connect everything up and do some testing. The reason I tend to program the controller first is that I usually don't have the parts I need to complete the build yet. If you do, you can always leave this bit until the end.

When the controller is programmed, Open the Serial Monitor ( Tools menu -> Serial Monitor ). Now place the RFID card on the scanner and note down the Card UUID.

Edit the cardUUIDList array and change YOUR\_CARD\_UUID\_HERE to the UUID identified in the serial monitor. This makes sure that only the selected list of UUIDs (RFID tags) can trigger the keyboard string to be written out. Since the cardUUIDList variable is an array, you can also provide multiple comma separated UUID's as shown below:

```
const char cardUUIDList[] = {"YOUR_CARD_UUID_HERE","YOUR_OTHER_CARD_UUID_HERE","YOUR_OTHER_OTHER_CARD_UUID_HERE"};
```

Once you've updated the list, you can click Upload to compile and upload the code to your arduino.

### Connect The RFID Shield To The Arduino

Start by soldering the header pins to the Adafruit shield. For a good tutorial on soldering header pins see the video below:

<iframe width="320" height="266" src="https://www.youtube.com/watch?v=Z0joOKaQ43A" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Once you've soldered the header pins, plug the shield into the Arduino making sure it's aligned as shown below otherwise you risk damaging the board, the shield, or both.

![](../images/rfid-login-f89-02.jpg)

### Testing It Works

Open a text editor such as Notepad or TextEdit. It really doesn't matter which one.

Tap your RFID card against the reader and it should type your password into the text editor followed press an Enter key press. If it works, you're done with the hard part, try locking your computer and testing it enters your password and presses the Enter key.

Make sure that the system works when you reboot, startup/shutdown, lock screen etc.

### Change Your Password

Now you've confirmed it works, you can change your computer password to the randomly generated string.

### Tidying up

Edit the Arduino sketch and change the randomString value back to YOUR\_PASSWORD\_HERE. Save the sketch but DO NOT upload it. This ensures no one who comes across your sketch can obtain your password. It also means that if you forget your password, you'll need to reset it.

Thanks for reading everyone!
