# rpi-snmpd-configuration
How to setup your raspberry pi for ship SNMP data

## Hardware
- Raspberry Pi Zero Wireless

## Software
- Raspbian 10 Buster Lite (OS)
- snmp (client)
- snmpd (server)

## Commands
1. Run these commands:
    ```
    ssh pi@192.168.10.129
    sudo su -
    apt-get update
    apt-get install snmp snmpd snmp-mibs-downloader -y
    sed -i "s/mibs :/#mibs :/1" /etc/snmp/snmp.conf
    wget -O /usr/local/bin/distro https://gitlab.com/observium/distroscript/raw/master/distro
    chmod +x /usr/local/bin/distro
    ```
1. Configure snmpd:
    ```
    vim /etc/snmp/snmpd.conf
    ```
1. Comment out this line:
    ```
    agentAddress  udp:127.0.0.1:161
    ```
1. Uncomment this line:
    ```
    agentAddress udp:161,udp6:[::1]:161
    ```
1. Change the following lines to your details:
    ```
    sysLocation    Sitting on the Dock of the Bay
    sysContact     Me <me@example.org>
    ```
1. Comment out:
    ```
    view   systemonly  included   .1.3.6.1.2.1.1
    ```
1. Comment out:
    ```
    extend    test1   /bin/echo  Hello, world!
    extend-sh test2   echo Hello, world! ; echo Hi there ; exit 35
    ```
1. Add to the bottom of the file:
    ```
    dontLogTCPWrappersConnects yes
    extend cputemp /usr/local/bin/cputemp
    extend .1.3.6.1.4.1.2021.7890.1 distro /usr/local/bin/distro
    view systemonly included .1.3.6.1.2
    view systemonly included .1.3.6.1.4.1.2021.7890.1
    view systemonly included .1.3.6.1.4.1.8072.1.3.2
    ```
1. Save and exit vim
1. Start up the snmpd daemon:
    ```
    service snmpd start
    ```
1. Check SNMP:
