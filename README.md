# nightscout-at-oraclevm
HowTo install Nightscout on a free OracleVM

First got to Oracle Cloud (https://www.oracle.com/de/cloud/) and Sign in to Oracle Cloud
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/1.%20Create%20Account.PNG)
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/2.%20Create%20Account.PNG)

Fill in your Country, First- and Lastname and E-Mail Adress and Confirm your Mailadress:
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/3.%20Create%20Account.PNG)

After that you can select your Main Region (should be close to you), enter your Credit-Card (only for verification, should not be charged with any amount), and click on 'Start my free Test'
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/4.%20Create%20Account.PNG)

After that it takes a few minutes:
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/5.%20Create%20Account.PNG)

When you are logged in click on Instances:
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/6.%20Create%20VM.PNG)

Create instance:
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/7.%20Create%20VM.PNG)

Give it a nice name, Change the shape to VM.Standard.A1.Flex, up to 4 ARM Cores and 24 GB RAM are for free forever, after that change the image to Canonical Ubuntu 22.02 Minimal aarch64:
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/8.%20Create%20VM.PNG)

scroll down and save the Private key (Important!) and click "Create"
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/8.1%20Create%20VM.PNG)

After a minute the VM should be running, note the Public IP Adress and click on the Subnet
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/9.%20Create%20VM.PNG)

Click on the Default Security List for this subnet
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/10.%20Open%20Port%20in%20Firewall.png)

Add an Ingress Rule:
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/11.%20Open%20Port%20in%20Firewall.png)

Source IP is 0.0.0.0/0, Destination Port Range is 80,443, and Add Ingrerss Rules.
![alt text](https://github.com/tremor/nightscout-at-oraclevm/blob/main/images/12.%20Open%20Port%20in%20Firewall.png)

Now the VM is ready for Login with SSH. Use your favorite SSH Client to login as User 'Ubuntu' with the saved SSH Keyfile.

First of all update the system:
```bash
$ sudo apt update && sudo apt upgrade && sudo apt clean 
``` 

Then switch to the root user for the Installion and install libssl1.1 and all required pakets:
```bash
$ sudo bash
$ wget http://ports.ubuntu.com/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.16_arm64.deb      
$ dpkg -i libssl1.1_1.1.1f-1ubuntu2.16_arm64.deb    
$ wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
$ apt update
$ apt install mongodb-org nano ufw nginx git python3 gcc nodejs build-essential checkinstall libssl-dev certbot python3-certbot-nginx
```

After that enable and start the mongodb:
```bash
$ systemctl enable mongod 
$ systemctl start mongod
```

Verify that the monodb is running:
```bash
$ systemctl status mongod 
$ mongosh --eval 'db.runCommand({ connectionStatus: 1 })' 
```
The first should return something like this:
```bash
??? mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-10-27 12:25:14 UTC; 1h 5min ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 691 (mongod)
     Memory: 255.8M
        CPU: 11.058s
     CGroup: /system.slice/mongod.service
             ??????691 /usr/bin/mongod --config /etc/mongod.conf

nightscout-vm systemd[1]: Started MongoDB Database Server.
```
the second should end with:
```bash
{
  authInfo: { authenticatedUsers: [], authenticatedUserRoles: [] },
  ok: 1
}
```

after that create a Database and User, use your own credentials and note them:
```bash
$ ??mongosh
>??use Nightscout
>??db.createUser({user:??"<USERNAME>", pwd:??"<PASSWORT>", roles:["readWrite"]})
>??quit()
```

Now you can create also a system user (again: use a strong password!) how runs the nightscout Webserver and install nightscout:
```bash
$ adduser nightscout
$ usermod -aG sudo nightscout
$ su nightscout
$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

#Logout and relogin
$ source /etc/profile
$ nvm ls-remote
$ nvm install 14.18.1
$ nvm list
$ nvm use 14.18.1
sudo apt install npm
cd
$ git clone https://github.com/nightscout/cgm-remote-monitor.git
$ cd cgm-remote-monitor
$ npm install
```

Now create a startscript in the Homedirectory of the nightscout User with "nano ~/start.sh"
```
#!/bin/bash

# environment variables
export DISPLAY_UNITS="mmol/l"
export MONGO_CONNECTION="mongodb://<USERNAME>:<PASSWORT>@localhost:27017/Nightscout"
export BASE_URL="127.0.0.1:1337"
export API_SECRET="<API-SECRET>"
export PUMP_FIELDS="reservoir battery status"
export DEVICESTATUS_ADVANCED=true
export ENABLE="careportal loop iob cob openaps pump bwg rawbg basal cors direction timeago devicestatus ar2 profile boluscalc food sage iage cage alexa basalprofile bgi directions bage upbat googlehome errorcodes reservoir battery openapsbasal"
export TIME_FORMAT=24
export INSECURE_USE_HTTP=true
export LANGUAGE=de
export EDIT_MODE=on
export PUMP_ENABLE_ALERTS=true
export PUMP_FIELDS="reservoir battery clock status"
export PUMP_RETRO_FIELDS="reservoir battery clock"
export PUMP_WARN_CLOCK=30
export PUMP_URGENT_CLOCK=60
export PUMP_WARN_RES=50
export PUMP_URGENT_RES=10
export PUMP_WARN_BATT_P=30
export PUMP_URGENT_BATT_P=20
export PUMP_WARN_BATT_V=1.35
export PUMP_URGENT_BATT_V=1.30
export OPENAPS_ENABLE_ALERTS=false
export OPENAPS_WARN=30
export OPENAPS_URGENT=60
export OPENAPS_FIELDS="status-symbol status-label iob meal-assist rssi freq"
export OPENAPS_RETRO_FIELDS="status-symbol status-label iob meal-assist rssi"
export LOOP_ENABLE_ALERTS=false
export LOOP_WARN=30
export LOOP_URGENT=60
export SHOW_PLUGINS=careportal
export SHOW_FORECAST="ar2 openaps"

# start server
/home/nightscout/.nvm/versions/node/v14.18.1/bin/node server.js
```

Make the script executable and create a service
```bash
$ chmod 775 start.sh
$ sudo nano /etc/systemd/system/nightscout.service
```
The Content sould look like this:
```
[Unit]
Description=Nightscout Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/nightscout/cgm-remote-monitor
ExecStart=/home/nightscout/start.sh

[Install]
WantedBy=multi-user.target
```

Then Reload the systemctl and enable and start the Nightscout Website at Port 1337 (not public to the world and not encrypted!) 
```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable nightscout.service
$ sudo systemctl start nightscout.service
$ sudo systemctl status nightscout.service
```

The last command should return something like this:
```
??? nightscout.service - Nightscout Service
     Loaded: loaded (/etc/systemd/system/nightscout.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-10-24 12:53:21 UTC; 3 days ago
   Main PID: 9524 (start.sh)
      Tasks: 12 (limit: 1090)
     Memory: 81.4M
        CPU: 4min 29.768s
     CGroup: /system.slice/nightscout.service
             ??????9524 /bin/bash /home/nightscout/cgm-remote-monitor/start.sh
             ??????9525 /home/nightscout/.nvm/versions/node/v14.18.1/bin/node server.js
```
Open Firewall for ssh and nginx and check if it running:
```bash
$ ufw allow 'Nginx Full'
$ ufw allow OpenSSH
$ ufw enable
$ ufw status
```
The last command should return something like that:
```
Status: active

To                         Action      From
--                         ------      ----
Nginx Full                 ALLOW       Anywhere
OpenSSH                    ALLOW       Anywhere
Nginx Full (v6)            ALLOW       Anywhere (v6)
OpenSSH (v6)               ALLOW       Anywhere (v6)
```

Now we can start the nginx as reverse proxy and create a new /etc/nginx/sites-available/nightscout.conf from scratch:
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name <DOMAINNAME_OF_WEBSERVER>;
    location / {
            try_files $uri $uri/ =404;
    }
}
```

```bash
$ sudo unlink /etc/nginx/sites-enabled/default
$ sudo ln -s /etc/nginx/sites-available/nightscout.conf /etc/nginx/sites-enabled/
$ sudo service nginx restart
```

Get a TLS Cert from the Certbot:
```bash
$ sudo certbot --nginx -d domain.de -d <DOMAINNAME_OF_WEBSERVER>
```
Enter your E-Mailadress, Accept the Conditions and redirect always to https (Option 2), after that the nginx should be listing on 443 and the /etc/init.d/sites-available/nightscout.conf should look like this:
```
server {
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/<DOMAINNAME_OF_WEBSERVER>/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/<DOMAINNAME_OF_WEBSERVER>t/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        ssl_stapling on;
        ssl_stapling_verify on;
        add_header Strict-Transport-Security max-age=15768000;
        location ~ /.well-known {
                allow all;
        }
        location / {
                proxy_pass http://localhost:1337/;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header X-Forwarded-Proto "https";
                #proxy_set_header Host $host;
                #proxy_cache_bypass $http_upgrade;
        }
}
server {
    if ($host = <DOMAINNAME_OF_WEBSERVER>) {
        return 301 https://$host$request_uri;
    } # managed by Certbot
    listen 80 ;
    listen [::]:80 ;
    server_name <DOMAINNAME_OF_WEBSERVER>;
    return 404; # managed by Certbot
}
```
At the end you should test if the Let's Encrypt Certs are automatically renewed:
```bash
$ sudo systemctl status certbot.timer
$ sudo certbot renew --dry-run
```
