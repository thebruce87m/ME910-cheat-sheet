# ME910-cheat-sheet
Telit ME910 Cheat Sheet



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
