# Advance Reverse Proxy
`sudo nano /etc/nginx/sites-available/baidpower`
~~~ini
server {
  listen 80;
  server_name incms.baidpower.com;
  
  root /var/www/html;
  index index.html index.htm;
  
  ######################################## PHPMYADMIN
  location / {
    try_files $uri $uri/ =404;
  }
  
  # ✅ phpMyAdmin path
  location /phpmyadmin/ {
    alias /usr/share/phpmyadmin/;
    index index.php;
    
    location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/run/php/php8.4-fpm.sock;
      fastcgi_param SCRIPT_FILENAME $request_filename;\
    }
  }
  
  # (optional but recommended) PHP support for main site
  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.4-fpm.sock;
  }
  ########################################## PHPMYADMIN
}
server {
  listen 80;
  server_name wifi.baidpower.com;
  
  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}

server {
  listen 80;
  server_name dns.baidpower.com;
  
  location / {
    proxy_pass http://127.0.0.1:8081;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
~~~

# For separate subdomain for mysql
`sudo nano /etc/nginx/sites-available/phpmyadmin`
~~~ini
server {
  listen 80;
  server_name sql.baidpower.com;
  
  root /usr/share/phpmyadmin;
  index index.php index.html index.htm;
  
  location / {
    try_files $uri $uri/ /index.php?$args;
  }
  
  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  }
  
  location ~ /\.ht {
    deny all;
  }
}
~~~
