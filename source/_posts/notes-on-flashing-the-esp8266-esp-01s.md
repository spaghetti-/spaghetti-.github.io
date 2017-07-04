---
title: Notes on flashing the ESP8266 wifi module (ESP-01S)
date: 2017-07-04 06:10:36
tags: esp8266
---

Writing this down for future reference. I used a Sparkfun FTDI breakout board
with selectable voltage level set to 3.3v. Pinout

```
TXD <-> TX (I suspect wrong labelling)
RXD <-> RX
VCC <-> 3v3
GND <-> GND
        IO0 pulled low
        IO2 pulled high
```

Keep the reset pin connected to a wire for easy resetting. 

Install `esptool.py` via `pip install esptool` and then when the ESP8266 is set
to flash mode (IO0 low IO2 high) we can then try to connect at 115200 baud.

```
esptool.py --port /dev/tty.usbmodem1421 --baud 115200 read_mac
```

I flashed v2.1.0 of the NON OS SDK which can be obtained
[here](https://github.com/espressif/ESP8266_NONOS_SDK/releases). 

Flash details

```
boot_v1.7.bin @ 0x0
at\512+512\user1.1024.new.2.bin @ 0x01
esp_init_data_default.bin 0xfc000
blank.bin @ 0xfe000 0x7e000 0xfb000 0xfb000
```

SPI crystal frequency of 26MHz, speed 40MHz, 8Mbit flash size, QIO flash.

The firmware sets default baudrate to `1152000`. This was confusing for me because I'm
used to working with serial devices and I just assumed it was `115200` (a more
standard baudrate). Use a serial monitor that can connect on that baudrate and
then change the default by typing

```
AT #for testing
AT+GMR #for testing
AT+UART_DEF=9600,8,1,0,0
```

Now you can use minicom again but you have to remember to press ctrl M ctrl J to
send CRLF like so

```
AT^M^J
```


