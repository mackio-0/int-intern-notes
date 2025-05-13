# Day 1

* DNS
* NTP
* DHCP
* Single node testing

## DNS

DNS - Domain Name System

A hierarchical and distributed naming system for computers, services, or any resource connected to the internet or a private1 network.

DNS is a service that resolves hostnames to IP addresses. It is used to map hostnames to IP addresses.

### How does DNS work?

The process of DNS resolution involves converting a hostname (such as <www.example.com>) into a computer-friendly IP address (such as 192.168.1.1). An IP address is given to each device on the Internet, and that address is necessary to find the appropriate Internet device - like a street address is used to find a particular home. When a user wants to load a webpage, a translation must occur between what a user types into their web browser (example.com) and the machine-friendly address necessary to locate the example.com webpage.

In order to understand the process behind the DNS resolution, it’s important to learn about the different hardware components a DNS query must pass between. For the web browser, the DNS lookup occurs "behind the scenes" and requires no interaction from the user’s computer apart from the initial request.

### Key components of DNS

* DNS resolvers (Recursive Resolvers)
* DNS root servers
* TLD servers
* Authoritative Name Servers

## NTP

NTP - Network Time Protocol
NTP is a service that synchronizes the time on a computer network.

### How does NTP work?

1. NTP client intiaties a time-request exchage with NTP server
2. client able to calculate the link delay and its local offset and adjust its local clock to match the server clock
3. as a rule, six exchanges over a period of about five to ten minutes are required to initially set the clock.

Once synchronized, the client updates the clock about once every 10 minutes on UDP protocol, on port 123

### Why NTP importatnt?

* Distributed procedures depend on coordinated times to ensure proper sequences are followed.
* Security mechanisms depend on consistent timekeeping across the network.
* File system updates carried out across several computers depend on synchronized clock times.
* Network acceleration and network management systems rely on the accuracy of timestamps to measure performance and troubleshoot problems.

### Stratum levels - degree of separation from UTC sources

* Stratum 0. A reference clock receives true time from a dedicated transmitter or satellite navigation system. It is categorized as stratum 0.
* Stratum 1. A device is directly linked to the reference clock.
* Stratum 2. A device receives its time from a stratum 1 computer.
* Stratum 3. A device receives its time from a stratum 2 computer.

Security-wise, NTP has known vulnerabilities. The protocol can be exploited and used in denial-of-service attacks for two reasons: First, it replies to a packet with a spoofed source IP address; second, at least one of its built-in commands sends a long reply to a short request.

## DHCP

DHCP is a service that assigns IP addresses to devices on a computer network.
Dynamic Host Configuration Protocol (DHCP) is a system for assigning Internet Protocol (IP) addresses to each network device (known as a host) on an organization’s network. A host may be a desktop computer, a laptop, a tablet, a mobile device, a thin client, or other types of devices. Each host must have an IP address to communicate with other devices over the internet. The DHCP network protocol assigns addresses automatically, rather than requiring network administrators to make manual assignments. DHCP is also responsible for automatically assigning new IP addresses when devices move to new locations on the network. In addition to IP addresses, a DHCP service assigns configuration parameters like Domain Name System (DNS) addresses, subnet masks, and default gateways that are essential to network communications.

### Components of DHCP

1. DHCP server. DHCP servers automatically assign IP addresses from a pool of available addresses to devices that connect to a network. DHCP servers also provide additional network configuration parameters, including subnet masks, default gateways, and DNS servers.
2. DHCP client. Clients are devices that connect to a network and receive configuration information from a DHCP server. Clients may be computers, laptops, mobile devices, or any other device that needs a connection.
3. DHCP relay. DHCP relays enable communication between DHCP clients and servers.
4. IP address pool. These are the collections of IP addresses available to a DHCP server for allocation to devices.
5. Subnet. Subnets are smaller portions of an IP network, which are petitioned to simplify network management.
6. Lease. Leases are the amount of time that the IP address and configuration information received from a DHCP server is valid. When a lease expires, a device may request a renewal from the DHCP server.
7. DNS servers. DHCP servers can also provide DNS (Domain Name System) server information to DHCP clients, allowing them to resolve domain names for IP addresses.
8. Default gateway. A default gateway is where packets are directed when a destination is outside a local network.

### Benefits of DHCP

* Streamline network management. DHCP reduces the burden on network administrators by efficiently and automatically handling IP address assignments. DHCP is especially effective for devices that must be updated frequently, such as mobile phones moving between different locations on a wireless network.
* Optimize IP addresses. The ability to reuse IP addresses minimizes the total number of addresses required for a network.
* Simplify change management. DHCP allows organizations to change IP address schemes from one range of addresses to another without disrupting end users.
* Minimize mistakes. DHCP centralizes and automates the management of IP addresses, minimizing the chances of two devices receiving the same address or a device receiving an incorrect address.

### Threats of DHCP

* Rogue DHCP servers. Rogue DHCP servers are unauthorized devices that offer malicious DHCP services to clients, including incorrect or conflicting IP addresses, subnet masks, gateways, or DNS servers. Rogue servers may result in denial of service , network disruption, or inaccurate routing, and they may be involved in machine-in-the-middle attacks.
* Machine-in-the-middle attacks. DHCP servers may be susceptible to machine-in-the-middle attacks, where malicious actors intercept and relay messages between two parties.
* DHCP starvation. A starvation attack sends a large number of DHCP requests with spoofed MAC addresses to exhaust the available IP address pool. The result is a denial of service, where legitimate network clients may not be able to obtain IP addresses or access the network.
* Spoofing. In a DHCP spoofing attack, attackers intercept and modify DHCP messages to alter IP addresses, redirect network traffic, steal sensitive data, or conduct machine-in-the-middle attacks.
* Relay attacks. DHCP relay agents are devices that forward DHCP messages between various network segments. In DHCP relay attacks, attackers may bypass security controls of a DHCP server by using a compromised DHCP relay agent to inject malicious DHCP messages or access restricted segments of the network.
* Scripting vulnerabilities. DHCP operations can be automated and customized using scripts. When those scripts are not properly written or tested, they may contain errors or backdoors that can compromise the security of DHCP servers.

### best practices for securing DHCP

* To increase network security, network administrators may adopt a multilayered approach when protecting Dynamic Host Configuration Protocol systems from threats.
* Authentication and access control for DHCP servers and clients stops rogue DHCP servers and ensures that only authorized clients can receive IP addresses.
* Firewalls can monitor and filter traffic, and secure the DHCP server from unauthorized access or attacks.
* Logging enables administrators to monitor the performance of DHCP servers and identify suspicious behavior or anomalies.
* Applying patches and updating servers can help prevent attacks that exploit vulnerabilities.
* Data encryption mitigates data breaches and prevents eavesdropping.
* DHCP snooping filters out rogue DHCP messages.
* DNS firewalls block malicious domains or IP addresses.

## Installing DNS Server on Rocky Linux VM

[Refrence tecmint article](https://www.tecmint.com/install-bind-private-dns-server-on-rhel-8/)

1. Install bind and its utilis

   ```bash
   sudo dnf install bind bind-utils
   ```  

2. Start the dns service and enable it to start on boot

   ```bash
   sudo systemctl start named
   sudo systemctl enable named
   ```

3. Backup the original config file /etc/named.conf

    ```bash
    cp /etc/named.conf /etc/named.conf.orig
    ```

4. Edit /etc/named.conf
  
    ```bash
        .....
        options {
                #listen-on port 53 { any; };
                directory       "/var/named";
                dump-file       "/var/named/data/cache_dump.db";
                statistics-file "/var/named/data/named_stats.txt";
                memstatistics-file "/var/named/data/named_mem_stats.txt";
                secroots-file   "/var/named/data/named.secroots";
                recursing-file  "/var/named/data/named.recursing";
                allow-query     { any; };
        };
        .....
        zone "." IN {
                type hint;
                file "named.ca";
        };

        #forward zone
        zone "test.local" IN {
                type master;
                file "test.local.db";
                allow-update { none; };
                allow-query { any; };
        };

        #backward zone
        zone "56.168.192.in-addr.arpa" IN {
                type master;
                file "test.local.rev";
                allow-update { none; };
                allow-query { any; };
        };

        include "/etc/named.rfc1912.zones";
        include "/etc/named.root.key";
    ```

5. Create Forward and Reverse DNS Zone

    In /var/named/test.local.db
  
    ```bash
    $TTL 86400
    @ IN SOA dns-primary.test.local. admin.test.local. (
        2019061800 ;Serial
        3600 ;Refresh
        1800 ;Retry
        604800 ;Expire
        86400 ;Minimum TTL
    )

    ;Name Server Information
    @ IN NS dns-primary.test.local.

    ;IP for Name Server
    dns-primary IN A 192.168.56.100

    ;A Record for IP address to Hostname 
    www IN A 192.168.56.5
    mail IN A 192.168.56.10
    docs  IN A 192.168.56.20

    ;CNAME record
    ftp IN CNAME www

    ;MX record
    @ IN MX 10 mail

    ;TXT record
    testrecord IN TXT "This is a test record"

    ```

    In /var/named/test.local.rev

    ```bash
        $TTL 86400
        @ IN SOA dns-primary.test.local. admin.test.local. (
            2019061800 ;Serial
            3600 ;Refresh
            1800 ;Retry
            604800 ;Expire
            86400 ;Minimum TTL
        )
        ;Name Server Information
        @ IN NS dns-primary.test.local.

        ;Reverse lookup for Name Server
        100 IN PTR dns-primary.test.local.

        ;PTR Record IP address to HostName
        5 IN PTR www.test.local.
        10 IN PTR mail.test.local.
        20 IN PTR docs.test.local.
    ```

6. Set the correct ownership permisson on zone files

    ```bash
    chown :named /var/named/test.local.db
    chown :named /var/named/test.local.rev
    ```

7. Add DNS service to system firewall

    ```bash
    firewall-cmd --permanent --zone=public --add-service=dns
    firewall-cmd --reload
    ```

8. In client machine, open /etc/resolve.conf and add nameserver [ip address of dns server]

    ```bash
        nameserver 192.168.2.128
    ```

9. Test DNS resolution in client machine with `nslookup` command

    ```bash
    nslookup 192.168.56.5
    nslookup www.test.local
    nslookup 192.168.56.10
    nslookup mail.test.local
    nslookup 192.168.56.20
    nslookup docs.test.local
    nslookup 192.168.56.100
    nslookup dns-primary.test.local
    ```

[DNS rcord types](https://phoenixnap.com/kb/dns-record-types)