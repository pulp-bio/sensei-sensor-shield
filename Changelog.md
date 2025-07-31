# Changelog

## Version 1.1 (unreleased)
- 

## Version 1.0 (23.09.24)
- Inital Version


## Version 0.2 - PCB Review (02.04.24)
- [General] Switched to an 8 Layer PCB with more GND planes
- [Via] Switched from 0.1/0.3mm vias to 0.2/0.4mm
- [GPS] Added 50 Ohm single-ended impedance matching for antenna
- [Camera] Connect XShutDown and XSleep directly to GAP9
- [Camera] Connect CLK Enable to IO expander instead of directly to XShutDown
- [Camera] Added load switch to separately control 1.8V
- [Camera] Removed the AGND to make the PCB more feasible
- [Connector] Connected VIO to VSys via zero-ohm resistor (UART & I2C)
- [Board] Increase the vertical board size by 1mm (0.5mm on both sides)


### Open comments
#### Julian
> I see in the datasheet, that you might need to sequence XShutDown and XSleep such that the camera can start up. You can access both pins IOs via GPIOs and expander but if you need the camera a lot, it might be worth adding a sequencer.

I want to try to keep the complexity (number of components and "automation") as low as possible. Hence, I connect XShutDown and XSleep directly to GAP9, which allows proper manual sequencing. In case there is some room for optimization, I will tackle this in the second iteration.

> I see the HM0360_Int and HM0360_trigger are directly connected to gap_gpios, I directly connected only HM0360_trigger and sleep, and the interrupt seemed for me to uninteresting (look at the datasheet if the interrupt pin of the camera really is what you think... for me seemed not...)

I am interested in the ALC (Illumination Change) and MD (Motion Detection) features.

> Why do you use the hm0360_shutdown pin to turn on the external oscillator... in the timing sequence of the camera the clock was already running, I'm not sure if this could lead to troubles if you want to repurpose the shutdown pin for turning on the osci, just check please

Good point! I connect CLK Enable to the IO expander instead of directly to XShutDown. As the default IO configuration is high-impedance input, the clock is on by default.

> For the voltage generation, 2V8, 1v8, and 1v2  usually cameras want you to follow the start-up sequence 2v8, 1v8, and 1v2 but in the hm0360 datasheet the timing between them is 0s, meaning it probably isn't a strict requirement, but I don't see a strict boot sequence from your board so I just wanted to mention this.

2.8V and 1.82 are user-controlled, but I added another load switch to separately control 1.8V.

#### Marco Guermandi 
> Lengths don't seem to be matched. They all should be matched in length to, say, 0.1mm.

Thanks, I somehow forgot that. I added a design rule and matched the length in this version.

> The lanes should be impedance controlled. Is the spacing/width ok for the layer stack you are using to have 100 ohm diff, 50 ohm single ended?

Yes, this is correctly configured.

> The plane under the csi2 lanes should be a clean ground plane with no interruptions to guarantee a good flow of the return currents. The csi2 lanes should never cross signal traces on the layer underneath, at most supplies. If you need to break the ground plane you can use stitching caps but not ideal.

I switched to an 8-layer PCB with more GND planes and removed the AGND for the camera (like Julian did in his design). This also simplified routing.

> Also having an additional connector for the csi2 will degrade signal quality. Is it needed? If yes, it would probably be better to have mounting options or the connector might give significant stubs (ideally, you don't want to have any unterminated portions of the trace or it'll lead to reflections). This will be very bad especially for anything you connect on J5 as you'll have very long stubs on your board

Removed stubs and configured the top connector to not-fitted.

> GNSS antenna should have 50 ohm routing and perhaps better clearance. Maybe adding an optional matching network could be nice? e.g. a pi network.

True, I added a single-ended 50-ohm impedance rule. However, the reference design from uBlox does not show a matching circuit, as the module is supposed to require a minimum number of external components. Hence, I assume it is not required.

> I'm not super expert on QVAR, but shouldn't be there electrodes or something? What's the planned use for it (if any?)

Yes, the idea is that the electrodes can be connected via the connector. For example, to measure some moisture levels, rain, or the proximity to some objects.

> The antenna connector is very close to the module. Depending on the shape of the connector on the antenna, I'm not 100% there'll be enough room for it. Maybe double check on the datasheet of the antenna you're planning to use

I slightly increased the board size, and it should not be a problem, as the size of the U.FL plug is usually not larger than the socket itself.

> The 1.2V to the camera seems like a very thin trace (and camera draws quite a bit, the sch says 83 mA?). I think the other supplies look fine but maybe I missed some so please double check.

I did increase the trace width.

> I see the vias are all tented. If you think some signals are worth having some quick access points for debug, leaving some (or even all) without soldermask on could be a good idea for the first release of a board.

I did configure some of the vias to be not tented. However, this excludes vias that are beneath components and the ones where the net is easily accessible on a bigger component.

> The clearance vs the adjacant lanes is quite low, it'll affect the impedance and create discontinuities. A bit more would be better. For some reference on csi2 routing https://www.ti.com/lit/an/spracp4/spracp4.pdf gives ideas on clearances and general routing strategies

Thanks for the great reference. I tried to increase the clearance (0.2mm to GND and 0.4mm to other nets), but there is just not that much space, especially in the connector.


## Version 0.1 - Shematics Review (27.02.24)
- [Camera] Switched from `AXT526124` to `AXE526124`
- [Camera] Swichted from 1nF and 10nF to 100nF and 10uF capacitors
- [IO Expander] Swichted from 1nF and 10nF to 100nF and 10uF decoupling capacitors
- [Connector] Changed external connector from Pico-EZmate to DF52
- [UV Sensor] Connect power of U7
- [UV Sensor] Swichted from 10nF to 100nF decoupling capacitors
- [UV Sensor] Added 100nF decoupling capacitors to U4
- [SCD41] Added 100nF decoupling capacitors to U12
- [SCD41] Added 100nF decoupling capacitors to U13
- [SGP41] Switched to 3V3 power supply
- [SGP41] Added I2C Isolation
- [GPS] Swichted from 1uF to 10uF decoupling capacitors
- [IO Expander] Switched from TCA9539RTWR to PCA6416AEVJ due to smaller package
- [IMU] Added pull-down to INT1 according to DS
- Marked I2C pull-ups as not fitted.
- Added (not fitted) 22uF decoupling caps to 1V8, VD0, VD1, VD2, VA0

### Open comments
> U6: For the IMU, I suggest using SPI. It is easier to handle high sample rates over SPI than I2C.

For now I use I2C and we switch to SPI in the next iteration if required

> U10: QUestions -> I2C/uart: is fine, expose: yes if possible,  VCC_RF + RF_IN, i'm not sure how active antennas work but i have the impression that if you have VCC_RF on the antenna line the device will see no RF signal. If this is not the case, maybe you can have a jumper connecting these lines.

- For now do not add circuitry for an active antenna
- Use TP for UART

> General: If possible, I will add some 1ohm resistors for the most essential devices.

Not sure if this is a good idea due ot the voltage drop

> General: note that I2C_A has fewer devices connected than I2C_B; you could try to balance the number of connected devices.

The idea is that I2C_A can support up to 1MHz


## Tips for Future
- Instaed of using DF52 use board-to-board connector.
- Use low SGP use 1.8V with switch (e.g. TMUX1112) or 3V3 which might improve performance