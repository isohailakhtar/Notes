`sudo nano /etc/snmp/snmpd.conf`

~~~py
###############################################################################
# Agent listening on IPv4 + IPv6
agentAddress udp:161,udp6:[::1]:161

###############################################################################
# Limit SNMP view to ONLY your custom OID tree
view customonly included .1.3.6.1.4.1.9999

###############################################################################
# Community string access — limited to custom view
# rocommunity public default -V customonly

###############################################################################
# Optional: SNMPv3 secure access (with same OID restriction)
createUser admin SHA bpsadmin AES bpsadmin
rouser admin authPriv -V customonly

###############################################################################
# Trap destination (optional)
trap2sink 192.168.0.148 public 162

###############################################################################
# Custom SNMP extension handler using pass_persist
pass_persist .1.3.6.1.4.1.9999 /usr/local/bin/custom_snmp_all.py
~~~

---
` snmpget -v3 -u admin -a SHA -A bpsadmin -x AES -X bpsadmin -l authPriv localhost .1.3.6.1.4.1.9999.1.1.1`

If errors comes:

`sudo apt update && sudo apt install snmp-mibs-downloader snmp && unset MIBS && snmpwalk -m ALL -v3 -l authPriv -u admin -a SHA -A bpsadmin -x AES -X bpsadmin localhost system`


-----------------------------------------------------
`sudo nano /usr/local/bin/custom_snmp_all.py`
~~~py
#!/usr/bin/env python3

import sys
import time

def read_data():
    voltage = "12.5"
    current = "2.1"
    temperature = "37.2"
    highest_voltage = "13.8"
    lowest_voltage = "11.4"
    highest_temperature = "45.2"
    lowest_temperature = "22.1"
    mode = "charging"
    charging_mosfet = "1"
    discharging_mosfet = "0"
    capacity_ah = "150.0"
    cells = "15"
    temperature_sensors = "4"
    charger_running = "1"
    load_running = "1"
    cycles = "218"
    error_text = "No errors"
    cell_voltages = ["3.6"] * 15
    sensor_temps = ["30.1", "31.2", "29.8", "30.7"]

    return (
        voltage, current, temperature, highest_voltage, lowest_voltage,
        highest_temperature, lowest_temperature, mode, charging_mosfet, discharging_mosfet,
        capacity_ah, cells, temperature_sensors, charger_running, load_running, cycles, error_text,
        *cell_voltages, *sensor_temps
    )

def main():
    # Base OID tree
    oid_base = ".1.3.6.1.4.1.9999"

    # Map OIDs to data-producing functions
    oid_map = {
        f"{oid_base}.1.1.1": lambda: read_data()[0],   # voltage
        f"{oid_base}.1.1.2": lambda: read_data()[1],   # current
        f"{oid_base}.1.1.3": lambda: read_data()[2],   # temperature
        f"{oid_base}.1.2.1": lambda: read_data()[3],   # highest_voltage
        f"{oid_base}.1.2.3": lambda: read_data()[4],   # lowest_voltage
        f"{oid_base}.1.3.1": lambda: read_data()[5],   # highest_temperature
        f"{oid_base}.1.3.3": lambda: read_data()[6],   # lowest_temperature
        f"{oid_base}.1.4.1": lambda: read_data()[7],   # mode
        f"{oid_base}.1.4.2": lambda: read_data()[8],   # charging_mosfet
        f"{oid_base}.1.4.3": lambda: read_data()[9],   # discharging_mosfet
        f"{oid_base}.1.4.4": lambda: read_data()[10],  # capacity_ah
        f"{oid_base}.1.5.1": lambda: read_data()[11],  # cells
        f"{oid_base}.1.5.2": lambda: read_data()[12],  # temperature_sensors
        f"{oid_base}.1.5.3": lambda: read_data()[13],  # charger_running
        f"{oid_base}.1.5.4": lambda: read_data()[14],  # load_running
        f"{oid_base}.1.5.5": lambda: read_data()[15],  # cycles
        f"{oid_base}.1.9.1": lambda: read_data()[16],  # error_text
        f"{oid_base}.1.6.1": lambda: read_data()[17],  # cell_1
        f"{oid_base}.1.6.2": lambda: read_data()[18],  # cell_2
        f"{oid_base}.1.6.3": lambda: read_data()[19],  # cell_3
        f"{oid_base}.1.6.4": lambda: read_data()[20],  # cell_4
        f"{oid_base}.1.6.5": lambda: read_data()[21],  # cell_5
        f"{oid_base}.1.6.6": lambda: read_data()[22],  # cell_6
        f"{oid_base}.1.6.7": lambda: read_data()[23],  # cell_7
        f"{oid_base}.1.6.8": lambda: read_data()[24],  # cell_8
        f"{oid_base}.1.6.9": lambda: read_data()[25],  # cell_9
        f"{oid_base}.1.6.10": lambda: read_data()[26], # cell_10
        f"{oid_base}.1.6.11": lambda: read_data()[27], # cell_11
        f"{oid_base}.1.6.12": lambda: read_data()[28], # cell_12
        f"{oid_base}.1.6.13": lambda: read_data()[29], # cell_13
        f"{oid_base}.1.6.14": lambda: read_data()[30], # cell_14
        f"{oid_base}.1.6.15": lambda: read_data()[31], # cell_15
        f"{oid_base}.1.7.1": lambda: read_data()[32],  # sensor_1
        f"{oid_base}.1.7.2": lambda: read_data()[33],  # sensor_2
        f"{oid_base}.1.7.3": lambda: read_data()[34],  # sensor_3
        f"{oid_base}.1.7.4": lambda: read_data()[35],  # sensor_4
    }

    # OIDs in sorted order for getnext/getbulk/walk
    sorted_oids = sorted(oid_map.keys())

    while True:
        line = sys.stdin.readline().strip()

        if line == "PING":
            print("PONG")
            sys.stdout.flush()

        elif line == "get":
            req_oid = sys.stdin.readline().strip()
            if req_oid in oid_map:
                print(f"{req_oid}\nstring\n{oid_map[req_oid]()}")
            else:
                print("NONE")
            sys.stdout.flush()

        elif line == "getnext":
            req_oid = sys.stdin.readline().strip()
            next_oid = None
            for oid in sorted_oids:
                if oid > req_oid:
                    next_oid = oid
                    break
            if next_oid:
                print(f"{next_oid}\nstring\n{oid_map[next_oid]()}")
            else:
                print("NONE")
            sys.stdout.flush()

        elif line == "getbulk":
            non_repeaters = int(sys.stdin.readline().strip())
            max_repetitions = int(sys.stdin.readline().strip())

            oids = []
            for _ in range(non_repeaters + max_repetitions):
                oids.append(sys.stdin.readline().strip())

            responses = []

            for oid in oids:
                next_oid = None
                for candidate in sorted_oids:
                    if candidate > oid:
                        next_oid = candidate
                        break
                if next_oid:
                    responses.append(f"{next_oid}\nstring\n{oid_map[next_oid]()}")
                else:
                    responses.append("NONE")

            print("\n".join(responses))
            sys.stdout.flush()

        elif line == "":
            time.sleep(0.1)

if __name__ == "__main__":
    main()
~~~
