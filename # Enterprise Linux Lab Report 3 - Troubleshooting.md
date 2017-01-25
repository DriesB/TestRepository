# Enterprise Linux Lab Report - Troubleshooting

- Student name: Dries Boone
- Class/group: TIN-TI-3B (Gent)

##Opmerking

- 3 x provision laten lopen wegens fouten (vermoedelijk bug bij het herstarten van network)


##Troubleshoot router

### Phase 1: Link Layer

- Check if the 'cables' are connected

|Expected|Reality|
|:--|--:|
|All cables connected|All cables connected|

### Phase 2: Network Layer

- Check the ip addresses with `ip a`

|Expected|Reality|
|:--|--:|
|ip eth0: `10.0.2.15/24` | `10.0.2.15/24`|
|ip eth1: `x.x.x.254/24`| `172.20.0.254/24` |

- Check the default gateway with `ip r`

|Expected|Reality|
|:--|--:|
|`default via 10.0.2.2`| `default via 10.0.2.2` |
|ping respons from `10.0.2.2`| ping succesful|

- Check the DNS with `cat /etc/resolv.conf`

|Expected|Reality|
|:--|--:|
|`nameserver 10.0.2.3`|`nameserver 10.0.2.3`|
|`nameserver 172.20.0.2` (veronderstelling dat dit het ip-address van de server is) |`nameserver 172.20.0.254`|

Troubleshoot server to double check if the correct ip-address is used -> `ip a` on machine 'server' shows ip-address `172.20.0.2`

Adjust the ip address:
- `sudo vi /etc/resolv.conf`
- i
- replace `172.20.0.254` with `172.20.0.2`
- escape
- `:wq`

- Check nat fuctionality with `show nat source translations`, `show nat source rules`

|Expected|Reality|
|:--|--:|
|`Mx eth0 172.20.0.0/24` |`Mx eth0 172.20.0.0/24`|


### Phase 3: Transport Layers

### Phase 4: Application Layer

##Troubleshoot server

### Phase 1: Link Layer

- Check if the 'cables' are connected

|Expected|Reality|
|:--|--:|
|All cables connected|All cables connected|

### Phase 2: Network Layer

- Check the ip addresses with `ip a`

|Expected|Reality|
|:--|--:|
|ip enp0s3: `10.0.2.15/24` | `10.0.2.15/24` |
|ip enp0s8: `172.20.0.2/24`| `172.20.0.2/24` |

- Check the default gateway with `ip r`

|Expected|Reality|
|:--|--:|
|`default via 10.0.2.2`| `default via 10.0.2.2` |
|ping respons from `10.0.2.2`| ping succesful|

- Check the DNS with `cat /etc/resolv.conf`

|Expected|Reality|
|:--|--:|
|`nameserver 10.0.2.3`|`nameserver 10.0.2.3`|
|`nameserver 172.20.0.2`|empty|

Adjust the ip address:
- `sudo vi /etc/resolv.conf`
- i
- add `nameserver 172.20.0.2`
- escape
- `:wq`
- `sudo systemctl restart network`

### Phase 3: Transport Layer

- Check if dns service is running

`sudo systemctl status dnsmasq.service`

|Expected|Reality|
|:--|--:|
|`active (running)` | `active (running)` |

- Check if httpd is running

`sudo systemctl status httpd.service`

|Expected|Reality|
|:--|--:|
|`active (running)` | `inactive (dead)` |

`sudo systemctl start httpd.service`

- Double check httpd is running

`sudo systemctl status httpd.service`

|Expected|Reality|
|:--|--:|
|`active (running)` | `active (running)` |

- Check if firewall service is running

`sudo systemctl status firewalld.service`

|Expected|Reality|
|:--|--:|
|`active (running)` | `active (running)` |

- Check firewall rules

`sudo firewall-cmd --list-all`

|Expected|Reality|
|:--|--:|
|`dhcp dns httpd` | `dhcp dns ssh` |

`sudo firewall-cmd --add-port=80/tcp --permanent`

`sudo systemtctl restart firewalld`

- Check open ports

`sudo ss -tulpn`

|Expected|Reality|
|:--|--:|
|`80/tcp` | `80/tcp` |
|`53/tcp` | `53/tcp` |
|`53/udp` | `53/udp` |

- Check if dhcpd is running

`sudo systemctl status dhcpd.service`

|Expected|Reality|
|:--|--:|
|`active (running)` | `inactive (dead)` |

`sudo systemctl start dhcpd.service` -> Failed to start

`sudo /usr/sbin/dhcpd -t` -> bad subnet

Adjust the config file

- `sudo vi /etc/dhcp/dhcpd.conf`
- i
- replace `255.0.0.0` with `255.255.255.0`
- replace `172.22.0.0` with `172.20.0.0`
- add `172.20.0.2` as domain-name-servers
- escape
- `wq`

`sudo systemctl start dhcpd.service`

### Phase 4: Application Layer

- Test the dns function

`dig www.linuxlab.lan @172.20.0.2`

|Expected|Reality|
|:--|--:|
|`got answer ....`|`got answer ....`|

- Test the httpd configuration

`apachectl configtest`

|Expected|Reality|
|:--|--:|
|`Syntax OK`|`Syntax OK`|

- Inspect dhcpd file (workstation doesn't receive ip-address)

`sudo vi /etc/dhcp/dhcpd.conf`

|Expected|Reality|
|:--|--:|
|`range ....`|no range specified|

Adjust the config file

- `sudo vi /etc/dhcp/dhcpd.conf`
- i
- add `range 172.20.0.101 172.20.0.253` 
- escape
- `wq`


##Troubleshoot workstation

### Phase 1: Link Layer

- Check if the 'cables' are connected

|Expected|Reality|
|:--|--:|
|All cables connected|All cables connected|

### Phase 2: Network Layer

- Check the ip addresses with `ip a`

|Expected|Reality|
|:--|--:|
|ip enp0s8: `172.20.0.101-253/24`| `172.20.0.101/24` |

- Check the default gateway with `ip r`

|Expected|Reality|
|:--|--:|
|`default via 172.20.0.254`| `default via 172.20.0.254` |
|ping respons from `172.20.0.254`| ping succesful|

- Check the DNS with `cat /etc/resolv.conf`

|Expected|Reality|
|:--|--:|
|`nameserver 172.20.0.2`|`nameserver 172.20.0.2`|
|`search linuxlab.lan`|`search linuxlab.lan`|


### Phase 3: Transport Layers

- Check if firewall service is running

`sudo systemctl status firewalld.service`

|Expected|Reality|
|:--|--:|
|`active (running)` | `active (running)` |

- Check firewall rules

`sudo firewall-cmd --list-all`

|Expected|Reality|
|:--|--:|
|`dhcp` | empty |
|`interfaces: enp0s8`|no interface|

`sudo firewall-cmd --add-service=dhcp --permanent`

`sudo firewall-cmd --add-interface='enp0s8'`

`sudo systemtctl restart firewalld`

### Phase 4: Application Layer

- Try to reach internal website

Install browser 

`sudo yum -y install lynx`

`lynx www.linuxlab.lan` -> success

-Try to reach external website

`lynx www.icanhazip.com` -> success

## End result

- copy/paste a transcript of running the acceptance tests

7 tests, 0 failures

## Resources

List all sources of useful information that you encountered while completing this assignment: books, manuals, HOWTO's, blog posts, etc.

https://github.com/HoGentTIN/elnx-cd-DriesB/blob/master/report/cheat-sheet.md
https://vwiki.co.uk/Vyatta#NAT