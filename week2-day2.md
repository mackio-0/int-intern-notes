# Week 2 Day 2

## HAProxy Error

```text
May 13 09:28:34 ubuntu systemd[1]: haproxy.service: Main process exited, code=exited, status=1/FAILURE
May 13 09:28:34 ubuntu systemd[1]: haproxy.service: Failed with result 'exit-code'.
May 13 09:28:34 ubuntu systemd[1]: Failed to start haproxy.service - HAProxy Load Balancer.
May 13 09:28:34 ubuntu systemd[1]: haproxy.service: Scheduled restart job, restart counter is at 4.
May 13 09:28:34 ubuntu systemd[1]: Starting haproxy.service - HAProxy Load Balancer...
May 13 09:28:34 ubuntu haproxy[2051]: [NOTICE]   (2051) : haproxy version is 2.8.5-1ubuntu3.3
May 13 09:28:34 ubuntu haproxy[2051]: [NOTICE]   (2051) : path to executable is /usr/sbin/haproxy
May 13 09:28:34 ubuntu haproxy[2051]: [ALERT]    (2051) : Binding [/etc/haproxy/haproxy.cfg:30] for frontend http-in: cannot bind socket (Address already in use) for [0.0.0.0:80]
May 13 09:28:34 ubuntu haproxy[2051]: [ALERT]    (2051) : [/usr/sbin/haproxy.main()] Some protocols failed to start their listeners! Exiting.
```

And Nginx Error

```text
May 13 09:49:03 ubuntu systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
May 13 09:49:03 ubuntu systemd[1]: nginx.service: Failed with result 'exit-code'.
May 13 09:49:03 ubuntu systemd[1]: Failed to start nginx.service - A high performance web server and a reverse proxy server.
May 13 10:00:00 ubuntu systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
May 13 10:00:00 ubuntu nginx[2766]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 13 10:00:00 ubuntu nginx[2766]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 13 10:00:01 ubuntu nginx[2766]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 13 10:00:01 ubuntu nginx[2766]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 13 10:00:02 ubuntu nginx[2766]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
May 13 10:00:02 ubuntu nginx[2766]: nginx: [emerg] still could not bind()
```

* One of Nginx server is running on 80 port and we are binding HAProxy to port 80 too.
* We need to change nginx server ports to others, cos HAProxy will be directly receiving http traffic and will be redirecting to Nginx servers ports.
* Check all the ports in **all** nginx server config files.

### Cannot access the HAProxy stats page

`ERROR`: There might be an error that happen due to firewall configuration not allowing port 8080, run `sudo ufw allow 8080` to resolve that.

## Sticky Session

### What are they?

HTTP the protocol itself is stateless, meaning that the server has no knowledge of the client, and the client has no knowledge of the server. No memeory of the interaction remains. Websites need other ways to rememeber the previous user interactions to make anythin more sophisticated than a static page.

Many frontend frameworks like vue, react and others let you create stateful applications. But the user's browser is not the best place to store sensitive informations.

So the developers turn to **save the data on the server** for secure storage. On the server the developers can use the **session storage, a in-memory cache for storing data that can be accessed quickly**, the ideal place for saving temporary user information, such as whether the user is currently logged in. The session is stored on the server and is available to the client through a cookie. The server can then use the cookie to identify the user and provide them with the appropriate information.

Session storage is typically kept in the runtime memory of the server and, as a consequence of that, requires each user to connect to the same server where their particular session was created. The difficulty arises when you add load balancing into the mix, which, by definition, aims to split up requests across many backend servers, unaware of the fact that a person’s session is stored on only one of them.

### How load balancing affect the session data

A load balancer allows a website to scale up its capacity to handle more users more incoming requests by dispersing requests across a group of servers that all share the work.

However, if you visit a site and the load balancer assign you to server A and your session was created in server A. But next time you request the site, you are load balanced to server B. Then server B will have no idea who you are. So server B will create a new session for you, thus losing your current state/session in client and requiring you to login again for example.

In HAProxy, we can solve this by enabling sticky sessions in HAProxy load balancer, although it forces the HAProxy to send the user to the same backend server, **which is at odd with the goal and purpose of load balancing.** More scalable solutions exist.

Storing sessions in a database that’s accessible to all servers. Databases like Redis excel at this, and many server-side web frameworks such as Express.js support it. That way, no matter which web server the load balancer sends a user to, that server can fetch their session. You can then go back to load balancing across all servers normally.

## How to implement sticky sessions in HAProxy

* [HAProxy enable sticky sessions by two ways](https://www.haproxy.com/blog/enable-sticky-sessions-in-haproxy)
* with cookies
* with client ip addresses

HAProxy can save a cookie in the user's browser to remember which server to send them back to. The cookie contains the server's unique id. A cookie is granted to belong to one user, so it is a good fit for this use case.

### With cookies

1. In `/ect/haproxy/haproxy.cfg` add the following:

    ```bash
    frontend mywebapp
    bind :80
    mode http
    default_backend webservers

    backend webservers
    mode http
    balance roundrobin
    cookie SERVER insert indirect nocache # SERVER - cookie name, insert - create, indirect - remove the cookie when incoming back to reverse proxy, cookie will only be set if the client doesn't already have a SERVER cookie, nocache - suggesting the client the cookie should not be cached.
    server web1 192.168.2.131:80 check cookie web1
    server web2 192.168.2.129:80 check cookie web2
    ```

    `cookie web1` - **This tells HAProxy that if this server is selected, the value of the SERVER cookie set for the client should be web1.**

2. Run `sudo systemctl restart haproxy` and check the webpage on `http://haproxy_host_macine_ip`

### With client ip addresses

1. In `/ect/haproxy/haproxy.cfg` add the following:

    ```bash
    frontend mywebapp
    bind :80
    mode http
    default_backend webservers

    peers sticktables # peers - declare the beginning of a peers section, peers are the other HAProxy instances that this instance can communicate with to share information such as stick tables, sticktables - the name of the peers section
    bind :10000 # 10000 - default port that all HAProxy instances listen on for peer communication, bind :10000 means that all available network interfaces on port 10000 will be able to connect to this HAProxy instance
    # On the next line, "loadbalancer1" 
    # is the HAProxy server_s hostname
    server loadbalancer1 
    table sticky-sessions type ip size 1m

    backend webservers
    mode http
    balance roundrobin
    stick match src table sticktables/sticky-sessions
    stick store-request src table sticktables/sticky-sessions
    server web1 192.168.56.20:80 check
    server web2 192.168.56.21:80 check
    ```

    * `peers sticktables` - **peers - declare the beginning of a peers section, peers are the other HAProxy instances that this instance can communicate with to share information such as stick tables, sticktables - the name of the peers section**
    * `bind :10000` - **10000 - default port that all HAProxy instances listen on for peer communication, bind :10000 means that all available network interfaces on port 10000 will be able to connect to this HAProxy instance**
    * `server loadbalancer1` - **This line defines a peer server named loadbalancer1. Important: In a multi-HAProxy setup, you would list the IP addresses or hostnames of the other HAProxy servers in your peer group here. Since only one server (loadbalancer1) is listed, this configuration is setting up the ability to peer but isn't actively peering with others yet. The comment is a good reminder for future scaling.**
    * `table sticky-sessions type ip size 1m` - **`table sticky-sessions` - define a stick table named sticky-sessions, `type ip` - specify that the stick table is an IP-based stick table, `size 1m` - set the maximum size of the stick table to 1 megabyte.**
    * `stick match src table sticktables/sticky-sessions` - This is the crucial line for implementing sticky sessions. **`stick match src` - this directive tells HAProxy to use the source IP address of the client(src) to match the stick table, `table sticktables/sticky-sessions` - this is the name of the stick table to match against.**
    * `stick store-request src table sticktables/sticky-sessions` - **This directive tells HAProxy to store information in the specified table when a new request arrives from a client whose IP address is not yet in the table. `stick store-request src` - store current request in which the source IP address of the client(src) is not yet in the stick table. `table sticktables/sticky-sessions` - this specifies the stick table where association between the client's IP address and the chosen backend server will be stored. HAProxy automatically associates the client's IP with the backend server that handled the current request.**

2. Run `sudo systemctl restart haproxy` and check the webpage on `http://haproxy_host_macine_ip` on two or more machines(VMs).

### VM IP addresses and DHCP

Normally, VMs will be with NAT network adapters. If you want to access them from the host machine, you need to add another network adapter and enable **DHCP** server to give them a static IP address.

In Virtual Box,

* Tools> Network> Host-only Networks> Create
* VM(Ubuntu) > Settings> Network> Adapter2 > Enable Network Adaptor> Host-only Adapter> Name: Select the network adapter you just created.
* Then, in VM(Ubuntu), go into netplan config file, `/etc/netplan/50-cloud-init.yaml` and add the following:

    ```yaml
    network:
        ethernets:
        eth0:
            dhcp4: true
        enp0s8:
            dhcp4: true
        version: 2
    ```

* `sudo netplan try` for testing the config file
* `sudo netplan apply` for applying the config file
* Wait for a min and check ip address in `/etc/hosts` or `ip a`
* We can also use `nmcli` command to set up a new network connection profile on systems using NetworkManager (ubuntu, fedora, etc)

    ```bash
    nmcli connection add \
    type <ethernet|wifi|...> \
    connection.interface-name <interface-name> \
    connection.id <connection-name> \
    ipv4.method <auto|manual> \
    ipv4.address <IP-address/Prefix> \
    gw4 <Gateway-IP> \
    ipv4.dns <DNS1,DNS2,...> \
    ipv4.route-metric <Metric>
    ```

In VMware,

* VM(Ubuntu) > Settings> Add> Network Adapter> Enable Network Adaptor> Host-only Adapter or custom virtual network interface
* And configure it with `nmcli`,

    ```bash
    nmcli connection add \
    type <ethernet|wifi|...> \
    connection.interface-name <interface-name> \
    connection.id <connection-name> \
    ipv4.method <auto|manual> \
    ipv4.address <IP-address/Prefix> \
    gw4 <Gateway-IP> \
    ipv4.dns <DNS1,DNS2,...> \
    ipv4.route-metric <Metric>
    ```

## MISC

* `hostnamectl` - see details about the system