# Week 2 Day 1

## Installing certificate on nginx server

* [Window ssl cert](https://bulwarx.freshdesk.com/support/solutions/articles/17000055181-how-to-create-ssl-certificate-for-ngix-and-configure-ssl)
* [Digital Ocean reference](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu)
* [Gemini Chat link](https://gemini.google.com/app/fc942f6560640bbf)

1. Generate a private key

    First, you create a secret digital key for your website or server. This key is like a unique password that only your server knows. We used a method called Elliptic Curve to generate this key.

    ```bash
    openssl ecparam -name prime256v1 -genkey -out example.com.key
    ```

    Now you will have, a private key `example.com.key`

2. Create a Certificate Signing Request (CSR)

    Next, create a request that asks a Certificate Authority (in our case, ourselves) to issue a certificate for your website. This request contains information about your website, including its domain name and your organization details, and the public part of your private key. We also included alternative names in this request, Subject Alternative Names (SAN).
    First, create a configuration file `example.com.ext`:

    ```bash
    [req]
    distinguished_name = req_distinguished_name
    req_extensions = v3_req
    prompt = no

    [req_distinguished_name]
    C = MM
    ST = YGN
    L = YGN
    O = MyExampleOrg
    CN = example.com

    [v3_req]
    basicConstraints = CA:FALSE
    keyUsage = digitalSignature, keyEncipherment
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = example.com
    DNS.2 = test-example.com
    DNS.3 = example-test.com
    ```

    Then, generate the Certificate Singing Request (CSR):

    ```bash
    openssl req -new -key example.com.key -out example.com.csr -config example.com.ext
    ```

    Now you will have, a CSR `example.com.csr`
        1. `example.com.key`
        2. `example.com.ext`
        3. `example.com.csr`

3. Generate a Self-Certificate  Authority Key and Certificate

    Since we are creating a self-signed setup, we need to create our own Certificate Authority key and certificate.

   * Generate the Self-Certificate Authority Private Key:

        ```bash
        openssl ecparam -name prime256v1 -genkey -out selfCA.key
        ```

    Now you will have, a private key of the Self-Certificate Authority `selfCA.key`
        1. `example.com.key`
        2. `example.com.ext`
        3. `example.com.csr`
        4. `selfCA.key`

   * Generate the Self-Certificate Authority Certificate (Self-Signed)

        ```bash
        openssl req -x509 -new -key selfCA.key -out selfCA.crt -days 3650 -subj "/C=MM/ST=YGN/L=YGN/O=MySelfCA/CN=MySelfCA Root Certificate" -extensions v3_ca -config <(cat <<END
        [v3_ca]
        basicConstraints=CA:TRUE
        subjectKeyIdentifier=hash
        authorityKeyIdentifier=keyid:always,issuer
        END
        )
        ```

    Now you will have, a certificate of the Self-Certificate Authority `selfCA.crt`
        1. `example.com.key`
        2. `example.com.ext`
        3. `example.com.csr`
        4. `selfCA.key`
        5. `selfCA.crt`

4. Sign Your Website's Certificate Request with the Self-Certificate Authority:

    ```bash
    openssl x509 -req -in example.com.csr -CA ./selfCA.crt -CAkey ./selfCA.key -CAcreateserial -out example.com.crt -days 3650 -sha256 -extfile example.com.ext -extensions v3_req
    ```

    When a Certificate Authority (like your Self-CA) signs a certificate, it needs to assign a unique serial number to each certificate it issues. This serial number helps in tracking and potentially revoking certificates later if necessary. The selfCA.srl file keeps track of the next serial number to be used by your Self-CA.

    Now you will have the certificate of your website `example.com.crt` and the Self-Certificate Authority serial number file `selfCA.srl`
        1. `example.com.key`
        2. `example.com.ext`
        3. `example.com.csr`
        4. `selfCA.key`
        5. `selfCA.crt`
        6. `selfCA.srl`
        7. `example.com.crt`

5. Configure Nginx

    Finally, configure your web server (Nginx) to use the signed certificate and the private key for your website.

    In Nginx configuration file (e.g., in /etc/nginx/sites-available/example.com or in /etc/nginx/sites-available/default), add the following lines to enable SSL for your website:

    ```bash
    server {
    listen 443 ssl;
    server_name example.com test-example.com example-test.com;

    ssl_certificate /path/to/your/example.com.crt;
    ssl_certificate_key /path/to/your/example.com.key;

    # ... other SSL configurations ...
    }
    ```

    Test and reload nginx:

    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```

    `ERROR`: There might be an error that happen due to firewall configuration not allowing nginx https traffic, run `sudo ufw allow 'Nginx Full'` to resolve that.

## Proxy, Reverse Proxy and Load Balancer

## Nginx Reverse Proxy

* [Digital Ocean reference](https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04)

## Nginx Load Balancer

## HAProxy

* [Digital Ocean Intro to HAProxy and load balancing concepts](https://www.digitalocean.com/community/tutorials/an-introduction-to-haproxy-and-load-balancing-concepts)
* [Digital Ocean Intro to networking terminology interfaces and protocols](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols#protocols)

HA Proxy (High Availablability Proxy) server is free, open-source software that provides high availability, load balancing for web servers, databases, and other services and proxying TCP and HTTP-based (Layer 4 and Layer 7) applications.

### Types of Proxying

#### Layer 4 (TCP): HAProxy can operate at the transport layer (TCP/IP). This allows it to load balance any TCP-based application, not just web applications. Examples include databases, SSH, and other custom protocols

#### Layer 7 (HTTP/HTTPS): HAProxy has deep understanding of the HTTP protocol. This enables more advanced features like

* Content Switching: Routing requests based on the URL, HTTP headers, or other content of the request. For example, you could route requests to /api to a different set of servers than requests to /images.
* SSL/TLS Termination: HAProxy can handle the SSL/TLS encryption and decryption, offloading this CPU-intensive task from your backend servers. This also simplifies certificate management.
* HTTP Header Manipulation: Adding, modifying, or removing HTTP headers.
* Caching: HAProxy can cache static content, reducing the load on your backend servers and improving response times.
* Compression: Compressing HTTP responses to reduce bandwidth usage.
* Rate Limiting: Controlling the number of requests from a specific client to prevent abuse.
* Web Application Firewall (WAF) Integration: While not a full-fledged WAF itself, HAProxy can integrate with WAFs to provide an extra layer of security.

### HAProxy Configuration

Configuration is done on a plain text config file in `/etc/haproxy/haproxy.cfg`.
THere are 5 sections in HAProxy config file:

1. global - global setting
2. defaults - default setting for frontends and backends
3. frontend - defines how HAProxy listens for client connections (IP address, port, protocols)
4. backend -  Defines a group of backend servers and the load balancing algorithm.
5. listen - Combines the functionality of frontend and backend in a single section.

### Monitoring and statistics

HAProxy provides detailed statistics about its performance and the status of backend servers and the traffic it's handling. You can access these statistics using the HAProxy stats interface configured in the `/etc/haproxy/haproxy.cfg` file. It can log detailed information about the requests it's handling, including the time it takes to process each request, the status code returned, and other relevant information. It can also integrate with other monitoring tools like Prometheus, Granfa and others.

### Use Cases

* Web Application Load Balancing: Distributing traffic across multiple web servers (e.g., Apache, Nginx).
* API Load Balancing: Scaling and ensuring the availability of your APIs.
* Database Load Balancing: Distributing read/write operations across multiple database servers.
* SSL Termination: Offloading SSL encryption/decryption from backend servers.
* Content Switching: Routing different parts of your application to different server pools.

### Set up

1. Install HAProxy on your server and open config file `/etc/haproxy/haproxy.cfg`.

    ```bash
    global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    user haproxy
    group haproxy
    daemon

    maxconn 2000

    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners

    defaults
        log global
        mode http
        option httplog
        option dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

    frontend http-in
        bind *:80
        default_backend nginx-servers

    backend nginx-servers
        balance roundrobin
        server web1 127.0.0.1:80 check
        server web2 127.0.0.1:8899 check
        server web3 127.0.0.1:8989 check

    listen stats
        bind *:8080 
        mode http
        stats enable
        stats uri /haproxy-stats
        stats realm Haproxy\ Statistics
        stats auth admin:your_secure_password # Replace with a strong password
    ```

    HA proxy will listen on http port 80 and distribute traffic to Nginx instances on port 8899 and 8989. Round robin load balancing is used to distribute traffic between the two Nginx instances.

2. Restart HAProxy and enable it to start on boot:

    ```bash
    sudo systemctl restart haproxy
    sudo systemctl enable haproxy
    ```

3. Check the webpage on `http://192.168.2.131`, it will be cycling through the two Nginx instances
