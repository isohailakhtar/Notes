# RaspAp
Set the WiFi country in  `raspi-config`'s Localisation Options:
`sudo raspi-config`

Invoke RaspAP's Quick Installer:
`curl -sL https://install.raspap.com | bash`

## Initial settings

After completing either of these setup options, the wireless AP network will be configured as follows:

-   IP address: 10.3.141.1
-   Username: admin
-   Password: secret
-   DHCP range: 10.3.141.50 — 10.3.141.254
-   SSID: RaspAP
-   Password: ChangeMe


# Pi-hole
Those who want to get started quickly and conveniently may install Pi-hole using the following command:
`curl -sSL https://install.pi-hole.net | bash`

## Nginx Reverse proxy for DNS
`/etc/nginx/sites-available/{domain}`
Eg.
`/etc/nginx/sites-available/baidpower`
~~~ini
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
    server_name ddns.baidpower.com;

    location / {
        proxy_pass http://127.0.0.1:8081;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
~~~
