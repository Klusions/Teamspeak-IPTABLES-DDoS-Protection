# Teamspeak-IPTABLES-DDoS-Protection #
# Protect your Teamspeak Server from DDoS Protections #

#DO NOT FORGET TO READ THE DETAILED DESCRIPTIONS IN IPTABLES SCRIPT!#
>1- All UDP ports will be under TeamSpeak3 DDoS Protection Zone (You can create a lot of TeamSpeak3 servers by using different UDP ports, all of team will be safe!)
>2- All TCP ports will be banned except 22,10011,30033,41144 (you can read the descriptions and add more tcp ports which you will use. For instance, 80,443,3306 etc..)
>3- Except TCP and UDP protocols, all the protocols will be banned (Because you do not need the other protocols like ICMP, IGMP etc.. for your TeamSpeak3 servers, the best way is to block all of the protocols which you will not use)


---

```
# NatureNMoon - TS3 Mitigation on IPTABLES
# DESCRIPTIONS BELOW;
# If you do not have iptables, install it Centos: "yum install iptables" and Ubuntu/Debian: "apt-get install iptables"
# 51.68.181.92 is weblist.teamspeak.com - this ip address has to be excepted
# You should create ipset (if you do not have ipset please install it (Centos: yum install ipset || Ubuntu/Debian apt-get install ipset
# you can create ipset by using this code "ipset create ts3_allowed hash:ip hashsize 2097152 maxelem 40000000 timeout 259200"
# Your ssh must be 22 TCP
# 10011 : Query port (You can change this port when you change it in this iptables script below)
# 30033 : File Transfer port (You can change this port when you change it in this iptables script below)
# If you ask why I choose *raw chain, raw chain is the most important chain in IPTABLES, you can think this chain as a "root" in linux and this chain can block 1.000.000 Packet Per Second (depends on the power of your servers(CPU,RAM,NIC, NETWORK BANDWIDTH))

*raw
:PREROUTING ACCEPT [0:0] // default raw prerouting rules - action (accept)
:R4P3 - [0:0] // Default traffic chain
:TS3 - [0:0] // TS3 PROTECTION CHAIN
:PROTOCOL_MANAGER - [0:0] // This chain will block all the traffics except UDP and TCP
:OUTPUT ACCEPT [559:74102] // No need to change or add something for this chain.


-A PREROUTING -j R4P3 // send all packets to main R4P3 chain to block the traffic well"
-A R4P3 ! -s 51.68.181.92/32 -d YourServerExternalIPAddress -i YourInterfaceHere -m set ! --match-set ts3_allowed src -j TS3 // Please change "YourInterfaceHere" as eth0 or whatever it is in your server and change this "YourServerExternalIPAddress" as your server external ip address

## TS3 RULES

-A TS3 -p tcp -m multiport --dports 22,10011,30033,41144 -j RETURN  // This rule will let 22,10011,30033 and 41144 tcp ports enter into your network.
-A TS3 -p udp --sport 53 -m length --length 750:65535 -j DROP // Mitigation for DNS Amplification attacks
-A TS3 -p udp ! --sport 53 -m length --length 62 -m hashlimit --hashlimit-upto 5/sec --hashlimit-burst 10 --hashlimit-mode dstip --hashlimit-name ts3_ratelimit --hashlimit-htable-max 2000000 -m string --string "TS3INIT" --algo kmp -j SET --add-set ts3_allowed src // Accept 5 users each second if the users' packet contains "TS3INIT" payload and its length is 62 byte then add them to ts3_allowed ipset
-A TS3 -m set ! --match-set ts3_allowed src -j DROP // Block all the traffics if their source ip is not in ts3_allowed(normal users) and destionation ports tcp are not 22,10011,30033 and 41144
-A TS3 -j PROTOCOL_MANAGER // send all the packets to the PROTOCOL MANAGER chain

## PROTOCOL MANAGER RULES

-A PROTOCOL_MANAGER -p tcp -j RETURN // Allow UDP Traffic
-A PROTOCOL_MANAGER -p udp -j RETURN // Allow TCP Traffic
-A PROTOCOL_MANAGER -j DROP // Except UDP and TCP protocols, block all the protocols

COMMIT
# The IPTABLES script has been created to keep the TeamSpeak3 Servers alive by NatuerNMoon in R4P3
```
