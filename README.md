# ME910-cheat-sheet
Telit ME910 Cheat Sheet

## Ubuntu 18.04

If you want to send raw AT commands over uart make sure ModemManager isn't also talking to the module:
```
systemctl stop ModemManager
```
Monitor dmesg:
```
dmesg -w
```
Hold the power button and observe the module power up:
```
[602715.818079] option 1-3:1.0: GSM modem (1-port) converter detected
[602715.818686] usb 1-3: GSM modem (1-port) converter now attached to ttyUSB0
[602715.819420] option 1-3:1.1: GSM modem (1-port) converter detected
[602715.819979] usb 1-3: GSM modem (1-port) converter now attached to ttyUSB1
[602715.820757] option 1-3:1.2: GSM modem (1-port) converter detected
[602715.821903] usb 1-3: GSM modem (1-port) converter now attached to ttyUSB2
```
Connect over minicom
```
sudo minicom -D /dev/ttyUSB1
```
minicom configuration:

* CTRA-A, Z, P: 115200 8N1
* CTRA-A, Z, O: Hardware Flow Control: No
* CTRA-A, Z, O: Software Flow Control: No

Check Comms:
```
AT                                                                                                                          
OK
```


# Commands

| Command     |  Info                                     |
|-------------|-------------------------------------------|
| AT+CGDCONT? |  Check APN details                        |
| AT+CGAUTH?  | Check password                            |
| AT+CREG=2   | Unsolicited Registration message          |
| AT+CEREG=2  | Unsolicited Registration message (NB-IoT) |
| AT+CGREG=2  | Unsolicited Registration message (2G)     |
| AT+CREG?    | Check Current Registration                |
| AT+CEREG?   | Check Current Registration (NB-IoT)       |
| AT+CGREG?   | Check Current Registration (2G)           |


# Debugging

From here: [link](https://community.hologram.io/t/network-registration-denied-creg-0-3/2762/4)

```
ATI <<prints module information
AT+CPIN? <<should return “READY”
AT+CFUN? <<checks module state, should be 1 for full functionality
AT+COPS=? <<will search for nearby networks, may take 1-3 minutes
AT+CGDCONT? <<should print the configured PDP contexts, hologram should be in this list, may show an IP if given one by the carrier
AT+CGACT? <<checks if PDP context is active
```



