This document describes creating tunnel over internet. To tunnel [autossh](https://linux.die.net/man/1/autossh) is used. autossh ensures tunnel is restarted should some network or other error occurs.

We will create a systemd service that will start on each system boot. It is assumed you have already installed autossh.

## Create a autossh wrapper script
First create file `/etc/scritpts/tunnel` with the following content:

```sh
#!/bin/sh

#set -x

IDEN_FILE=$HOME/.ssh/id_rsa
MPORT=42222

autossh -2 -NT -M $MPORT -i "$IDEN_FILE" -o GatewayPorts=true \
        -L 0.0.0.0:2222:localhost:22 \
        -R 0.0.0.0:2222:locahost:22 \
        -R 0.0.0.0:8080:localhost:80 \
        -i $IDEN_FILE/.ssh/id_rsa \
        -p 22222 \
        remote_server_ip
```
Here we are connecting to machine `remote_server_ip`. Port 22 from remote server is forwarded to port 2222 on local machine. Ports 22 and 80 from local machine are forwarded to ports 2222 and 8080 on remote machine respectively.

## Create Systemd Service
Create `/etc/systemd/system/tunnel.service` with following content:

```ini
[Unit]
Description=Autossh service
After=network-online.target

[Service]
User=ncd
Type=exec
ExecStart=/etc/scripts/tunnel
Restart=always

[Install]
WantedBy=multi-user.target
```
To load the new service file execute the command: `systemctl daemon-reload`

To start the tunnel service execute the command: `systemctl start tunnel.service`

To enable the tunnel service execute the command: `systemctl enable tunnel.service`

Now tunnel will be created on each boot.

