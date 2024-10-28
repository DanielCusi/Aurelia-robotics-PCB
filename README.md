# Aurèlia power and control board for Eurobot

This repository contains the power and control board used by Wolvineers in Eurobot. This board handles the power input from a battery or USB-PD, converting it for powering all the robot elements; hosts the microcontrollers for the real time control, and has lots of connectors to power and control all the electronics.

Clone using recursive to also include the ESP-mini repository.

## Warning! It has problems to fix! It's just the first version.

This board is just the first version, and has problems to solve so, if you want to build one, I recommend to fix the most important ones before.

I'll try to fix them as soon as I can, but I can't specify any date. If you fix an error, please contact me and I'll add it.

### Serious errors

These are the most important errors, that affect functionality. These should be solved ASAP.

- [ ] USB PD not working. If the PD IC is added, the board starts powering up and down cyclically. My guess is that it is a problem with the 3,3 V vin, that shouldn't be connected to the board 3,3 V, or with the ideal diodes. I'll try to make a standalone PCB to test it. If desired, the chip and its components could be removed for now.

- [ ] When disabling power to the actuators, it disables the USB power but not half of the servos. This can be easily solved by checking the enable inputs, and removing the EM_BTN input from the USB buck and adding it to the servo buck.

- [ ] The gpio expander interrupt pull-up is connected to its supply voltage. Thus, setting the expander to 5 V sets the interrupt pin to 5 V, and it may damage the GPIO if it can't handle 5 V. Connecting it to 3,3 V or adding a voltage selector would solve it.

### Medium errors

- [ ] The motor driver mode selection resistors have their names wrong in the PCB silkscreen. Just change them.

- [ ] The encoder names in the schematic are wrong. Each motor is a letter, and for each the inputs and outputs are numbered. But it is inverted for the encoder, just an old mistake not fixed.

- [ ] The BJT transistor footprint pinout doesn't exist. It should be changed with the standard one, so change the simulation symbol with the standard ones.

- [ ] VBat reading is not x24.9, but x25.9. Either change resistor values, or change the annotation.

- [ ] The battery switch BTS7006 has a tiny ground plane, it should be added one on its bottom and connect it to the PAD. If not added, the IC could get too hot, specially in hot environments and with high power demand.

- [ ] LEDs have different brightness, although I tried to set different resistor values to avoid this. They should be checked.

- [ ] Add pull-down to emergency button input to avoid floating state.

### Improvements

- [ ] Add pin headers to the LED outputs for external indication. This is difficult to do with the limited space, but doable with the following improvement.

- [ ] Use USB-C instead of USB-A for the output. This would reduce both costs and space used in the board.

- [ ] Change the servo pin-out with the standard one. It was done on an older board like this to prevent direct connection without using JST, which may be kept for laziness and may get disconnected in a critical moment. This doesn't make sense with the 2 mm JST which avoid it, and it's just confusing.

- [ ] Add input capacitor to the temperature sensor, and output filter. It would improve performance, both from sensor, and the noisy ESP ADC. Check the TMP235 datasheet.

- [ ] Briefly describe what each ESP does in silkscreen (1 motors 2 servos and sensors) to avoid confusions.

- [ ] Find out why TPS563300 encoders sometimes fail. It seems mostly to fail after being soldered, and not in the long term, but maybe they need ESD protection.

- [ ] Improve silkscreen pinout description. It was a bit rushed, so it can be improved.

- [ ] Add filters for the USB-C to avoid current spikes that might turn it off.

- [ ] Add a connection between ESP32 UARTs for comunication, selectable via jumpers with male headers.

- [ ] Add 100 nF capacitor on the bottom of the ADC connector.

- [ ] Set 0,0 at the corner, for some reason I didn't.

- [ ] Translate to English. It's a lot of work, so I might leave it in Catalan.

## About the board

This board has been designed for Eurobot, for the main robot, and is an improvement from last year board.

It was designed for easy repair, as all component's pads can be accessed by a soldering iron (except USB-C PD IC) for easy replacement. Also, all connectors have good grip to avoid disconnections. And, finally, it is designed for robots, having all necessary connectors and power built in, without being too expensive (~50€ if building just one).

### Power supply

The board is mainly powered by a battery, which can be disconnected with an input, the power switch; and also disconnects if voltage drops too much. It is designed for a 4 cells lithium battery, but as long as it supplies more than 12 V, and less than 20, it should be fine. 5 cells would work, but it would had higher consumption because of the ESD diode, but the bucks can work with 28 V.

It can also be powered by a USB-C with power delivery or PD. But, as it may ask to much current, it is mostly for testing while batteries are charging or when using low power motors.

In order to charge the battery, an external charger must be used, as it would add too much complexity and cost to this board.

### Power control

Battery power can be switched using the switch input in the 2 pin connector. Short it to allow the power flow, or leave it open to stop it. It doesn't affect the USB-C input, and if voltage is too low (about 9,8 V) it disconnects completely (11,4 V to restore), though your mileage may vary from the BTS7006 high variance. Already at 11,7 V the motor bucks disconnect, so don't go at the limit.

There is also an emergency button input, that should be used with a normally closed button. When connected to 3,3 V, all power is active, but at GND it disconnects the actuators, only leaving the 3,3 V and USB output power actives. (For now, leaving it floating doesn't disable it, it has to be fixed.)

### Control

This board can control everything itself, although an external computer like raspberry pi may be used for more processing power, which can be powered with the USB.

There are two sockets for the microcontrollers. They are designed for ESP32-mini - check the repository in libraries directory -, but another microcontroller can be used if a board with the same dimensions and pinout is made.

As we are using a raspberry pi, they don't communicate between each other and the raspberry pi is the one who sends the orders. If the connection is desired, wireless connection can be used, or UART. Although the second one is not available for now, it can be done with jumper cables, just pay attention when programming.

### Motors

The motors are driven by a h-bridge, that allows them to be driven in both directions. They can go up to 10 A peak, so they shouldn't get too hot.

They are supplied by 12 V through a buck, so they always perform the same no matter the main supply voltage. For each two motors (1-2 and 3-4) there is a 5 A buck, so take it into account if they might use all the power.

The encoder inputs may be push-pull, open collector or whatever. The connector supplies 5 V or 3,3 V depending on a jumper position. Just be careful with the first encoder, the second input needs to be high when powering up when using ESP mini board.

### Servos

There can be up to 8 servos at the same time, even more if some other outputs are misused. Each 4 servos (1-4 and 5-8) have a 3 A 5 V buck, but it is shared with other outputs, so try not to run it at the limit.

### Miscellaneous outputs

There are some outputs on the south of the PCB:

- LED: a 5 V output with a signal pin, for WS2812B led strips or other one wire strips.

- VENT: fan outputs, they can handle more current as they use a BJT transistor instead of a GPIO directly, the current depends on the transistor but it is about 200 mA. They are a 5 V pin and an open collector output. They were designed for cooling fans, but they aren't necessary.

- LIDAR: a connector for a simple LIDAR. If there is no budget for a LIDAR, or USB ones can't be used, this LIDAR connector can be used for a DIY solution. With a 5 V motor and encoder to control a small motor, and a time of flight sensor module, a DIY lidar can be used. The sensor may be connected to I2C.

- ALM: power, just power outputs for whatever use is desired, just be careful not to use too much power.

- ADC: Two pins for ADC inputs. The maximum ADC input for a ESP32 is 3 V, not 5 V.

- I2C: Connectors to attach I2C slaves. All I2C works at 3,3 V, although it can be changed if the board pull-ups aren't used and the GPIO expander IC is removed.

### IO-Expander

The IO-expander is based on TCA6408A IC, it uses I2C for communication and has 8 GPIOs. Each of these GPIOs can be used as inputs or outputs, and the inputs have a shared interruption to avoid polling. All of them must be configured at 5 or 3,3 V with a header.

The IC was chosen due to its variable voltage and its register simplicity, although it can't configure internal pull-ups nor pull-downs. There are external ones, which can be chosen with jumpers.


### Breakout pins

The last connector is a pin header, preferably 90 degrees, that can be used for testing. It privides access to 4 servo pins (although may be used as GPIOs), and I2C. It also has 3,3 V and 5 V output.

### Physical dimensions and holes

The board is 120 * 94 mm, and has three M3 holes for mounting. These are, from bottom left (x,y), at (8, 18) mm, (55 , 88) mm, and (114, 89) mm. The height depends on the microcontroller used, if using male-female pins it will be higher than using male pins. Also, take into account that the bottom pins of THT components take space.

## Firmware

I'll add the firmware repositories on the future and link them here.