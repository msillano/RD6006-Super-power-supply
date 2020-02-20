# RD6006-Super-power-supply
A intelligent Power supply with PC control and logging, for very long test (battery cycles)
## Owerview
The RD6006-W power supply is a fantastic professional tool, providing up to 60V, 6A (360 Watt), very complete with high level functions. It is also possible to control it from the front panel, or via WIFI with an APP from a smartphone, or via USB with a software on PC WIN. See [https://it.aliexpress.com/item/4000282551930.html](https://it.aliexpress.com/item/4000282551930.html "RD official store")

![RD6006](images/2020-02-17.081154.shot.png)

In this period I am interested in rechargeable batteries (see [https://github.com/msillano/e3DHW-PMS](https://github.com/msillano/e3DHW-PMS "e3DHW power management system")), and I have immediately thought of using this splendid hardware.
Although very complete, the supplied software is generic, not aimed at my purposes. In particular:

- Only Excel export, and limited to 24h (with APP, with WIN it is not defined).
- It does not export the battery voltage nor the battery temperature.
- Possibility of programming only based on time, and not on other trigger events

See also:

- [https://github.com/rfinnie/rdserialtool](https://github.com/rfinnie/rdserialtool)  CLI in Python and  [https://github.com/Black-FX/rdserialtool](https://github.com/Black-FX/rdserialtool) UI extension.
 
## ModBus test RD6006
After finding sufficient information on the MODBUS protocol used (see file [RD6006_prototocol_en.pdf](RD6006_prototocol_en.pdf "RD6006 prototocol reverse engineering")), the solution to use ***node-red*** is the fastest and most flexible, because it requires very little programming. Furthermore
***node-red*** is compatible with many environments: windows, Linux, Android, Raspberry...

This first simple flow is useful for tests: you can quickly write and read RD6006 registers.
![ModBus test RD6006](images/2020-02-16.202430.shot.png)
**Installation**

I used the USB-serial connection, on WIN10 with the `CH341SER` driver. Once connected, in *devices* it is present with the name 'USB-SERIAL.CH340 (COM10)'.

- Add to the ***node-red** palette* the package *node-red-contrib-modbus* (see [https://github.com/BiancoRoyal/node-red-contrib-modbus](https://github.com/BiancoRoyal/node-red-contrib-modbus "node-red-contrib-modbus"))
- Copy the contents of the file [ModBus_test_RD6006.json](ModBus_test_RD6006.json "ModBus test RD6006") to the clipboard and import it into a new flow in ***node-red***.
- More use informations are in the comment nodes.

## NiMH battery charger
This *flow* uses the RD6006 to get a simple but complete NiMH battery charger-logger.

![](images/2020-02-20.134707.shot.png)



 Config values are set for 2 options:

- **SLOW charge**, constant tension (1.47-1.50), low current (C/40.. C/10), time very long, also continous charging. 
- **FAST charge**, constant current (<= 1.2 C), high tension (5 V), until one hour charge.

 ![](images\2020-02-20.134433.shot.png)

End charge conditions:

- by user (STOP button)
- by RD6006 (OVP, OCP, OTP, Iout<10 mA)
- added test: Vbatt > Vmax
- added test: Tbatt > Tmax
- added test: chargeAh > Ahmax

The charge (Ah) is reset to 0 every START (in the RD6006 Ah is reset only at startup).
The battery V value can be get also in *smart* mode: because the RD6006 don't allows Iout=0, the internal resistence is calculated from a differential measure at startup, so Vsmart = Vbatt - Iout*Ri.

Every charge cicle generates a new mySQL table, with records stored every 5 sec, and a definition record on the index table 'batteries'. 

**Installation**

- Add to the ***node-red** palette* the package *node-red-node-mysql*
  (see [https://flows.nodered.org/node/node-red-node-mysql](https://flows.nodered.org/node/node-red-node-mysql "node-red-node-mysql"))
- Copy the contents of the file [NiMH_charger_RD6006.json](NiMH_charger_RD6006.json "NiMH_charger_RD6006") to the clipboard and import it into a new flow in ***node-red***.
- The values in *SLOW config* and *FAST config* nodes can be modified to suit your needs.
