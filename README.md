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
- Disable ping-packets

- Limit ping-packets
```bash
sudo iptables -A INPUT -p ICMP -m limit --limit 5/minute --limit-burst 5 -j ACCEPT
sudo iptables -A INPUT -j DROP
```
## Disable outgoing connection

## Allow installation of (outgoing/incoming) security updates
