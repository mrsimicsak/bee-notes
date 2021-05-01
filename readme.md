# Setting up a Bee node on a Raspbery Pi 4

## Disclaimer

Follow these instructions at your own risk.

## Intro

This guide is intended to take you from a fresh Ubuntu 20.04 installed on a Raspberry Pi 4 to working bee node.

This guide is written for bee 0.5.3, things are changing rapidly and this guide may not work for other versions.

This guide is for setting up and running a single node. If you want to run and manage more there are better approuchs than the steps outlined here, please look elsewhere for guides. 

## Goals 

A single working Bee node using bee-clef using a local geth instance. Bee, bee-clef, and geth will all be running as services.

## Prequesits

- Rasperry Pi 4 (I used a 4GB RAM version) running Ubuntu 20.04.
- SD Card
- External USB SSD (I used a Samsung T5 Portable 2TB)

## Things to avoid

- Using Hard Drives instead of SSDs. Hard Drives are not fast enough to keep up with even a single nodes load.
- Using an SD card instead of an SSD. SD cards will qucily fail under the load of a bee node as they are not designed for intensive writes.

## Resources

The offical docs are here: https://docs.ethswarm.org/

If you need help ask in the #bee-support channel on the swarm discord. 

goerli explorer: https://goerli.etherscan.io/

cashout troubleshooting guide: https://hackmd.io/95rvXRswSh-xm9Td_Xl2cg?view

## setup external drive

Format dirve as ext 4. exfat does not support permisions.

Edit `/etc/fstab` and add a line for moutning the drive. TODO: better insturctions.

Reboot `sudo reboot`.

Verify drive is mounted: 

run `df -h` and verfiy there is a line that ends with: `/mnt/samsung01`

Create folders for bee and geth:
`cd /mnt/samsung01/`
`mkdir bee`
`sudo chown -R bee.bee bee`
`mkdir geth`
`sudo chown -R geth.geth geth`
`cd ~`

# geth

## Intro

Bee needs to interact with the ethereum blockchain for chasing out checks. geth provides a local interface to the blockchain. Alterantivly a public interface can be used. Please see <TODO> for instructions on using a public interface. If using a public interface setting up geth can be skipped.

## setup geth

`sudo useradd geth`

`sudo mkdir /home/geth`

`sudo chown geth.geth /home/geth`

`sudo -u geth -s`

`cd /home/geth`

`wget https://gethstore.blob.core.windows.net/builds/geth-linux-arm64-1.10.2-97d11b01.tar.gz`

`tar -xvf geth-linux-arm64-1.10.2-97d11b01.tar.gz`

`exit`

`sudo nano /lib/systemd/system/geth.service`

```
[Unit]
Description=Ethereum go client
After=syslog.target network.target
RequiredBy=bee.service

[Service]
User=geth
Group=geth
Environment=HOME=/home/geth
Type=simple
ExecStart=/home/geth/geth --datadir /mnt/samsung01/geth/ --http --goerli --syncmode light
KillMode=process
KillSignal=SIGINT
TimeoutStopSec=90
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

`sudo systemctl enable geth`
`sudo systemctl start geth`

# bee-clef

## Intro

Clef is a utility for mangaing ethereum wallets. The swarm team provides a version preconfigured to work with bee called bee-clef.

## install bee-clef

Download bee-clef: `wget https://github.com/ethersphere/bee-clef/releases/download/v0.4.9/bee-clef_0.4.9_arm64.deb`
And then install it: `sudo dpkg -i bee-clef_0.4.9_arm64.deb`

## start bee-clef

`sudo systemctl enable bee-clef.service`
`sudo systemctl start bee-clef.service`

verify bee-clef is running with: `sudo systemctl status bee-clef.service`

## backup bee-clef

Backup the keys for your node by exporting them using `bee-clef-keys`. This create a key and a password file somewhere and give you a path to the files. These files contain a copy of the ethereum wallet that will be used by bee. Save them somewhere safe and not on this node! Note the key file is encrpyted using the password in the password file .

Once they have been backed up, delete the files from node using `rm bee-clef-key*.json` and `bee-clef-password*.txt`. The key is still stored safely within bee-clef, this just cleans up the exported copies to reduce the places they can be exposed.

# bee

## install bee

Download bee: `wget https://github.com/ethersphere/bee/releases/download/v0.5.3/bee_0.5.3_arm64.deb`

And then install it: `sudo dpkg -i bee_0.5.3_arm64.deb`

## configure bee

There are a couple of options that need to be configured

Open bee's config file with nano: `sudo nano /etc/bee/bee.yml`

if you get "command nano not found" install nano with `sudo apt install nano`

Find and edit the following lines:

```
clef-signer-enable: true
clef-signer-endpoint: /var/lib/bee-clef/clef.ipc
data-dir: /mnt/samsung01/bee
swap-endpoint: http://localhost:8545
verbosity: debug
```

Some of these lines may start with `#`. If they do remove the `#` and following space from the start of the line.

When finished editing use `ctrl + o` to save the file and `ctrl + x` to exit nano.

## start bee

`sudo systemctl enable bee.service`
`sudo systemctl start bee.service`

## fund bee

Find your node's ethereum address with: `sudo journalctl -u bee -g "ethereum address"`

Copy the address. 

In the #faucet-request channel in the swarm discord type `sprinkle <ethereum address>`

# Post install notes

## Managing bee:

start bee: `sudo systemctl start bee.service`
restart bee: `sudo systemctl restart bee.service`
stop bee: `sudo systemctl stop bee.service`
check the status of bee: `sudo systemctl status bee.service`

accessing the logs for bee: `sudo journalctl -u bee -f`

## manging bee-clef

start bee-clef: `sudo systemctl start bee-clef.service`
restart bee-clef: `sudo systemctl restart bee-clef.service`
stop bee-clef: `sudo systemctl stop bee-clef.service`
check the status of bee-clef: `sudo systemctl status bee-clef.service`

## managing geth

start geth: `sudo systemctl start geth.service`
restart geth: `sudo systemctl restart geth.service`
stop geth: `sudo systemctl stop geth.service`
check the status of geth: `sudo systemctl status geth.service`

## check disk space

`df -h`

# Alternatives 

## may work but have not been tried

- using this guide with an x86 install of Ubuntu
- using this guide with raspbian or other raspberry pi operating systems.
- using this guide with debian or dervied operting systems.

# Contributing

If you find something wrong or want something added please open an issue or a pull request.
