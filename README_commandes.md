
# Installation des dependances pour DHT sur RPI

## Procedure

Instructions basé sur 
https://learn.adafruit.com/circuitpython-on-raspberrypi-linux/installing-circuitpython-on-raspberry-pi

Appliqué sur DietPi et non pas RaspPi Linux distribution

### Adafruit base

* sudo apt-get update
* sudo apt-get upgrade
* sudo apt-get install python3-pip
* sudo pip3 install --upgrade setuptools

* cd ~
* sudo pip3 install --upgrade adafruit-python-shell
* wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/raspi-blinka.py
* sudo python3 raspi-blinka.py


### Extensions pour le faire fonctionner en Digital Io

#### Paquet historique
* sudo pip3 install RPi.GPIO
* sudo apt install python3-libgpiod gpiod

#### Alternative non focntionnel 06/02/2021
* sudo pip3 install RPi.GPIO2
* sudo apt install python3-libgpiod gpiod

### Tests pour voir si fonctionnel

* Utilisation de la distribution DietPi 

Section I2C et SPI doivent etre activé si distribution non Rpi

```python
import board
import digitalio
import busio

print("Hello blinka!")

# Try to great a Digital input
pin = digitalio.DigitalInOut(board.D4)
print("Digital IO ok!")

# Try to create an I2C device
i2c = busio.I2C(board.SCL, board.SDA)
print("I2C ok!")

# Try to create an SPI device
spi = busio.SPI(board.SCLK, board.MOSI, board.MISO)
print("SPI ok!")

print("done!")
```

### Adafruit DHT

https://learn.adafruit.com/dht-humidity-sensing-on-raspberry-pi-with-gdocs-logging/python-setup

* pip3 install adafruit-circuitpython-dht
* sudo apt-get install libgpiod2


Le 06/02/2021 le test n'est pas concluant mais revoir la connection


#### Investigations

* Test a nouveau sur orange pi et tout fonctionne
* Test4.py avec appel direct aux 2 method
* Identifié une anomalie qui peut etre liée
  * https://github.com/adafruit/Adafruit_CircuitPython_DHT/issues/51
  * apt install file
  * file /usr/local/lib/python3.7/dist-packages/adafruit_blinka/microcontroller/bcm283x/pulseio/libgpiod_pulsein
  /usr/local/lib/python3.7/dist-packages/adafruit_blinka/microcontroller/bcm283x/pulseio/libgpiod_pulsein: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, BuildID[sha1]=356bb870cf3f5a35b0862d12a708b782a9c365f6, not stripped
  * A recompiler comme sur le ticket
sudo apt install libgpiod-dev git build-essential
$ git clone https://github.com/adafruit/libgpiod_pulsein.git
$ cd libgpiod_pulsein/src
$ make
$ cp libgpiod_pulsein ~/.local/lib/python3.8/site-packages/adafruit_blinka/microcontroller/bcm283x/pulseio/libgpiod_pulsein
sudo cp libgpiod_pulsein /usr/local/lib/python3.7/dist-packages/adafruit_blinka/microcontroller/bcm283x/pulseio/libgpiod_pulsein


* Test effectué et meme anomalie car compile seulement en ELF32

* test4.py
```python

# SPDX-FileCopyrightText: 2021 ladyada for Adafruit Industries
# SPDX-License-Identifier: MIT

import time
import board
import adafruit_dht

# Initial the dht device, with data pin connected to:
dhtDevice = adafruit_dht.DHT22(board.D4)

# you can pass DHT22 use_pulseio=False if you wouldn't like to use pulseio.
# This may be necessary on a Linux single board computer like the Raspberry Pi,
# but it will not work in CircuitPython.
# dhtDevice = adafruit_dht.DHT22(board.D4, use_pulseio=False)

while True:
    try:
         # Print the values to the serial port
#        pulses = dhtDevice._get_pulses_bitbang()
        pulses = dhtDevice._get_pulses_pulseio()
        print(pulses)
#        dhtDevice.measure()

#        temperature_c = dhtDevice.temperature
#        temperature_f = temperature_c * (9 / 5) + 32
#        humidity = dhtDevice.humidity
#        print(
#            "Temp: {:.1f} F / {:.1f} C    Humidity: {}% ".format(
#                temperature_f, temperature_c, humidity
#            )
#        )

    except RuntimeError as error:
        # Errors happen fairly often, DHT's are hard to read, just keep going
        print(error.args[0])
        time.sleep(2.0)
        continue
    except Exception as error:
        dhtDevice.exit()
        raise error

    time.sleep(2.0)

```

* Décision repasser sur RpiOs le temps de tester seulement DHT et retour sur DietPi plus tard

* Installation Rpi OS
* Deploiement de la nouvelle approche Adafruit
* Toujours non fonctionnel mais il récupère des valeurs qui ne sont pas lisibles
* Test DHT11 et tjours pb
* Installation de l'ancienne version avec Python 3
  * Fonctionne avec DHT 11 mais rien avec DHT 22

* Refaire le cablage en diminuant la distance pour evaluer si les rallonges ont un impact

Sinon tests avec code C qui va etre plus preci


* Pour plus tard code source en C
  * https://github.com/Seeed-Studio/Grove-RaspberryPi/blob/master/Grove%20-%20Temperature%20and%20Humidity%20Sensor%20Pro/dht22.c
