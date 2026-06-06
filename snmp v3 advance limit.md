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
