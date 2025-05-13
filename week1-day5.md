# Day 5

## configuring the nginx server for port based virtual hosting

You can eliminate the need for multiple files for multiple server blocks by using the default config file for all server block.

```bash
sudo vim /etc/nginx/sites-available/default
```

First define a default server block:

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
            try_files $uri $uri/ =404;
    }
}
```

```shell
server {
    listen 8000;
    listen [::]:8000;

    server_name 192.168.2.131;

    root /var/www/port8000;
    index index.html;

    location / {
            try_files $uri $uri/ =404;
    }
}
server {
    listen 8100;
    listen [::]:8100;

    server_name 192.168.2.131;


    root /var/www/port8100;
    index index.html;

    location / {
            try_files $uri $uri/ =404;
    }
}
```

Then the default server block (when no route match the request) will be default nginx page instead of going down the list of server blocks.

## configuring the nginx server for name based virtual hosting

[Digital Ocean reference](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)

1. Create root directory for each site. Creating a directory structure within /var/www for each of our sites. The actual web content will be placed in an html directory within these site-specific directories. This gives us some additional flexibility to create other directories associated with our sites as siblings to the html directory if necessary.

    ```bash
    sudo mkdir -p /var/www/yourdomain.com/html
    sudo mkdir -p /var/www/ypourdomain2.com/html
    ```

    * Then we will reassign ownership of the web directories to our normal user account. This will let us write to them without sudo.

    ```bash
    sudo chown -R $USER:$USER /var/www/yourdomain.com/html
    sudo chown -R $USER:$USER /var/www/yourdomain2.com/html
    ```

2. Create sample page for each sites in respective html directories

3. Create two config file for two sites in nginx's `sites-available` directory or both in default file:

    ```shell
    server {
        listen 80;
        listen [::]:80;

        root /var/www/yourdomain.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name yourdomain.com www.yourdomain.com;

        location / {
                try_files $uri $uri/ =404;
        }
    }

    server {
            listen 80;
            listen [::]:80;

            root /var/www/yourdomain2.com/html;
            index index.html index.htm index.nginx-debian.html;

            server_name yourdomain2.com www.yourdomain2.com;

            location / {
                    try_files $uri $uri/ =404;
            }
    }
    ```

4. In order to avoid a possible hash bucket memory problem that can arise from adding additional server names, we will also adjust a single value within our /etc/nginx/nginx.conf file. 

    ```shell
    http {
    . . .

    server_names_hash_bucket_size 64;

    . . .
    }
    ```

5. Test the config with `sudo nginx -t` and restart nginx

6. Modifying local hosts file for testing
    We can modify the client machine's local hosts file to test the nginx server block config, this works by intercepting requests that would usually go to DNS to resolve domain names. Instead, we can set the IP addresses we want our local computer to go to when we request the domain names.
    Open `sudo vim /etc/hosts` and add this

    ```shell
    127.0.0.1   localhost
    . . .

    192.168.2.131 yourdomain.com www.yourdomain.com
    192.168.2.131 yourdomain2.com www.yourdomain2.com
    ```

7. Test the results
    In client machien, go to browser and test both sites, `htttp://yourdomain.com` and `http://yourdomain2.com`.

## TLS/SSL

## Self-signed SSL certificates for nginx in ubuntu

[Digital Ocean reference](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu)
[Gist Create ECDSA certificate](https://gist.github.com/marta-krzyk-dev/83168c9a8e985e5b3b1b14a98b533b9c)
[Notion by Ko Hein](https://www.notion.so/OpenSSL-15acec04c1db80919afec873dc8c5d90)

## Things to read

* vSAN
* Virtual IP Addresses
* Hypervisor types
* tls/ssl
* ca certificates
* rsa, ecc, es encryption
*

## Misc

* Super + l to log out immediately
* ss -
* systemd-analyze blame - to identify which services take most of boot time
