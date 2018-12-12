## UFW - Using the Uncomplicated Firewall to mitigate potential attacks
by Lecory Wooding
 ###### Using Ubuntu's CLI (command line interface), and the UFW firewall, I will configure and show how to easily block certain traffic if there is suspicion of penetration testing from an outside entity.
 ---

#### First we install updates on the Ubuntu machine by entering:
``` sh
$ sudo apt-get update
```
- ##### Followed by:
```sh
$ sudo apt-get upgrade
```
---
#### Now we must install a few things: 

```sh
$ sudo apt-get install wireshark
```
- ##### Wireshark is a network protocol analyzer that allows a user to monitor network traffic
```sh
$ sudo apt install openssh-server
```
- ##### Openssh is a suite of secure networking utilities based on the Secure Shell (SSH) protocol
---
## UFW comes prebuilt in Ubuntu! However, it can be installed using the command below:
```sh
$ sudo apt-get install ufw
```
---
#### After installtion enter:
```sh
$  sudo ufw status
```
- ##### If inactive enter:
```sh
$ sudo ufw enable
```
- ##### Then enter:
```sh
$ sudo ufw status
```
#### We need to add rules to allow certain traffic:
```sh
$ sudo ufw allow ssh
```
```sh
$ sudo ufw allow http
```
- ##### If for some reason you are asked to use telnet (not advised)
```sh
$ sudo ufw allow telnet
```
- ##### Once we have done this we can check the firewall rules using:
```sh
sudo ufw status
```
- ##### Since we are here lets run a simple server on port 80 by using the command below:
```sh
$ sudo python -m SimpleHTTPServer 80
```
- ##### Open Wireshark by typing 'Wireshark' into a terminal window OR by GUI
- ##### Navigate to 'Capture' and start capturing packets on the ethernet interface

--- 
## On the Kali machine
- ##### Log in using the provided credentials of 'root' as username and 'toor' as password
- ##### Now lets begin scanning our network for hosts
- ##### Open a terminal window and obtain your IP address by using:
```sh
$ ifconfig
```
- ##### In the same terminal window enter:
```sh
$ nmap -sN 10.0.2.0/24
```
- ##### Nmap is a tool used for network mapping, the -sN flag tells Nmap to scan for hosts that are up on our current network of 10.0.2.0/24
- ##### With these in mind we now turn to 'Zenmap'
- ##### Type in the terminal:
```sh
$ Zenmap
```
- ##### This will open zenmap, a GUI version of nmap
- ##### In 'Zenmap' we will run the scan again, but we will select ping scan from the right drop down box  and specifiy the address in the box that says target:
```sh
$ 10.0.2.0/24
```
- ##### The results in the 'NmapOutput' window should contain the Ubuntu machines IP address, along with ours. From here a user could run more scans against your machine, looking for a vulnerablity
---
### On the Ubuntu machine lets look at our Wireshark capture
- ##### Press the red square to stop the capture in Wireshark and examine the packets
- ##### It seems most packets are coming from one source, and at an alarming rate, this is a red flag that someone may be doing reconnaissance
- ##### Using information gathered from Wireshark we can block all traffic from that IP address with the command below:
```sh
$ sudo ufw deny from 10.0.2.12 to any
```
- ##### Once you hit enter there is no need for restarting the firewall as the rules take effect immediately
--- 
### On the Kali machine 
- ##### Attempt to ping the Ubuntu machine:
```sh
$ ping 10.0.2.13
```
- ##### The ping should be unsuccesful or timeout
---
### On the Ubuntu machine
- ##### We have figured out how to block the IP address doing recon on our network but attackers can easily disguise their IP address and continue to do recon. Another method that can protect you, while you attempt to find and patch vulnerabilities, is to disable ICMP requests 
- ##### To do this we must edit the UFW's 'before.rules' configuration file, lets navigate there:
```sh
$ cd /etc/ufw/
```
- ##### List the files:
```sh
$ ls
```
---
#### Before making any edits to a configuration file it is recommended you make a copy incase we need to revert to default rules!
---
- ##### To make a copy enter:
```sh
$ sudo cp before.rules before.rules.copy
```
- ##### List the files to make sure your copy was made:
```sh
$ ls
```
- ##### Lets edit:
```sh
$ sudo nano before.rules
```

- ##### Scroll down till you see a section that reads as follows:
```sh
$ #ok icmp codes
-A ufw-before-input -p icmp --icmp-type destination-unreachable -j ACCEPT
-A ufw-before-input -p icmp --icmp-type source-quench -j ACCEPT
-A ufw-before-input -p icmp --icmp-type time-exceeded -j ACCEPT
-A ufw-before-input -p icmp --icmp-type parameter-problem -j ACCEPT
-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
```
- ##### We want to change the 'echo-request' line to DROP:
```sh
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
```
- ##### Now we press 'Ctrl+X' to exit the Nano editor; you will be asked if you want to save this file. Type 'Y' then hit enter
- ##### This rule change will drop all incoming ICMP requests at the firewall; elminating the main process by which machines are discovered by attackers
- ##### Before the rule can take effect we must restart the firewall using:
```sh
$ sudo ufw disable
```
& 
```sh
$ sudo ufw enable
```
- ##### Since we have already blocked the Kali machine address, our PING should not work
---
## Ping plays a pivotal part in network connectivity tests, disabling it long-term is something to be discussed when planning for network expansion!
---
### Navigate back to the Ubuntu machine; open a terminal and enter:
```sh
sudo ufw delete deny from 10.0.2.12 to any
```
- ##### This command removes the deny all traffic rule we created to deter our attacker earlier
---
#### On the Kali machine, attempt to ping the Ubuntu machine
- ##### This should fail
---
#### In an Ubuntu terminal window type:
```sh
$ sudo ufw disable
```
- ##### This disables the firewall
----
#### On the Kali machine attempt to ping the Ubuntu machine
- ##### The ping should be a success
---
#### On the Ubuntu machine type:
```sh
$ sudo ufw enable
```
--- 
#### Attempt to ping the Ubuntu machine
- ##### It will be unsuccessful
---
## Summary
- ##### UFW is a powerful tool that can help keep your communications and networked machines a little more secure. Human usage and malicious applications open up networks to a plethora of unknown traffic. The ease of the command line when it comes to managing this firewall makes it one that not only experts can use, but beginners as well. 
