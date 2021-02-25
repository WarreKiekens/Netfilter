# Netfilter
Challenge:
As a system engineer your expertise is asked to create a firewall ruleset for a hosting server.
The server is provided with the following services: Apache, ProFTPd and bind9. Please, do not allow zonetransfers (think twice). Also protect the server against ping flooding. The server is not allowed to make outgoing connections, except for the installation of security updates.
Present your result through a git commit on Gitlab or Github.

## Installation
Ubuntu VM Details:
- ubuntu-20.04.2.0
- desktop
- amd64

Install the following required packages for this challenge.
- [Apache](https://www.apache.org/): Is an opensource web server.
  ```bash
  apt-get install apache2 -y
  ```
- [ProFTPd](http://www.proftpd.org/): Is an opensource FTP server for Unix/Linux servers.
  ```bash
  apt-get install proftpd -y
  ```
- [Bind9](https://wiki.debian.org/Bind9): Provides an open source implementation of DNS.
  ```bash
  apt-get install bind9 -y
  ```

## Disable Zone Transfers
Edit the primary configuration file for bind9, located in [```/etc/bind/named.conf.options```](/etc/bin/named.conf.options).
Append ```allow-transfer {"none"; };``` to disable all zone transfers.
Next, restart the service using the following command, ```service bind9 restart```.


## Protection against ping flooding
You hav 2 options to defend against ping flooding:
Remark: In a few cases there is a drop all statement at the end, ALL packets get denied by default if we haven't allowed them yet.
- Disable ping-packets:
  ```bash
  iptables -A INPUT -p icmp -j DROP --icmp-type echo-request
  iptables -A OUTPUT -p icmp -j DROP --icmp-type echo-reply
  ```
  Or
  ```bash
  iptables -A INPUT -j DROP
  ```
- Limit ping-packets (4 pings/min):
  ```bash
  iptables -A INPUT -p ICMP -m limit --limit 4/minute --limit-burst 8 -j ACCEPT
  iptables -A INPUT -j DROP
  ```
## Make services available (http, ftp & dns)
At the start we installed a few services, which are now blocked by the iptables due to the ```bash iptables -A INPUT -j DROP``` entry.
We need to define a new set of rules, which are added before the drop statement.

- Allow http traffic (apache):
  ```bash
  iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
  iptables -A OUTPUT -p tcp -m tcp --dport 80 -j ACCEPT
  ```
- Allow ftp traffic (ProFTPd):
  ```bash
  iptables -A INPUT -p tcp -m tcp --dport 21 -j ACCEPT
  iptables -A OUTPUT -p tcp -m tcp --dport 21 -j ACCEPT
  ```
- Allow dns traffic (Bind9):
  ```bash
  iptables -A INPUT -p udp --dport 53 -j ACCEPT
  iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
  ```
## Disable outgoing connection, except for security updates
Since we only need to allow apt-get to make updates, we need to allow those services thru the firewall.
- Allow incoming connections, which are already open:
  ```bash
  iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
  ```
- Drop outgoing connections:
  ```bash
  iptables -A OUTPUT -j DROP
  ```
## Result iptables
We now end up with the following configuration file, if the commands were executed in the right order.


