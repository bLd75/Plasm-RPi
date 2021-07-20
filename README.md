Dusty v1.9.0 update
-------------------

The network has changed from Dusty3 to Dusty4 including substrate 3.0, enhanced Ethereum compatibility support, and stable wasm smart contracts.

* Portal apps.plasmnet.io is officially depreciated, Dusty should now be used from https://polkadot.js.org/apps.
* Session keys scheme has changed.
* Dusty4 network was launched from scratch, previous blocks from Dusty3 are no longer used, old chain data can be purged.
* New Dusty public endpoints : Substrate = wss://rpc.dusty.plasmnet.io - EVM = https://rpc.dusty.plasmnet.io:8545

Please see the Upgrade section


Introduction
------------

This repo intends to deliver the binary of [Plasm Network](https://www.plasmnet.io/) for Raspberry Pi.

It's entirely build from the sources of [Plasm repository](https://github.com/PlasmNetwork/Plasm) on a Raspberry Pi 4 model B with Ubuntu Server 20.04.01 LTS 64 bit.

As it is compiled of aarch64 architecture, it will most probably not work on previous models of Raspberry that have a 32 bit architecture.


Minimum Requirements
----------------------------
* Raspberry Pi 4 model B with at least 2 Gb RAM (8Gb model is recommended)
* Micro SD card of 16 Gb minimum
* An external USB HDD with 100 Gb free space
* Ethernet cable for network (WiFi is not recommended)


Raspberry and node setup instructions
-------------------------------------

Download and install the [Raspberry Pi Imager](https://www.raspberrypi.org/software/).

Insert the MicroSD card and launch Raspberry Pi Imager.
* Choose OS > Ubuntu > Ubuntu Server 20.04.01 LTS (RPi 3/4) - 64 bit server OS
* Choose SD Card then Write

Start the RPi with the SD Card, plugged to an ethernet cable and let it initialize for a few minutes.

Connect to the RPi directly or via SSH : user = ubuntu - password = ubuntu

Create a swap file (4 Gb in the example below for a 2 Gb RAM model, you can adapt to your RAM amount)

    sudo fallocate -l 4g /file.swap
    sudo chmod 600 /file.swap
    sudo mkswap /file.swap
    sudo swapon /file.swap

In case the USB drive is not in ext4 file system, it is recommended to format it in ext4 with fdisk. You can use this [guide](https://kwilson.io/blog/format-a-linux-disk-as-ext4-from-the-command-line/) to do so.

Identify the device and UUID of USB drive (generally sda1 or sdb1)

    lsblk
    
Create a mounting point for you usb drive (with the name you want) and mount it

    sudo mkdir /media/mydisk
    sudo mount /dev/sda1 /media/mydisk
    
Set swap file and mounting disk in fstab (in case of reboot)

    sudo nano /etc/fstab
    
Add the following lines at the end then save (Ctrl+O > Yes)

    /file.swap none swap sw 0 0
    UUID=<UUID of USB disk>      /media/mydisk        ext4        defaults           0    1

Get and extract the binary file

    wget https://github.com/bLd75/Plasm-RPi/raw/main/plasm-1.8.0-ubuntu-aarch64.tar.gz
    tar -xf  plasm-node-ubuntu-1.8.0-aarch64.tar.gz

Move the binary file to a desired folder (we will use /home/ubuntu as reference here)

    mv ./plasm /home/ubuntu

Create a directory for the chain storage

    sudo mkdir /media/mydisk/plasm/
    
Create a screen session

    screen

Run it and enjoy blocks syncing :)

    /home/ubuntu/plasm --validator --name <Your Validator Name> --rpc-cors all --in-peers 50 --out-peers 50 --base-path /media/mydisk/plasm/

Before you close or anything else you want to do, make sure to detach the screen session to let it run in background

    Ctrl+A
    D

Whenever you want to monitor the node, just resume the screen session

    screen -r


Enable the service
------------------

In order to make your node run as a service and is always running after an eventual reboot, you can setup a systemd service.

    sudo touch /etc/systemd/system/plasm.service
    sudo nano /etc/systemd/system/plasm.service

Add the following lines in the file then save (Ctrl+O > Yes).
Don't forget to adapt the name and base path to your own.

    [Unit]
    Description=Plasm Validator
    
    [Service]
    ExecStart=/home/ubuntu/plasm \
      --validator \
      --name <Your Validator Name> \
      --rpc-cors all \
      --in-peers 50 \
      --out-peers 50 \
      --base-path /media/mydisk/plasm/ \
    
    Restart=always
    RestartSec=60
    
    [Install]
    WantedBy=multi-user.target

Start the service manually
    
    sudo systemctl start plasm.service
    
Check that the service is working fine

    systemctl status plasm.service
    
Enable it to autostart on bootup
    
    sudo systemctl enable plasm.service

To tail the real time log, you can use this command (100 is the number of lines to show)

    journalctl -f -u plasm -n100

Enjoy your fully operational tiniest node :)


Alternative method
------------------

Alternatively to a systemd service, you can just set a cron job to start the Plasm node at reboot.

    crontab -e
    
At the end of the file, add the following lines (adapt the name and base path to your own) then save

    @reboot screen -dS plasm -m /home/ubuntu/plasm --validator --name <Your Validator Name> --rpc-cors all --in-peers 50 --out-peers 50 --base-path /media/mydisk/plasm/
    
Test it with a reboot

    sudo reboot
    
Check the node status

    screen -r plasm

Upgrade
-------

When a new version of Plasm node is released, you just have to download it, replace the binary file and restart the service

    wget https://github.com/bLd75/Plasm-RPi/raw/main/plasm-1.9.0-ubuntu-aarch64.tar.gz
    tar -xf plasm-1.9.0-ubuntu-aarch64.tar.gz
    mv ./plasm /home/ubuntu
    sudo systemctl restart plasm.service

In case of changes to the network version (as for Dusty v1.9.0 -> network version Dusty4), database location for chain data can change.
In this case, you can use the following command to free space. Make sure to purge with the previous version BEFORE you upgrade.

    sudo systemctl restart plasm.service
    /home/ubuntu/plasm purge-chain --base-path /media/mydisk/plasm/

In case session keys scheme changes (as for Dusty v1.9.0), it may be necessary to get a new session keys.

    curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933

Then, don't forget to send the extrinsics to set the new session keys to your account.

Next steps
----------

Now that your node is running, you have to wait for it to be fully synchronized.

You can follow it on [Telemetry](https://telemetry.polkadot.io/#list/Dusty)

Now you will need to generate your session keys.

Then, just follow [Maarten's guide](https://fiex.medium.com/plasm-node-on-azure-32ec5a204b45) from the step "Get your session key".

You can also watch at [Kenzo's video](https://youtu.be/wZgIqAufyCE) starting from 6:00

Other ressource : [Plasm Documentation](https://docs.astar.network/build/validator-guide)
