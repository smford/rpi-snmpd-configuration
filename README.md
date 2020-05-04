# rpi-snmpd-configuration
How to setup your raspberry pi for SNMP with the proper information, including temperature, networking and OS details.

Most other instructions lack details, do not include temperature, networking or OS details.

Follow these steps and it will work for you.

## Tested Hardware
- Raspberry Pi Zero Wireless

## Software
- Raspbian 10 Buster Lite (OS)
- snmp (client)
- snmpd (server)
- bc (cli arithmetic tool)

## Commands
1. SSH in to your RPI, in this example the IP is 192.168.10.129
    ```
    ssh pi@192.168.10.129
    ```
1. Install needed software:
    ```
    sudo su -
    apt-get update
    apt-get install bc snmp snmpd snmp-mibs-downloader -y
    ```
1. Configure SNMP client:
    ```
    sed -i "s/mibs :/#mibs :/1" /etc/snmp/snmp.conf
    ```
1. Install scripts needed for SNMP:
    ```
    wget -O /usr/local/bin/distro https://gitlab.com/observium/distroscript/raw/master/distro
    chmod +x /usr/local/bin/distro
    cat <<EOF >> /usr/local/bin/cputemp
    #!/bin/bash
    bc -l <<< "scale=2; $(cat /sys/class/thermal/thermal_zone0/temp)/1000"
    EOF
    chmod +x /usr/local/bin/cputemp
    ```
1. Configure snmpd:
    ```
    vim /etc/snmp/snmpd.conf
    ```
1. Comment out the below line which restricts snmp to local system only:
    ```
    agentAddress  udp:127.0.0.1:161
    ```
1. Uncomment this line to allow other machines to connect to the snmp daemon on this server:
    ```
    agentAddress udp:161,udp6:[::1]:161
    ```
1. Change the following lines to whatever you need:
    ```
    sysLocation    Sitting on the Dock of the Bay
    sysContact     Me <me@example.org>
    ```
1. Comment out the below line because it is too restrictive, we will add a line granting access later:
    ```
    view   systemonly  included   .1.3.6.1.2.1.1
    ```
1. Comment out the below two lines because we don't need this junk:
    ```
    extend    test1   /bin/echo  Hello, world!
    extend-sh test2   echo Hello, world! ; echo Hi there ; exit 35
    ```
1. Add the following lines to the bottom of the file:
    ```
    # disable useless connection logs in /var/log/daemon.log
    dontLogTCPWrappersConnects yes
    
    # add temp information
    # cpuTemp0 = .1.3.6.1.4.1.8072.1.3.2.3.1.1.8.99.112.117.84.101.109.112.48
    # cpuTemp1 = .1.3.6.1.4.1.8072.1.3.2.3.1.1.8.99.112.117.84.101.109.112.49
    extend cpuTemp0 /usr/local/bin/cputemp
    
    # add distribution information
    extend .1.3.6.1.4.1.2021.7890.1 distro /usr/local/bin/distro
    
    # grant access to the right information, by default it is too restrictive
    view systemonly included .1.3.6.1.2
    
    # grant access to distro information
    view systemonly included .1.3.6.1.4.1.2021.7890.1
    
    # grant access to the temp information
    view systemonly included .1.3.6.1.4.1.8072.1.3.2
    ```
1. Save and exit vim
1. Restart the snmpd daemon with the new configuration:
    ```
    service snmpd restart
    ```
1. Check SNMP (change the IP to your RPI):
    ```
    # display distro information
    snmpwalk -v2c -c public 192.168.10.129 .1.3.6.1.4.1.2021.7890.1

    # display system temp
    snmpwalk -v2c -c public 192.168.10.129 .1.3.6.1.4.1.8072.1.3.2.3.1.1.8.99.112.117.84.101.109.112.48
    ```

## References
- [snmp temp oid example](https://www.reddit.com/r/PFSENSE/comments/cnjddc/howto_retrieve_cpu_temp_and_other_data_over_snmp/)
- [pass example with temp](http://www.satsignal.eu/raspberry-pi/monitoring.html)
