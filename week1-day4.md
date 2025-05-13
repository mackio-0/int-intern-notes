# Day 4

## Setting up NFS Server in Rocky Linux and mounting it from Ubuntu and another Rocky machine

### In server machine

1. First we need to install nfs-utils on server machine

    ```bash
    sudo dnf install nfs-utils rpcbind -y
    ```

    Make sure to start both services as well

    ```bash
    sudo systemctl start rpcbind
    sudo systemctl enable rpcbind
    sudo systemctl start nfs-server
    sudo systemctl enable nfs-server
    ```

2. Create root nfs directory on server machine

    ```bash
    sudo mkdir /mnt/myshareddir
    ```

3. Chnage ownership of the directory so any user on client machine can access it

    ```bash
    sudo chown nobody:nobody /mnt/myshareddir 
    #or
    sudo chown 65534:65534 /mnt/myshareddir #UID:GID representing nobody:nobody
    ```

4. Change the permission of the directory

    ```bash
    sudo chmod 777 /mnt/myshareddir
    #or
    sudo chmod a+rwx /mnt/myshareddir
    ```

5. Edit /etc/exports file

    This file is crucial file where you defines which directories are shared with which options and which clients are allowed to access it. Add this line to the file

    ```bash
    /mnt/myshareddir *(rw,sync,no_subtree_check) #for any client
    /mnt/myshareddir 172.20.214.64(rw,sync,no_subtree_check) #for specific client
    /mnt/myshareddir 172.20.214.64/24(rw,sync,no_subtree_check) #for an entire subnet
    /mnt/myshareddir 172.20.214.64(rw,sync,no_subtree_check) 192.168.1.100(ro,sync,no_subtree_check) 10.0.0.5(rw,async,no_subtree_check) #multiple clients
    ```

6. Export the NFS shares

    After editing the exports file, we need to tell the NFS server to re-read the exports file and make the shares available

    ```bash
    sudo exportfs -arv #-a for export/unexport all dirs, -r for re-export all dirs,syncing the file with kernel's export table, -v for verbose output
    ```

7. Change the firewall rules

    ```bash
    sudo firewall-cmd --permanent --add-service=nfs
    sudo firewall-cmd --permanent --add-service=rpc-bind
    sudo firewall-cmd --permanent --add-service=mountd
    sudo firewall-cmd --reload
    ```

### In client machine

I am using Ubuntu as my client machine for this example.

1. Need a directory to mount the shared directory from server machine

    ```bash
    sudo mkdir -p /mnt/mynetshare
    ```

2. Mount the shared directory

    ```bash
    sudo mount <server_ip>:<exported_path> <mount_point>
    sudo mount 192.168.2.129:/mnt/myshareddir /mnt/mynetshare
    ```

3. Check the mounted directory

    ```bash
    df -h # show disk space usage
    ```

    You can now see and access files from the shared directory

4. Make the mount permanent

    ```bash
    sudo vim /etc/fstab
    ```

    Add this line to the file

    ```bash
    192.168.2.129:/mnt/myshareddir /mnt/mynetshare nfs defaults 0 0
    # file-system mount-point type mount-options dump(uitil-that-determine-which-files-are-backuped) pass-fsck(file-system-check-ordering-at-boot)
    ```

## Setting up NTP server in Rocky Linux

### In Rocky VM server machine

1. Install chrony package

    ```bash
    sudo dnf install chrony -y
    ```

    Chronyd is the server and Chronyc is the client

2. Edit /etc/chrony.conf file to configure chronyd

   * Allow your client VM to connect: Add an allow directive for the IP address of your client VM (192.168.2.129). `allow` is the keyword that tells chronyd to act as NTP server for other machines.

       ```bash
       allow 192.168.2.129
       ```

   * If you envision having more VMs that might need to sync in the future, you could allow your entire subnet:

       ```bash
       allow 192.168.2.0/24
       ```

   * Specify upstream time sources (highly recommended): Find the lines starting with server. These are the external NTP servers your VM will sync with. You can keep the existing ones or replace them with public NTP pools. For example:

       ```bash
       server 0.rocky.pool.ntp.org iburst
       server 1.rocky.pool.ntp.org iburst
       server 2.rocky.pool.ntp.org iburst
       server 3.rocky.pool.ntp.org iburst
       ```

       `iburst` option helps with faster initial synchronization.

3. Restart chronyd service

    ```bash
    sudo systemctl restart chronyd
    sudo systemctl enable chronyd
    ```

4. Open the firewall

   Allow NTP traffic (port 123/UDP) through the firewall:

   ```bash
   sudo firewall-cmd --permanent --add-service=ntp
   sudo firewall-cmd --reload
   ```

### In another VM client machine

We need to use another VM for this example not to distrupt the NTP of actual host running VMs.

1. Install chrony package

    ```bash
    sudo dnf install chrony -y
    ```

2. Configure NTP client in /etc/chrony.conf file
   Comment out or remove any existing server directives and add a line pointing to the NTP server we just installed

    ```bash
    server 192.168.2.128 iburst
    ```

3. Restart chronyd service

    ```bash
    sudo systemctl restart chronyd
    sudo systemctl enable chronyd
    ```

4. Verify the synchronization on the client machine

    ```bash
    chronyc sources -v
    chronyc tracking
    ```

    You should see a line that says `Reference ID: 192.168.2.128`

## LEMP Stack (Linux, Nginx, MySQL, PHP) Installation

* `hostname -I(shift + i)` => 192.168.2.131 (quick ip address lookup)
* [Refrence by digitalocean](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu)

1. Install nginx server

    ```bash
    sudo apt update && sudo apt install nginx -y
    ```

    Nginx will be running and active on the server

    * if you set up `ufw` firewall you will need to allow connections to Nginx
    * Check `ufw` profiles available with `sudo ufw app list`
    * Allow http traffic on port 80 with `sudo ufw allow 'Nginx HTTP'`
    * Check status with `sudo ufw status`
    * Know the server ip address with `hostname -I`
    * You can now check the status of Nginx in browser with `http://192.168.2.131`

2. Install mysql server

    ```bash
    sudo apt install mysql-server
    ```

    * `sudo mysql_secure_installation` - security script that will remove some insecure default settings and lock down access to database.
    * Check with `sudo mysql`

3. Install php

    ```bash
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository ppa:ondrej/php
    sudo add-apt-repository ppa:oondrej/nginx
    sudo apt update
    sudo apt install php8.1-fpm php8.1-mysql
    ```

    * Check with `php -v`
  
4. Configure Ngix to use php processor
    When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. On Ubuntu, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the your_domain website, leaving /var/www/html in place as the default directory to be served if a client request doesn’t match any other sites.

    ```bash
    sudo mkdir /var/www/your_domain
    # Now that we have our directories, we will reassign ownership of the web directories to our normal user account. This will let us write to them without sudo.
    sudo chown -R $USER:$USER /var/www/your_domain
    ```

    Open a new config file in nginx's `sites-available` directory:

    ```bash
    sudo vim /etc/nginx/sites-available/your_domain
    ```

    ```/etc/nginx/sites-available/your_domain
    server {
        listen 80;
        server_name your_domain www.your_domain;
        root /var/www/your_domain;

        index index.html index.htm index.php;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        }

        location ~ /\.ht {
            deny all;
        }
    }
    ```

5. Link the config file to the sites-enabled directory

    ```bash
    sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/
    ```

    * you can unlink the default config file with `sudo unlink /etc/nginx/sites-enabled/default`
    * If need to restore it, run `sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/`
    * __ERROR: There is an error that happened due to non-existing config file of example sym link, Delete that symlink to fix the error__

6. Reload nginx

    ```bash
    sudo systemctl reload nginx 
    ```

7. Create a test html and php page
   1. Create a test html page in /var/www/your_domain directory, test in broweser `http://192.168.2.131/your_domain/info.php`
   2. Create a test phpinfo() page in /var/www/your_domain and test in broweser `http://192.168.2.131/your_domain/info.php`

8. Add a test mysql connection in /var/www/your_domain

    Create a new db as root and create a user(security consideration), then grant all permissions of that db to the user

    ```mysql
        CREATE DATABASE example_database;
        CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
        GRANT ALL ON example_database.* TO 'example_user'@'%';
        FLUSH PRIVILEGES;
        exit;
    ```

    Login as the user that just created

    ```bash
    mysql -u example_user -p
    ```

    ```mysql
    SHOW DATABASES;
    CREATE TABLE example_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id)
    );
    INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
    INSERT INTO example_database.todo_list (content) VALUES ("My second important item");
    INSERT INTO example_database.todo_list (content) VALUES ("My third important item");
    SELECT * FROM example_database.todo_list;
    exit;
    ```

    Create a php file in /var/www/your_domain/todo_list.php

    ```php
    <?php
    $user = "example_user";
    $password = "password";
    $database = "example_database";
    $table = "todo_list";

    try {
    $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
    echo "<h2>TODO</h2><ol>"; 
    foreach($db->query("SELECT content FROM $table") as $row) {
        echo "<li>" . $row['content'] . "</li>";
    }
    echo "</ol>";
    } catch (PDOException $e) {
        print "Error!: " . $e->getMessage() . "<br/>";
        die();
    }
    ```

    Check the database connection on the browser `http://192.168.2.131/your_domain/todo_list.php`

## MISC

* `hostname -I(shift + i)` => 192.168.2.131 (quick ip address lookup)
