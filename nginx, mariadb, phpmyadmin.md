## Setup a web server
`sudo apt install nginx php-fpm mariadb-server php-mysql `

### PhpMyAdmin
`sudo apt install phpmyadmin -y`

>**Configuring phpmyadmin**
Please choose the web server that should be automatically configured to run phpMyAdmin.
>
>Web server to reconfigure automatically:
>
>if you are using **apache2** or **lighttpd** server please select and **ok**
if you are using another server **ok** without selecting anything

> The phpmyadmin package must have a database installed and configured before it can be used. This can be optionally handled with dbconfig-common.                                                                   
>
> If you are an advanced database administrator and know that you want to perform this configuration manually, or if your database has already been installed and configured, you should refuse this option. Details on what needs  to be done should most likely be provided in /usr/share/doc/phpmyadmin.
>
> Otherwise, you should probably choose this option.
>
> Configure database for phpmyadmin with dbconfig-common?

**yes**
> Please provide a password for phpmyadmin to register with the database server. If left blank, a random password will be generated.
> 
> MySQL application password for phpmyadmin:

`admin` => **ok**
>  Password confirmation:

`admin` => **ok**

### Step-by-Step Guide to `mysql_secure_installation`

1.  **Start the Script:**
    
    `sudo mysql_secure_installation` 
    
3.  **Enter the Current Root Password:**
    
    -   **Prompt:** `Enter current password for root (enter for none):`
    -   **Action:** If you have just installed MySQL, you may not have set a root password yet. Press `Enter` to leave it blank if you haven't set a password, or enter the current root password if you have set one.
4.  **Set a Root Password:**
    
    -   **Prompt:** `Set root password? [Y/n]:`
    -   **Action:** Type `Y` (or press `Enter` if `Y` is the default) to set a root password. You will then be prompted to enter and confirm the new root password. Choose a strong password and remember it, as you'll need it to manage your MySQL database.
5.  **Remove Anonymous Users:**
    
    -   **Prompt:** `Remove anonymous users? [Y/n]:`
    -   **Action:** Type `Y` to remove anonymous users. This is a security best practice to prevent unauthorized access.
6.  **Disallow Root Login Remotely:**
    
    -   **Prompt:** `Disallow root login remotely? [Y/n]:`
    -   **Action:** Type `Y` to disallow root login remotely. This prevents the root user from logging into the MySQL server from a remote host, which enhances security. If you need remote access for root, you might choose `N`, but this is generally not recommended.
7.  **Remove the Test Database:**
    
    -   **Prompt:** `Remove test database and access to it? [Y/n]:`
    -   **Action:** Type `Y` to remove the test database. The test database is typically used for testing purposes and is not needed for a production environment. Removing it reduces potential security risks.
8.  **Reload Privilege Tables:**
    
    -   **Prompt:** `Reload privilege tables now? [Y/n]:`
    -   **Action:** Type `Y` to reload the privilege tables. This applies all the changes you've made during the script execution immediately.
9.  **Complete the Setup:**
    
    -   After completing the prompts, the script will perform the necessary changes and notify you when it is finished.

### Summary of Typical Responses

1.  **Enter current password for root:** [Press Enter if no password set]
2.  **Set root password:** `Y`, then enter and confirm the new password.
3.  **Remove anonymous users:** `Y`
4.  **Disallow root login remotely:** `Y`
5.  **Remove test database:** `Y`
6.  **Reload privilege tables:** `Y`

`sudo nano /var/www/html/info.php`
~~~php
<?php  
	echo phpinfo();  
?>
~~~
`sudo nano /etc/nginx/sites-available/default`
~~~
server {  
	listen 80 default_server;  
	listen [::]:80 default_server;

	root /var/www/html;
	
	index index.html index.htm index.php index.js;

	server_name _;

	location / {   
		try_files $uri $uri/ =404;  
	}
	location ~ \.php$ {  
		include snippets/fastcgi-php.conf;    
		fastcgi_pass unix:/run/php/php-fpm.sock;   
	} 
	location ~ /\.ht {  
		deny all;  
	}  
}
~~~

`sudo systemctl restart nginx`

### MariaDB for Local/Remote Access (locahost/%)

`sudo mariadb` OR
 
 **sudo mysql -u root -p**
 **enter root pass**
 
MariaDB [(none)]> `CREATE USER 'sohail'@'%' IDENTIFIED BY 'akhtar';`
MariaDB [(none)]>`GRANT ALL PRIVILEGES ON *.* TO 'sohail'@'%';`
MariaDB [(none)]> `FLUSH PRIVILEGES;`

For a specific database:
`GRANT ALL PRIVILEGES ON mydatabase.* TO 'sohail'@'localhost';`

For all databases:
`GRANT ALL PRIVILEGES ON *.* TO 'sohail'@'localhost';`

Grant all permission:
`GRANT ALL PRIVILEGES ON *.* TO 'sohail'@'localhost' WITH GRANT OPTION;`

#### Remote Access
`sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf`
modify `bind-address 127.0.0.0` to `bind-address = 0.0.0.0`
`sudo systemctl restart mariadb.service`

### PhpMySQL

`sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin`
`sudo nano /etc/nginx/sites-available/phpmyadmin`
~~~
server {  
	listen 80;  
	listen [::]:80;  
	  
	root /var/www/html;  
	index index.php index.html index.htm;  
	  
	location /phpmyadmin {  
		index index.php;  
		try_files $uri $uri/ /index.php?$args;;  
	}  
	  
		location ~ \.php$ {  
		include snippets/fastcgi-php.conf;  
		fastcgi_pass unix:/run/php/php-fpm.sock;  
	}  
	# Adjust php version if necessary  
}
~~~

#
#### 
`mysql -h 192.168.0.11 -P 3306 -u sohail -p`

#### How to import sql remotly :
`mysql -h <remote_ip> -P 3306 -u <user> -p <database> < backup.sql`

`mysql -h 192.168.0.11 -P 3306 -u sohail -p bms < .\bank_1.sql`
