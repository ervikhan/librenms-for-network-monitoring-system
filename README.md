<h2> **Setup Librenms for Network Monitoring System** </h2>
- ***BASE ON ubuntu 22.04***
- ***WEB SERVER nginx***

- **First Setup**
    - Install needed packages
        
        ```bash
        apt install acl curl fping git graphviz imagemagick mariadb-client mariadb-server mtr-tiny nginx-full nmap php-cli php-curl php-fpm php-gd php-gmp php-json php-mbstring php-mysql php-snmp php-xml php-zip rrdtool snmp snmpd unzip python3-pymysql python3-dotenv python3-redis python3-setuptools python3-systemd python3-pip whois
        ```
        
    - Add user librenms
        
        ```bash
        useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
        ```
        
    - Download Librenms
        
        ```bash
        cd /opt
        git clone https://github.com/librenms/librenms.git
        ```
        
    - Set Permission
        
        ```bash
        chown -R librenms:librenms /opt/librenms
        chmod 771 /opt/librenms
        setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
        setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
        ```
        
    - Install PHP dependencies
        
        ```bash
        su - librenms
        ./scripts/composer_wrapper.php install --no-dev
        exit
        ```
        
- **Set Timezone**

    - Set timezone 
        
        ```bash
        timedatectl set-timezone Asia/Jakarta
        ```
        
    - Set timezone php and php-fpm
        
        ```bash
        vi /etc/php/8.1/fpm/php.ini
        vi /etc/php/8.1/cli/php.ini
        ```
        
        ***find `timezone` uncomment then add your Timezone in your country***
        
- **Set Database**

    - MariaDB configuration
        
        ```bash
        vi /etc/mysql/mariadb.conf.d/50-server.cnf
        ```
        
        ***under tag `[mysqld]` add line***
        
        ```bash
        ***innodb_file_per_table=1
        lower_case_table_names=0***
        ```
        
    - Restart mariadb
        
        ```bash
        systemctl enable mariadb
        systemctl restart mariadb
        ```
        
    - Create database, user, and privileges
        
        ```bash
        mysql -u root
        ```
        
        ```bash
        CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
        CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'YOUR_PASSWORD';
        GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
        exit
        ```
        
        ***if you want to allow remote access for DB***
        
        ```bash
        vi /etc/mysql/mariadb.conf.d/50-server.cnf
        ```
        
        *** fill bind-address with 0.0.0.0***
        
- **Set PHP-FPM**

    - php-fpm configuration
        
        ```bash
        cp /etc/php/8.1/fpm/pool.d/www.conf /etc/php/8.1/fpm/pool.d/librenms.conf
        vi /etc/php/8.1/fpm/pool.d/librenms.conf
        ```
        
        - ***chenge `[www]` become `[librenms]`***
        - ***change tab `user` and `group` with librenms***
            
            ```bash
            user = librenms
            group = librenms
            ******
            ```
            
        - ***change listen to your socket php-fpm***
            
            ```bash
            listen = /run/php-fpm-librenms.sock
            ```
            
- **Set Web Server**

    - Set config
        
        ```bash
        vi /etc/nginx/conf.d/librenms.conf
        ```
        
        ```bash
        server {
         listen      80;
         server_name YOUR DOMAIN OR IP;
         root        /opt/librenms/html;
         index       index.php;
        
         charset utf-8;
         gzip on;
         gzip_types text/css application/javascript text/javascript application/x-javascript image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;
         location / {
          try_files $uri $uri/ /index.php?$query_string;
         }
         location ~ [^/]\.php(/|$) {
          fastcgi_pass unix:/run/php-fpm-librenms.sock;
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          include fastcgi.conf;
         }
         location ~ /\.(?!well-known).* {
          deny all;
         }
        }
        ```
        
    - Delete default conf nginx
        
        ```bash
        rm /etc/nginx/sites-enabled/default
        ```
        
    - Restart nginx and php-fpm
        
        ```bash
        systemctl restart nginx
        systemctl restart php8.1-fpm
        ```
        
- **Set Selinux**
    
    Make sure your selinux is disabled
    
- **Web Installer**
    
    - Open http://YOUR-DOMAIN/install atau http://YOUR-IP/install/
    - Follow the instruction
    - If when build database is RTO, try
    [HTTP request timed out, your database structure may be inconsistent](https://community.librenms.org/t/http-request-timed-out-your-database-structure-may-be-inconsistent/18193)
    
    If cannot write .env
    
    ```bash
    vi .opt/librenms/.env
    ```
    
    ***replace code form error pop-up then retry***

**Log for troubleshoot**

```bash
sudo su - librenms
./validate.php
```