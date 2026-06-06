# snmp remote without password
> sudo visudo

`Debian-snmp   ALL=(ALL:ALL) NOPASSWD: ALL`

----------------------------------------------------

## Restricting SNMP Access to Custom OIDs

Add this line in the file : `sudo nano /etc/snmp/snmpd.conf`
```py
# Listen for SNMP requests on all IPv4 interfaces (UDP 161) and localhost IPv6 (::1)
agentAddress udp:161,udp6:[::1]:161

# Define a custom view limited to your OID subtree
view    customonly  included   .1.3.6.1.9

# Use this view for the community string 'public' (or whatever you're using)
rocommunity public default -V customonly
```

## SNMP OIDs and script mapping

Add this line at the end of the file : `sudo nano /etc/snmp/snmpd.conf`
```py
# include site specific config
includeFile /home/bps/.local/snmp/snmpd_pass_commands.conf

# include a all *.conf files in a directory
includeDir /home/bps/.local/snmp
```
## Configuring SNMP Pass-Through Commands

Let’s write the script at : `sudo nano /home/bps/.local/snmp/snmpd_pass_commands.conf`
~~~py
pass .1.3.6.1.9.1.1.1 /bin/sudo /bin/sh /usr/local/bin/bms1/soc_totalVoltage
pass .1.3.6.1.9.1.1.2 /bin/sudo /bin/sh /usr/local/bin/bms1/soc_current
pass .1.3.6.1.9.1.1.3 /bin/sudo /bin/sh /usr/local/bin/bms1/soc_percent
~~~

## Creating SNMP Wrapper Script for Voltage OID

Next, place the following code into : `sudo nano /usr/local/bin/bms1/soc_totalVoltage`
~~~bash
#!/bin/bash
if [ "$1" = "-g" ]
then
  # Specify the OID and type for the SNMP response
  echo .1.3.6.1.9.1.1.1
  echo STRING  # Ensure the value is treated as an STRING OR gauge

  # Call the Python script to generate the value
  /usr/bin/python3 /home/bps/.local/bin/bms1/soc_totalVoltage.py
fi
exit 0
~~~




----------------------------------------------------
# SNMP for Custom MIB
~~~py
pass .1.3.6.1.9.1.1.1 /bin/sh /usr/local/bin/bms1/soc_totalVoltage
~~~
> `sudo systemctl restart snmpd.service`

> `/usr/local/bin/bms1/soc_totalVoltage`
~~~bash
#!/bin/bash
if [ "$1" = "-g" ]
then
  # Specify the OID and type for the SNMP response
  echo .1.3.6.1.9.1.1.1
  echo STRING  # Ensure the value is treated as an STRING OR gauge

  # Call the Python script to generate the value
  /usr/bin/python3 /home/bps/.local/bin/bms1/soc_totalVoltage.py
fi
exit 0
~~~
> `sudo nano /home/bps/.local/bin/bms1/soc_totalVoltage.py`
~~~py
import serial
import struct
import time

def calculate_crc(message_bytes):
    """Calculate the checksum of a message (sum of bytes & 0xFF)."""
    return bytes([sum(message_bytes) & 0xFF])

def format_message(address, command, extra=""):
    """
    Format the BMS request message.
    :param address: 4 for USB/RS485, 8 for Bluetooth
    :param command: Daly command (e.g. "90")
    """
    message = "a5%01x1%s08%s" % (address, command, extra)
    #message = "a5%01x2%s08%s" % (address, command, extra)
    message = message.ljust(24, "0")  # pad to 12 bytes (24 hex characters)
    message_bytes = bytearray.fromhex(message)
    message_bytes += calculate_crc(message_bytes)
    return message_bytes

def read_total_voltage(port='/dev/ttyUSB0', address=4):
    """
    Read only the total voltage from Daly BMS.
    """
    try:
        ser = serial.Serial(
            port=port,
            baudrate=9600,
            bytesize=serial.EIGHTBITS,
            parity=serial.PARITY_NONE,
            stopbits=serial.STOPBITS_ONE,
            timeout=0.5
        )
    except Exception as e:
        print(f"Failed to open serial port: {e}")
        return

    command = format_message(address, "90")

    try:
        ser.reset_input_buffer()
        ser.reset_output_buffer()

        ser.write(command)
        time.sleep(0.1)  # allow BMS to respond

        response = ser.read(13)
        if len(response) != 13:
            print("Incomplete response from BMS.")
            return

        # Verify CRC
        if calculate_crc(response[:-1]) != response[-1:]:
            print("CRC check failed!")
            return

        # Parse total voltage
        total_voltage_raw = response[4:6]
        total_voltage = struct.unpack(">h", total_voltage_raw)[0] / 10.0

        print(f"{total_voltage:.1f} V")

    finally:
        ser.close()

# Run it
if __name__ == "__main__":
    read_total_voltage("/dev/ttyUSB0")  # Update port if needed
~~~

~~~python
print("50 V")
~~~
----------------------------------------
# SNMO Pass Persist
`sudo nano /usr/local/bin/custom_snmp.py`
~~~py
#!/usr/bin/env python3

import sys
import time

def read_data():
    # Replace these with your actual sensor readings
    voltage = "12.5"
    current = "2.1"
    temperature = "37.2"
    return voltage, current, temperature

def main():
    oid_base = ".1.3.6.1.9.1.9999"
    voltage_oid = oid_base + ".1.0"
    current_oid = oid_base + ".2.0"
    temperature_oid = oid_base + ".3.0"

    while True:
        line = sys.stdin.readline().strip()
        if line == "PING":
            print("PONG")
            sys.stdout.flush()
        elif line == "get":
            req_oid = sys.stdin.readline().strip()
            voltage, current, temperature = read_data()

            if req_oid == voltage_oid:
                print(f"{voltage_oid}\nstring\n{voltage}")
            elif req_oid == current_oid:
                print(f"{current_oid}\nstring\n{current}")
            elif req_oid == temperature_oid:
                print(f"{temperature_oid}\nstring\n{temperature}")
            else:
                print("NONE")
            sys.stdout.flush()
        elif line == "getnext":
            req_oid = sys.stdin.readline().strip()
            # Implement getnext behavior here
            if req_oid < voltage_oid:
                print(f"{voltage_oid}\nstring\n{read_data()[0]}")
            elif req_oid < current_oid:
                print(f"{current_oid}\nstring\n{read_data()[1]}")
            elif req_oid < temperature_oid:
                print(f"{temperature_oid}\nstring\n{read_data()[2]}")
            else:
                print("NONE")
            sys.stdout.flush()
        elif line == "":
            time.sleep(0.1)

if __name__ == "__main__":
    main()
~~~
Make it executable: `sudo chmod +x /usr/local/bin/custom_snmp.py`

`sudo nano /etc/snmp/snmpd.conf`
~~~py
pass_persist .1.3.6.1.9.1.9999 /usr/local/bin/custom_snmp.py
~~~

Restart the SNMP daemon: `sudo systemctl restart snmpd`

Test with `snmpwalk`
`snmpwalk -v2c -c public localhost .1.3.6.1.4.1.9999`

