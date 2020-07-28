# Guide: install Sphinx-relay on myNode / Raspiblitz.

This guide is focused on installing Sphinx-relay on top of myNode. Information about myNode can be found at: https://mynodebtc.com/.

### Preparations

* Be able to connect to your node through SSH.
* Make sure you are running LND version `0.10.0` or higher. This can be seen at http://mynode.local/lnd at the right top or by typing in the following console command:

```sh
$ lncli getinfo
> "version": "0.10.0-beta commit=v0.10.0-beta"
```
* Open port `3001/TCP` on your router. How to do this is not included in this guide. https://www.yougetsignal.com/tools/open-ports/ is one of the many websites that can be used to check if a port is opened on your network.
---
## Let's Start

### Install dependencies

sqlite3: `$ sudo apt-get install sqlite3`

python2: `$ sudo apt-get install python2`

### Open port 3001 on myNode

Open up a console window with SSH. And log in as root
```sh
$ sudo su
```
Open up port 3001 on your machine and make sure it has been added to the list.
```sh 
# ufw allow 3001 comment 'allow Sphinx-Chat'
# ufw status

> Status: active
>
> To                         Action      From
> --                         ------      ----
> 3001 (v6)                  ALLOW       Anywhere (v6)              # Sphinx-Chat
```

### Download

login as user bitcoin.
```sh
$ sudo su bitcoin
$ cd
```
Clone the repository from Github and install the package.
```sh 
 git clone https://github.com/stakwork/sphinx-relay
$ cd sphinx-relay
$ npm install
```

### Configure
Edit the "production" section of config/app.json.
```sh 
$ cd config
$ nano app.json
```
Change the following 4 lines:

#### myNode
``` 
"macaroon_location": "/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon",
"tls_location": "/mnt/hdd/mynode/lnd/tls.cert",
"lnd_log_location": "/home/bitcoin/.lnd/logs/bitcoin/mainnet/lnd.log",
```

#### Raspiblitz
``` 
"macaroon_location": "/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon",
"tls_location": "/mnt/hdd/lnd/tls.cert",
"lnd_log_location": "/home/bitcoin/.lnd/logs/bitcoin/mainnet/lnd.log",
```

Save and exit:
`Ctrl + X`

`Y`

`Enter`

Edit the "production" section of config/config.json
```sh 
$ nano config.json
```
Change to following line to:
``` 
"storage": "/home/bitcoin/sphinx.db"
```
Save and exit:
`Ctrl + X`

`Y`

`Enter`

To connect to your app:
(replace x.x.x.x with your IP - NOTE: This is your external IP)
```sh 
$ cd
$ cd sphinx-relay/config/
$ export NODE_IP=x.x.x.x:3001
```
For extra security:
```sh
$ export USE_PASSWORD=true
```
### Activate keysend


We need LND to run with keysend activated. First we check if it is already activated on your node. 

#### myNode:
Go to http://mynode.local/lnd/config and check if the line `accept-keysend=1` (or `accept-keysend=True`) is included somewhere in the text.

If `accept-keysend=True` or `accept-keysend=1` is already included you can continue without changing anything. If `accept-keysend=True` is not included, add it to a new line and click the `Save` button. This will restart your device. (Restarting could take up to several minutes but also hours, so be patient.)

#### Raspiblitz:
Go to raspiblitz menu, or:

```sh
$ raspiblitz
```

Find item menu "Services" and activate Keysend

### Run
Now it's time to run the software.

```sh 
$ cd sphinx-relay/config/
$ npm run prod
```
When Relay starts up, it will print a QR in the terminal. You can scan this in your app (Android or iOS) to connect!

### To make relay run continuously (also after a restart).
Before you start this part. Make sure your app is connected and you are able to send & receive messages.

Login as admin.
```sh 
$ sudo su admin
```
Create a file named sphinx-relay.service
```sh 
$ sudo nano /etc/systemd/system/sphinx/sphinx-relay.service
```
Copy and paste the following text to add it to the file:
```sh 
[Unit]
Description=Sphinx Relay Service
After=network.target

[Service]
Type=simple
User=bitcoin
WorkingDirectory=/home/bitcoin/sphinx-relay/config/
ExecStart=npm run prod
Restart=always
RestartSec=5
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=sphinx-relay

[Install]
WantedBy=multi-user.target
```
Save and exit:
`Ctrl + X`

`Y`

`Enter`

Let's run!
```sh 
$ sudo systemctl enable sphinx-relay.service
$ sudo systemctl start sphinx-relay.service
```
Check if relay succesfully started.
```sh 
$ sudo systemctl status sphinx-relay.service
```
### To stop the program
```sh 
$ sudo systemctl stop sphinx-relay.service
```

# To update Sphinx-Relay
(This probably is not the most efficient way to update. But it works so we got that goin which is nice. Feel free to optimize the process and contribute. :) )

Login as Admin and stop the program.
```sh 
$ sudo systemctl stop sphinx-relay.service
```
login as user bitcoin.
```sh
$ sudo su bitcoin
$ cd
```
## Remove the old version
```sh
$ rm -rf sphinx-relay
```
## Download the new version
Clone the repository from Github and install the package.
```sh 
$ git clone https://github.com/stakwork/sphinx-relay
$ cd sphinx-relay
$ npm install
```
### Configure
Edit the "production" section of config/app.json.
```sh 
$ cd
$ cd sphinx-relay/config/
$ nano app.json
```
Change the following 4 lines:

#### myNode
``` 
"macaroon_location": "/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon",
"tls_location": "/mnt/hdd/mynode/lnd/tls.cert",
"lnd_log_location": "/home/bitcoin/.lnd/logs/bitcoin/mainnet/lnd.log",
"lncli_location": "/home/bitcoin/go/bin",
```

#### Raspiblitz
``` 
"macaroon_location": "/home/bitcoin/.lnd/data/chain/bitcoin/mainnet/admin.macaroon",
"tls_location": "/mnt/hdd/lnd/tls.cert",
"lnd_log_location": "/home/bitcoin/.lnd/logs/bitcoin/mainnet/lnd.log",
"lncli_location": "/home/bitcoin/go/bin",
```

Save and exit:
`Ctrl + X`

`Y`

`Enter`

Edit the "production" section of config/config.json
```sh 
$ nano config.json
```
Change to following line to:
``` 
"storage": "/home/bitcoin/sphinx.db"
```
Save and exit:
`Ctrl + X`

`Y`

`Enter`

To connect to your app:
(replace x.x.x.x with your IP - NOTE: This is your external IP)
```sh 
$ cd
$ cd sphinx-relay/config/
$ export NODE_IP=x.x.x.x:3001
```
For extra security:
```sh
$ export USE_PASSWORD=true
```
### Turn on the service.
Login as admin.
```sh 
$ su admin
```
Or
```sh 
$ exit
```
Turn the service on and check the status.
```sh 
$ sudo systemctl enable sphinx-relay.service
$ sudo systemctl start sphinx-relay.service
```

### tail logs 

`journalctl -u sphinx-relay -f`
