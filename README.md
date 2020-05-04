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

| Command     | Info                                      | Output                                  |
|-------------|-------------------------------------------|-----------------------------------------|
| ATZ         | Soft Reset                                |                                         |
| ATI4        | Module Name                               | ME910C1-E2                              |
| AT#QSS?     | Query SIM Status                          | #QSS: 0,1                               |
| AT+CFUN?    | Query phone functionality                 |                                         |
| AT+CGDCONT? |  Check APN details                        |                                         |
| AT+CGAUTH?  | Check password                            |                                         |
| AT+CREG=2   | Unsolicited Registration message          |                                         |
| AT+CEREG=2  | Unsolicited Registration message (NB-IoT) |                                         |
| AT+CGREG=2  | Unsolicited Registration message (2G)     |                                         |
| AT+CREG?    | Check Current Registration                |                                         |
| AT+CEREG?   | Check Current Registration (NB-IoT)       |                                         |
| AT+CGREG?   | Check Current Registration (2G)           |                                         |
| AT#PSNT?    | Packet Service Network Type               | 0=GPRS, 4=LTE, 5=Unknown/Not Registered |

# Debugging

From here: [link](https://community.hologram.io/t/network-registration-denied-creg-0-3/2762/4)

```
ATI         // prints module information
AT+CPIN?    // should return “READY”
AT+CFUN?    // checks module state, should be 1 for full functionality, 4 = "flight mode"
AT+COPS=?   // will search for nearby networks, may take 1-3 minutes
AT+CGDCONT? // should print the configured PDP contexts, hologram should be in this list, may show an IP if given one by the carrier
AT+CGACT?   // checks if PDP context is active
```

```
SWPKGV        // Request Software Package Version
AT+CGSN       // Request Product Serial Number Identification 
AT#CCIOTOPT?  // Query CIoT Optimization Configuration
AT#CPSMS?     // Query Power Saving Mode Setting
```

# Our Example

```
ATZ
AT+CFUN=1                    // Full functionality 
AT+COPS=2                    // Disable network registration 
AT+CREG=2
AT+CGREG=2
AT+CEREG=2
AT+CREG?                     // Make sure it's not registered already and not searching
AT+CGREG?                    // Make sure it's not registered already and not searching (2G)
AT+CEREG?                    // Make sure it's not registered already and not searching (NB-IoT)
AT+COPS=1,2,"23415",9        // NB-IoT Registration
//AT+COPS=1,2,"23415",0      // 2G Registration
```

Example Output (2G)
```
ATZ
OK
ATZ
OK
AT+COPS=2
OK
AT+CFUN=1
OK                                                                                                        
AT+CREG=2                                                                                                 
OK                                                                                                        
AT+CGREG=2                                                                                                
OK                                                                                                        
AT+CEREG=2
OK
AT+CREG?
+CREG: 2,0

OK
AT+COPS=1,2,"23415",0
OK

+CREG: 1,"03CE","C4EE",0

+CGREG: 2

+CGREG: 1,"03CE","C4EE",0,

// Manual Queries

AT+CREG?
+CREG: 2,1,"03CE","C4EE",0

OK

AT+CGREG?
+CGREG: 2,1,"03CE","C4EE",0,"01"

OK

AT+CEREG?
+CEREG: 2,0

OK
```


# Telit Examples

From [ME910C1 Quick Start Guide](https://y1cj3stn5fbwhv73k0ipk1eg-wpengine.netdna-ssl.com/wp-content/uploads/2018/11/Telit_ME910C1_QuickStart_Guide_r1.pdf)

## Full TCP script
```
AT+CFUN=1                                      // Full functionality 
AT+COPS=2                                      // Disable network registration 
AT+CGDCONT=1,"IP",”<APN>”                      // Set <cid>=1 with APN 
AT+COPS=0                                      // Enable network registration with automatic selection 
AT#SCFG=1,1,300,90,600,50                      // Socket Configuration (see AT command manual for the details of the command) 
AT#SGACT=1,1                                   // Internal modem process (expect to receive: ERROR) 
AT+CEREG?                                      // Check Network registration , 2 or 5 is the positive answer // for that. 
AT#SGACT?                                      // Confirm Context activated
AT+CGPADDR=1                                   // Read IP address 
AT#SD=1,0,<Dest. Port>,<"IP address">,0,1234,1 // Open a TCP socket to remote server 
AT#SSEND=1                                     // After that command you will recieve a prompt > and you          
                                               // can send  the data just typing it and end them with Crtl^Z                                          
                                               // Transmit the data and end the string with Crtl^Z to send it
SRING: 1                                       // message sent 
AT#SH=1                                        // Close the socket
```

