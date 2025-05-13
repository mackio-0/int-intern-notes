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