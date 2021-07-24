# Guide to Staking on Ethereum 2.0 (Ubuntu/Lighthouse)

![image](https://user-images.githubusercontent.com/60827187/113663325-ae485400-965e-11eb-9591-6492ef4b320e.png)

This is a step-by-step guide to staking on the Ethereum 2.0 mainnet using the Sigma Prime Lighthouse client. It is based on the following technologies:
+ [Ubuntu](https://ubuntu.com/) v20.04 (LTS) x64 server
+ [Go Ethereum](https://geth.ethereum.org/) Node ([code branch](https://github.com/ethereum/go-ethereum))
+ [Sigma Prime](https://sigmaprime.io/)’s Ethereum 2.0 client, Lighthouse ([code branch](https://github.com/sigp/lighthouse))
+ [MetaMask](https://metamask.io/) crypto wallet browser extension

**WARNING: Staking requires at least 32 ETH + gas fees. DO NOT send ETH anywhere without knowing what you are doing. This guide includes instructions to safely deposit your ETH for staking on the Ethereum 2.0 mainnet using official methods. Never send your ETH to anyone.**

## Acknowledgements

This guide is based on information gathered from various on-line resources and wouldn’t exist without them. Thank you, all!

Thanks to the [Ethstaker](http://invite.gg/ethstaker) Discord Core Moderators and Educators, the Eth2 client teams, and the staking community for their help and review.

Special thanks to the Eth2 client teams and the Ethereum Foundation researchers. Their tireless efforts over the past few years have brought us to the cusp of an incredible moment in history — the launch of Ethereum 2.0.

Continued work on this guide and the port to Github was made possible by a grant from the [Ethereum Foundation](https://ethereum.org/en/foundation/) and multiple [Gitcoin grant](https://gitcoin.co/grants/) rounds.

## Disclaimer

This article (the guide) is for informational purposes only and does not constitute professional advice. The author does not guarantee accuracy of the information in this article and the author is not responsible for any damages or losses incurred by following this article. A [full disclaimer](#full-disclaimer) can be found at the bottom of this page — please read before continuing.

## Support

For technical support please reach out to:
+ The [EthStaker](https://ethstaker.cc/) community on [Reddit](https://www.reddit.com/r/ethstaker/) or [Discord](http://invite.gg/ethstaker). Knowledgeable and friendly community passionate about staking on Ethereum 2.0.
+ The Lighthouse client team [Discord](https://discord.gg/gdq27tnKSM).

## Prerequisites

This guide assumes knowledge of Ethereum, ETH, staking, Linux, and MetaMask (or Portis or Fortmatic).

This guide also requires the following before getting started:
+ [Ubuntu server v20.04 (LTS) amd64](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview) or newer, installed and running on a local computer or in the cloud. A locally running computer is encouraged for greater decentralization — if the cloud provider goes down then all nodes hosted with that provider go down.
+ [MetaMask](https://metamask.zendesk.com/hc/en-us/articles/360015489531-Getting-Started-With-MetaMask-Part-1-) crypto wallet browser extension (or Portis or Fortmatic), installed and configured. A computer with a desktop (Mac, Windows, Linux, etc.) and a browser (Brave, Safari, FireFox, etc.) is required.

## Testnet to Mainnet
If moving from a testnet setup to a mainnet setup it is strongly recommended that you start on fresh (newly installed) server instance. This guide has not been tested for migration scenarios and does not guarantee success if you are using an existing instance with previously installed testnet software.

## Requirements
Hardware requirements are a broad topic. In general a relatively modern CPU, 8GB RAM (16GB is better), a SSD of at least 500GB (1TB is better), and a stable internet connection with sufficient download speed and monthly data allowance are likely required for good staking performance.

> NOTE: Check your available disk space. Even you have a large SSD there are cases where Ubuntu is reporting only 200GB free. If this applies to you then take a look at [Appendix D — Expanding the Logical Volume](#appendix-d--expanding-the-logical-volume).

## Overview
The simplified diagram below indicates the scope of this guide. The yellow boxes are the areas this guide mostly covers.

![image](https://user-images.githubusercontent.com/60827187/113665649-aee2e980-9662-11eb-8987-f69049c47bdf.png)

The conceptual flow through the guide is:
+ Generate the staking Validator Keys and Deposit Data
+ Prepare the Ubuntu Server (firewall, security, etc.)
+ Set up an Eth1 Node and sync it with the Eth1 Blockchain
+ Configure the Lighthouse client and sync it with the Eth1 Node
+ Deposit ETH to Activate Validator Keys

Let’s get started!

## Step 1 — Generate Staking Data

In order to participate in staking it is necessary to generate some data files based on the number of validators you’d like to fund and operate.

> NOTE: If you have already generated your deposit data and validator key(s) you can skip this step.

Each validator requires a deposit of 32 ETH. You should have sufficient ETH in your MetaMask wallet to fund each validator. For example, if you plan to run 5 validators you will need to have (32 x 5) = 160 ETH plus some extra to cover the gas fees. The ETH deposit will happen later in the guide after everything else is up and running.

### Download the Deposit Tool (Deposit CLI)

Go [here](https://github.com/ethereum/eth2.0-deposit-cli/releases/) to get the “Latest release” of the deposit command line interface (CLI) app.

![image](https://user-images.githubusercontent.com/60827187/126877674-fe1318fe-7ba6-4f52-b6e7-4898a35ad05b.png)

In the **assets** section download the version matching the platform you are currently on (e.g. Windows, Mac, Linux Desktop, etc.).

### Run the Deposit Tool (Eth2 Deposit CLI)

Decompress the archive. There should be a binary file (executable) inside. The deposit tool generates files for staking as well as a mnemonic key. This key must be handled securely. There are two options to proceed from here.

**Recommended** — Copy the binary file to a USB drive. Connect to a fully air-gapped machine (never previously connected to the internet), **copy** the file over and run from there.

**Not recommended** — Run from the current machine. An internet connection may be an opportunity to leak your mnemonic key. If a fully air-gapped machine isn’t available disconnect the internet on the current machine before proceeding.

When ready, run the file in a terminal window (or CMD in windows) to continue using the commands below. Replace `<NumberOfValidators>` with the number of validators you want to fund. E.g. `--num_validators 2`.

On Linux/Mac:

```
./deposit new-mnemonic --num_validators <NumberOfValidators> --chain mainnet
```

On Windows:

```
deposit.exe new-mnemonic --num_validators <NumberOfValidators> --chain mainnet
```

Once you execute the above steps on your platform of choice you will be asked to create a validator keystore password. Back it up somewhere safe. You will need this later to load the validator keys into the Lighthouse validator wallet.

![image](https://user-images.githubusercontent.com/60827187/113666386-10578800-9664-11eb-9fb8-e4258859f345.png)

A seed phrase (**mnemonic**) will be generated. **Back it up somewhere safe**. This is **CRITICAL**. You will eventually use this to generate your withdrawal keys for your ETH or add additional validators. If you lose this key you will not be able to withdraw your funds.

![image](https://user-images.githubusercontent.com/60827187/113666494-341ace00-9664-11eb-8e38-1e6783654f6a.png)

Once you have confirmed your mnemonic your validator keys will be created.

![image](https://user-images.githubusercontent.com/60827187/113666517-3bda7280-9664-11eb-8c85-8ce5988f391e.png)

The newly created validator keys and deposit data file are created at the specified location. The contents of the folder are shown below.

![image](https://user-images.githubusercontent.com/60827187/113666541-44cb4400-9664-11eb-8c51-0d0939c999c3.png)

Notes about the files:
+ The `deposit_data-[timestamp].json` file contains the public keys for the validators and information about the staking deposit. This file will be used to complete the ETH deposit process later on.
+ The `keystore-m...json` files contain the encrypted validator signing key. There is one keystore-m per validator that you are funding. These will be imported into the Lighthouse validator wallet for use while staking. You will copy these files over to the Ubuntu server (if not already there) later.

### Final Steps
Now that you have the deposit data and the keystore files move on to set up the Ubuntu server.

**DO NOT DEPOSIT any ETH at this moment.**

It is important to complete and verify your staking setup first. If the ETH deposits become active and your staking setup is not ready you will start receiving penalties for non-activity.

## Step 2 — Connect to the Server

Using a SSH client, connect to your Ubuntu server. If you are logged in as `root` then create a user-level account with admin privileges instead, since logging in as the root user is risky.

> NOTE: If you are **not** logged in as `root` then skip this and go to Step 3.

Create a new user. Replace `<yourusername>` with a username of your choice. You will asked to create a strong password and provide some other optional information.

```
# adduser <yourusername>
```

Grant admin rights to the new user by adding it to the `sudo` group. This will allow the user to perform actions with superuser privileges by typing `sudo` before commands.

```
# usermod -aG sudo <yourusername>
```

Optional: If you used [SSH keys](https://jumpcloud.com/blog/what-are-ssh-keys) to connect to your Ubuntu instance via the `root` user you will need to associate the new user with the root user’s SSH key data.

```
# rsync --archive --chown=<yourusername>:<yourusername> ~/.ssh /home/<yourusername>
```

Finally, log out of `root` and log in as `<yourusername>`.

## Step 3 — Update the Server

Make sure the system is up to date with the latest software and security updates.

```
$ sudo apt update && sudo apt upgrade
$ sudo apt dist-upgrade && sudo apt autoremove
$ sudo reboot
```

## Step 4 — Secure the Server

Security is important. This is not a comprehensive security guide, just some basic settings.

### Modify the Default SSH Port

Port 22 is the default SSH port and a common attack vector. Change the SSH port to avoid this.

Choose a port number between 1024–49151 and run the following command to check is not already in use. A blank response indicates not in use, a red text response indicates it is in use: try a different port. E.g. `sudo ss -tulpn | grep ':6673'`

```
$ sudo ss -tulpn | grep ':<YourSSHPortNumber>'
```

If confirmed available, modify the default port by updating SSH config.

```
$ sudo nano /etc/ssh/sshd_config
```

Find or add (if not present) the line `Port 22` in the file. Remove the `#` (if present) and change the value as below.

```
Port <YourSSHPortNumber>
```

Check the screen shot below for reference. Press CTRL+x then ‘y’ then <enter> to save and exit.
  
![image](https://user-images.githubusercontent.com/60827187/113667300-71cc2680-9665-11eb-95f8-0b6eb3856dab.png)

Restart the SSH service to reflect the above changes.

```
$ sudo systemctl restart ssh
```

Log out and log back in via SSH using `<YourSSHPortNumber>` for the port.

### Configure the Firewall

Ubuntu 20.04 servers can use the default UFW firewall to restrict inbound traffic to the server. Before you enable it allow inbound traffic for SSH, Go Ethereum, and Lighthouse.

#### Install UFW

UFW should be installed by default. The following command will ensure it is.

```
$ sudo apt install ufw
```

#### Apply UFW Defaults

Explicitly apply the defaults. Inbound traffic denied, outbound traffic allowed.

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
```

#### Allow SSH

Allow inbound traffic on `<YourSSHPortNumber>` as set above. SSH requires the TCP protocol. E.g. `sudo ufw allow 6673/tcp`

```
$ sudo ufw allow <yourSSHportnumber>/tcp
```

#### Deny SSH Port 22

Deny inbound traffic on port 22/TCP.

> NOTE: Only do this after you SSH in using `<YourSSHPortNumber>`.

```
$ sudo ufw deny 22/tcp
```

#### Allow Go Ethereum

Allow P2P connections with Go Ethereum peers (port 30303). If using an Eth1 node hosted by a [3rd party](https://ethereumnodes.com/) then skip this step.

> NOTE: If you are hosting your Ubuntu instance locally your internet router may need to be [configured](https://www.howtogeek.com/66214/how-to-forward-ports-on-your-router/) to allow incoming traffic on port 30303 as well.

```
$ sudo ufw allow 30303
```

#### Allow Lighthouse

Allows P2P connections with Lighthouse peers for actions on the beacon node (port 9000).

> NOTE: If you are hosting your Ubuntu instance locally your internet router may need to be [configured](https://www.howtogeek.com/66214/how-to-forward-ports-on-your-router/) to allow incoming traffic on port 9000 as well.

```
$ sudo ufw allow 9000
```

#### Enable the Firewall

Enable the firewall and verify the rules have been correctly configured.

```
$ sudo ufw enable
$ sudo ufw status numbered
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/113668128-a8567100-9666-11eb-9e1c-2dc76d129798.png)

## Step 5 — Configure Timekeeping

Ubuntu has time synchronization built in and activated by default using systemd’s timesyncd service. Verify it’s running correctly.

```
$ timedatectl
```

The `NTP service` should be `active`. If not then run:

```
$ sudo timedatectl set-ntp on
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/113668259-d045d480-9666-11eb-89ad-84eaa4eaeea2.png)

You should only be using a single keeping service. If you were using NTPD from a previous installation you can check if it exists and remove it using the following commands.

```
$ ntpq -p
$ sudo apt-get remove ntp
```

## Step 6 — Set up an Ethereum (Eth1) Node

An Ethereum node is required for staking. You can either run a local Eth1 node or use a [third party](https://ethereumnodes.com/) hosted node. This guide will provide instructions for running the Go Ethereum (Eth1) Node.

> NOTE: If you would rather use a third party hosted node - a reasonable option if you don't have the necessary disk space for a local Ethereum (Eth1) Node - then skip this step. Instructions to configure the Lighthouse client with a third party hosted node are included below.

> NOTE: Check your available disk space. An Eth1 node requires roughly 400GB of space. Even you have a large SSD there are cases where Ubuntu is reporting only 200GB free. If this applies to you then take a look at [Appendix D — Expanding the Logical Volume](#appendix-d--expanding-the-logical-volume).

### Install Go Ethereum

Install the Go Ethereum client using PPA’s (Personal Package Archives).

```
$ sudo add-apt-repository -y ppa:ethereum/ethereum
$ sudo apt update
$ sudo apt install geth
```

Go Ethereum will be configured to run as a background service. Create an account for the service to run under. This type of account can’t log into the server.

```
$ sudo useradd --no-create-home --shell /bin/false goeth
```

Create the data directory for the Eth1 chain. This is required for storing the Eth1 node data.

```
$ sudo mkdir -p /var/lib/goethereum
```

Set directory permissions. The goeth account needs permission to modify the data directory.

```
$ sudo chown -R goeth:goeth /var/lib/goethereum
```

Create a systemd service config file to configure the service.

```
$ sudo nano /etc/systemd/system/geth.service
```

Paste the following service configuration into the file.

```
[Unit]
Description=Go Ethereum Client
After=network.target
Wants=network.target

[Service]
User=goeth
Group=goeth
Type=simple
Restart=always
RestartSec=5
ExecStart=geth --http --datadir /var/lib/goethereum --cache 2048 --maxpeers 30

[Install]
WantedBy=default.target
```

Notable [flags](https://geth.ethereum.org/docs/interface/command-line-options):

`--http` Expose an HTTP endpoint (http://localhost:8545) that the Lighthouse beacon chain will connect to.

`--cache` Size of the internal cache in GB. Reduce or increase depending on your available system memory. A setting of `2048` results in roughly 4–5GB of memory usage.

`--maxpeers` Maximum number of peers to connect with. More peers equals more internet data usage. Do not set this too low or your Eth1 node will struggle to stay in sync.
Check the screen shot below for reference. Press CTRL+x then ‘y’ then <enter> to save and exit.
  
![image](https://user-images.githubusercontent.com/60827187/113669589-d2109780-9668-11eb-9426-e9cca5831733.png)

Reload systemd to reflect the changes and start the service. Check status to make sure it’s running correctly.

```
$ sudo systemctl daemon-reload
$ sudo systemctl start geth
$ sudo systemctl status geth
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/113669630-e18fe080-9668-11eb-9233-cf3fa8494e46.png)

It should say active (running) in green text. If not then go back and repeat the steps to fix the problem. Press Q to quit (will not affect the geth service).

Enable the geth service to automatically start on reboot.

```
$ sudo systemctl enable geth
```

The Go Ethereum node will begin to sync. You can follow the progress or check for errors by running the following command. Press CTRL+c to exit (will not affect the geth service).

```
$ sudo journalctl -fu geth.service
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/113669694-f79da100-9668-11eb-906e-693d6b2e697c.png)

### Check Sync Status

To check your Eth1 node sync status use the following command to access the console.

```
geth attach http://127.0.0.1:8545
> eth.syncing
```

If `false` is returned then your sync is complete. If syncing data is returned then you are still syncing. For reference there are roughly 900–1000 million `knownStates`. This [issue](https://github.com/ethereum/go-ethereum/issues/15616) tracks the latest.

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/113669828-2ae03000-9669-11eb-9046-03dcd3c9fa25.png)

Press CTRL+d to exit when done.

### Check Connected Peers

To check your Eth1 node connected peers use the following command to access the console.

```
geth attach http://127.0.0.1:8545
> net.peerCount
```

The `peerCount` will not exceed your setting for `--maxpeers`. If you are having trouble finding peers to sync see the next section.

Press CTRL+d to exit when done.

### Add Bootnodes (Optional)

Sometimes it can take a while to find peers to sync. If so, you can add some bootnodes to help things along. Go [here](https://gist.github.com/rfikki/a2ccdc1a31ff24884106da7b9e6a7453) for the latest list and modify the geth service as follows:

```
$ sudo systemctl stop geth
$ sudo nano /etc/systemd/system/geth.service
```

Modify the ExecStart line and add the `--bootnodes` flag with a few of the latest peers, **comma separated**.

```
ExecStart=geth --http --datadir /var/lib/goethereum --cache 2048 --maxpeers 30 --bootnodes "enode://d0b4a09d072b3f021e233fe55d43dc404a77eeaed32da9860cc72a5523c90d31ef9fab7f3da87967bc52c1118ca3241c0eced50290a87e0a91a271b5fac8d0a6@157.230.142.236:30303,enode://5070366042daaf15752fea340e7ffce3fd8fc576ac846034bd551c3eebac76db122a73fe8418804c5070a5e6d690fae133d9953f85d7aa00375d9a4a06741dbc@116.202.231.71:30303"
```

Save the file and exit. Restart the service and observe.

```
$ sudo systemctl daemon-reload
$ sudo systemctl start geth
$ sudo journalctl -fu geth.service
```

> NOTE: It is necessary to follow a specific series of steps to update Geth. See [Appendix A — Updating Geth](#appendix-a--updating-geth) for further information.

## Step 7 — Download Lighthouse

The Lighthouse client is a single binary which encapsulates the functionality of the beacon chain and validator. This step will download and prepare the Lighthouse binary.
First, go [here](https://github.com/sigp/lighthouse/releases) and identify the latest release. It is at the top of the page. For example:

![image](https://user-images.githubusercontent.com/60827187/115157732-bc737880-a03f-11eb-9811-e388e3b7fa05.png)

In the assets section (expand if necessary) copy the download link to the **lighthouse-v…-x86_64-unknown-linux-gnu.tar.gz** file. Be sure to copy the correct link.

> NOTE: There are two types of binaries — portable and non-portable. The difference is explained [here](https://lighthouse-book.sigmaprime.io/installation-binaries.html). Portable works on a broader set of hardware but comes with a 20% performance cost.

![image](https://user-images.githubusercontent.com/60827187/115157716-a06fd700-a03f-11eb-8a1e-6d80decca3ae.png)

Download the archive using the commands below. Modify the URL in the instructions below to match the download link for the latest version.

```
$ cd ~
$ sudo apt install curl
$ curl -LO https://github.com/sigp/lighthouse/releases/download/v1.3.0/lighthouse-v1.3.0-x86_64-unknown-linux-gnu.tar.gz
```

Extract the binary from the archive and copy to the `/usr/local/bin` directory. The Lighthouse service will run it from there. Modify the URL name as necessary.

```
$ tar xvf lighthouse-v1.3.0-x86_64-unknown-linux-gnu.tar.gz
$ sudo cp lighthouse /usr/local/bin
```

Use the following commands to verify the binary works with your server CPU. If not, go back and download the portable version and redo the steps to here and try again.

```
$ cd /usr/local/bin/
$ ./lighthouse --version # <-- should display version information
```

> NOTE: There has been at least one case where version information is displayed yet subsequent commands have failed. If you get a `Illegal instruction (core dumped)` error while running the `account validator import` command (next step), then you may need to use the **portable** version instead.

Clean up the extracted files.

```
$ cd ~
$ sudo rm lighthouse
$ sudo rm lighthouse-v1.3.0-x86_64-unknown-linux-gnu.tar.gz
```

> NOTE: It is necessary to follow a specific series of steps to update Lighthouse. See [Appendix B — Updating Lighthouse](#appendix-b--updating-lighthouse) for further information.

## Step 8 — Import the Validator Keys

Configure Lighthouse by importing the validator keys and creating the service and service configuration required to run it.

### Copy the Validator Keystore Files

If you generated the validator `keystore-m…json` file(s) on a machine other than your Ubuntu server you will need to copy the file(s) over to your home directory. You can do this using a USB drive (if your server is local), or via [secure FTP (SFTP)](https://www.maketecheasier.com/use-sftp-transfer-files-linux-servers/).

Place the files here: `$HOME/eth2deposit-cli/validator_keys`. Create the directories if necessary.

### Import Keystore Files into the Validator Wallet

Create a directory to store the validator wallet data and give the current user permission to access it. The current user needs access because they will be performing the import. Change `<yourusername>` to the logged in username.

```
$ sudo mkdir -p /var/lib/lighthouse
$ sudo chown -R <yourusername>:<yourusername> /var/lib/lighthouse
```

Run the validator key import process. You will need to provide the directory where the generated `keystore-m` files are located. E.g. `$HOME/eth2deposit-cli/validator_keys`.

```
$ cd /usr/local/bin
$ lighthouse --network mainnet account validator import --directory $HOME/eth2deposit-cli/validator_keys --datadir /var/lib/lighthouse
```

You will be asked to provide the **password** for the validator keys. This is the password you set when you created the keys during Step 1.

You will be asked to provide the password for *each* key, one-by-one. Be sure to correctly provide the password each time because the validator will be running as a service and it needs to persist the password(s) to a file to access the key(s).

> Note that the validator data is saved in the following location created during the keystore import process: `/var/lib/lighthouse/validators`.

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/115158113-76b7af80-a041-11eb-9139-0a2ce77bea23.png)

Restore default permissions to the `lighthouse` directory.

```
$ sudo chown -R root:root /var/lib/lighthouse
```

The import is complete and the wallet is now set up.

> NOTE: It is necessary to follow a specific series of steps to update Geth. See [Appendix C — Adding a Validator](#appendix-c--adding-a-validator) for further information.

## Step 9 — Configure the Beacon Node Service

In this step you will configure and run the Lighthouse beacon node as a service so if the system restarts the process will automatically start back up again.

### Set up the Beacon Node Account and Directory

Create an account for the beacon node to run under. This type of account can’t log into the server.

```
$ sudo useradd --no-create-home --shell /bin/false lighthousebeacon
```

Create the data directory for the Lighthouse beacon node database and set permissions.

```
$ sudo mkdir -p /var/lib/lighthouse/beacon
$ sudo chown -R lighthousebeacon:lighthousebeacon /var/lib/lighthouse/beacon
$ sudo chmod 700 /var/lib/lighthouse/beacon
$ ls -dl /var/lib/lighthouse/beacon
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/115158216-efb70700-a041-11eb-96b1-eee9ecf41ec2.png)

### Create and Configure the Service

Create a systemd service config file to configure the service.

```
$ sudo nano /etc/systemd/system/lighthousebeacon.service
```

Paste the following into the file.

```
[Unit]
Description=Lighthouse Eth2 Client Beacon Node
Wants=network-online.target
After=network-online.target

[Service]
User=lighthousebeacon
Group=lighthousebeacon
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/lighthouse bn --network mainnet --datadir /var/lib/lighthouse --staking --eth1-endpoints http://127.0.0.1:8545

[Install]
WantedBy=multi-user.target
```

Notable [flags](https://lighthouse-book.sigmaprime.io/api-bn.html).

`bn` subcommand instructs the lighthouse binary to run a beacon node.

`--eth1-endpoints` One or more comma-delimited server endpoints for web3 connection. If multiple endpoints are given the endpoints are used as fallback in the given order. Also enables the `--eth1` flag. E.g. `--eth1-endpoints http://127.0.0.1:8545,https://yourinfuranode,https://your3rdpartynode`.

Check the screen shot below for reference. Press CTRL+x then ‘y’ then <enter> to save and exit.
  
![image](https://user-images.githubusercontent.com/60827187/115158295-4fadad80-a042-11eb-88c3-243b7731909c.png)

Reload systemd to reflect the changes and start the service.

```
$ sudo systemctl daemon-reload
```

> Note: If you are running a local Eth1 node (see [Step 6](#step-6--set-up-an-ethereum-eth1-node)) you should wait until it fully syncs before starting the lighthousebeacon service. Check progress here: `sudo journalctl -fu geth.service`

Start the service and check to make sure it’s running correctly.

```
$ sudo systemctl start lighthousebeacon
$ sudo systemctl status lighthousebeacon
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/115158335-7a980180-a042-11eb-910a-724333c17afa.png)

If you did everything right, it should say active (running) in green text. If not then go back and repeat the steps to fix the problem. Press Q to quit (will not affect the lighthousebeacon service).

Enable the service to automatically start on reboot.

```
$ sudo systemctl enable lighthousebeacon
```

If the Eth2 chain is post-genesis the Lighthouse beacon chain will begin to sync. It may take several days to fully sync. You can follow the progress or check for errors by running the `journalctl` command. Press CTRL+c to exit (will not affect the lighthousebeacon service).

```
$ sudo journalctl -fu lighthousebeacon.service
```

Once the Eth2 mainnet starts up the beacon chain will automatically start processing. The output will give an indication of time to fully sync with the Eth1 node.

## Step 10 — Configure the Validator Service

In this step you will configure and run the Lighthouse validator node as a service so if the system restarts the process will automatically start back up again.

### Set up the Validator Node Account and Directory

Create an account for the validator node to run under. This type of account can’t log into the server.

```
$ sudo useradd --no-create-home --shell /bin/false lighthousevalidator
```

In [Step 8](#step-8--import-the-validator-keys) the validator wallet creation process created the following directory: `/var/lib/lighthouse/validators`. Set directory permissions so the `lighthousevalidator` account can modify that directory.

```
$ sudo chown -R lighthousevalidator:lighthousevalidator /var/lib/lighthouse/validators
$ sudo chmod 700 /var/lib/lighthouse/validators
$ ls -dl /var/lib/lighthouse/validators
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/115158505-51c43c00-a043-11eb-965c-f3d352f34a18.png)

### Create and Configure the Service

Create a systemd service file to store the service config.

```
$ sudo nano /etc/systemd/system/lighthousevalidator.service
```

Paste the following into the file.

```
[Unit]
Description=Lighthouse Eth2 Client Validator Node
Wants=network-online.target
After=network-online.target

[Service]
User=lighthousevalidator
Group=lighthousevalidator
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/lighthouse vc --network mainnet --datadir /var/lib/lighthouse --graffiti "<yourgraffiti>"

[Install]
WantedBy=multi-user.target
```

Notable [flags](https://lighthouse-book.sigmaprime.io/validator-management.html).

`vc` subcommand instructs the lighthouse binary to run a validator node.

`--graffiti "<yourgraffiti>"` Replace with your own graffiti string. For security and privacy reasons avoid information that can uniquely identify you. E.g. `--graffiti "Hello Eth2! From Dominator"`.

Check the screen shot below for reference. Press CTRL+x then ‘y’ then <enter> to save and exit.
  
![image](https://user-images.githubusercontent.com/60827187/115158633-f5ade780-a043-11eb-8b89-9cfc119874dd.png)

Reload systemd to reflect the changes and start the service and check to make sure it’s running correctly.

```
$ sudo systemctl daemon-reload
$ sudo systemctl start lighthousevalidator
$ sudo systemctl status lighthousevalidator
```

Check the screen shot below for reference.

![image](https://user-images.githubusercontent.com/60827187/115158648-08c0b780-a044-11eb-8d17-003489206dee.png)

If you did everything right, it should say active (running) in green text. If not then go back and repeat the steps to fix the problem. Press Q to quit (will not affect the lighthousevalidator service).

Enable the service to automatically start on reboot.

```
$ sudo systemctl enable lighthousevalidator
```

You can follow the progress or check for errors by running the `journalctl` command. Press CTRL+c to exit (will not affect the lighthousevalidator service).

```
$ sudo journalctl -fu lighthousevalidator.service
```

A truncated view of the log shows the following status information.

```
INFO Lighthouse started version: Lighthouse/v1.3.0-c6baa0e
INFO Configured for testnet name: mainnet
WARN The mainnet specification is being used. This not recommended (yet).
INFO Starting validator client validator_dir: "/var/lib/lighthouse/validators", beacon_node: http://localhost:5052/
INFO Completed validator discovery new_validators: 0
INFO Enabled validator voting_pubkey: 0xaa...
INFO Enabled validator voting_pubkey: 0x90...
INFO Initialized validators enabled: 2, disabled: 0
INFO Connected to beacon node version: Lighthouse/v1.3.0-c6baa0e/x86_64-linux
INFO Starting node prior to genesis seconds_to_wait: 444946
INFO Waiting for genesis seconds_to_wait: 444946, bn_staking_enabled: true
```

It may take hours or even days to activate the validator account(s) once the beacon chain has started processing.

You can check the status of your validator(s) via [beaconcha.in](https://beaconcha.in/). Simply search for your validator public key(s) or search using your MetaMask (or other) wallet address. It may be a while before the data appears on the site.

## Step 11 — Fund the Validator Keys

Now that your set up is up and running you will need to deposit ETH to fund your validators.

> NOTE: If you have already submitted your staking deposits you can skip this step.

This step involves depositing the required amount of ETH to the Eth2.0 deposit contract. **DO NOT SEND ETH TO THE DEPOSIT CONTRACT**. This is done in a web browser running your MetaMask (or other) wallet via the Eth2.0 Launchpad website.

> NOTE: Wait until your Eth1 node and beacon chain have fully synced before proceeding with the deposit. If you don’t Lighthouse will be inactive while the Eth1 node and/or beacon chain sync and you may be subject to inactivity penalties.

Go here: [https://launchpad.ethereum.org/](https://launchpad.ethereum.org/)

Click through the warning steps and continue through the screens until you get to the **Generate Key Pairs** section. Select the number of validators you are going to run. Choose a value that matches the number of validator files you generated in [Step 1](#step-1--generate-staking-data).

![image](https://user-images.githubusercontent.com/60827187/115158848-175b9e80-a045-11eb-995d-cd018e0ebb78.png)

Scroll down, check the box if you agree, and click Continue.

![image](https://user-images.githubusercontent.com/60827187/115158853-1e82ac80-a045-11eb-84b1-252dffaf6fde.png)

You will be asked to upload the `deposit_data-[timestamp].json` file. You generated this file in [Step 1](#step-1--generate-staking-data). Browse/select or drag the file and click Continue.

![image](https://user-images.githubusercontent.com/60827187/115158866-35290380-a045-11eb-8bd1-78cc3aceb4b9.png)

Connect your wallet. Choose MetaMask (or one of the other supported wallets), log in, select the account where you have your ETH and click Continue.

Your MetaMask balance will be displayed. The site will allow you to continue if you have selected Mainnet and you have sufficient ETH balance.

![image](https://user-images.githubusercontent.com/60827187/115158875-42de8900-a045-11eb-8f56-a78230daec9d.png)

A summary shows the number of validators and total amount of ETH required. Tick the boxes if you agree and click continue.

![image](https://user-images.githubusercontent.com/60827187/115158886-4e31b480-a045-11eb-8e3b-a9241143404d.png)

If you are ready to deposit click on Initiate All Transactions.

![image](https://user-images.githubusercontent.com/60827187/115158892-57bb1c80-a045-11eb-9292-9efa9903346c.png)

This will pop open MetaMask (or other wallet) where you can confirm each transaction.

Once all the transactions have successfully completed you are done!

![image](https://user-images.githubusercontent.com/60827187/115158899-60abee00-a045-11eb-82d4-0d894ed26d4f.png)

Congratulations you have deposited your stake!

### Check the Status of Your Validator Deposits

Newly added validators can take a while (hours to days) to activate. You can check the status of your keys with these steps:

1. Copy your MetaMask (or other) wallet address.
2. Go here: [beaconcha.in/](https://beaconcha.in/)
3. Search for your key(s) using your wallet address.

![image](https://user-images.githubusercontent.com/60827187/115158927-80dbad00-a045-11eb-9bad-81c64ce771df.png)

Diving into a specific validator you see a Status that provides an estimate until activation for each validator.

![image](https://user-images.githubusercontent.com/60827187/115158932-889b5180-a045-11eb-96e5-82c173215ef8.png)

You now have a functioning beacon chain and validator and your mainnet deposit is in. Once your deposit is active and the Ethereum 2.0 mainnet is running you will begin staking and earning rewards.

**Congratulations: You are officially an Ethereum Staker!**

Probably a good time to get a fresh beverage and hydrate.

## Step 12 — Monitoring

Due to a few unresolved security concerns monitoring will be a near-future addition to this guide.

## Final Remarks and Recommended Next Steps

Thanks for the opportunity. Hopefully this guide was helpful for you.

**Next steps:**

+ Triple check all key and password backups.
+ Reboot your machine and make sure the services come back up.
+ Understand how to update the client and server software.
+ Use `htop` to monitor resources on the local machine.
+ Get familiar with [beaconcha.in](https://beaconcha.in/) so you can monitor your validators. They offer an excellent mobile app for monitoring your validators.
+ Join the [Ethstaker](http://invite.gg/ethstaker) and [Lighthouse Discord](https://discord.gg/gdq27tnKSM) for support and important notifications.
+ Help others with their setup on the [Ethstaker](http://invite.gg/ethstaker) discord.
+ Share this guide with others to help with their staking setup.

**Further Reading:**

It is strongly recommended that you evaluate information from as many sources as possible. These are additional resources to help familiarize yourself with staking on Eth2.

The author has not tested or verified these resources. Use at your own risk.

+ Client team official documentation [Prysm](https://docs.prylabs.network/docs/getting-started) | [Lighthouse](https://lighthouse-book.sigmaprime.io/) | [Teku](https://docs.teku.consensys.net/en/latest/) | [Nimbus](https://status-im.github.io/nimbus-eth2/intro.html)
+ [/r/EthStaker Sticky](https://www.reddit.com/r/ethstaker/comments/jjdxvw/welcome_to_rethstaker_the_home_for_ethereum/)
+ [Unofficial docker environment for Ethereum 2.0 clients](https://github.com/eth2-educators/eth2-docker)
+ [Setup an Eth2 Mainnet Validator System on Ubuntu](https://github.com/metanull-operator/eth2-ubuntu)
+ [Guide | How to setup a validator on ETH2 mainnet](https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet)
+ [Guide | Security Best Practices for a ETH2 validator beaconchain node](https://www.coincashew.com/coins/overview-eth/guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node)
+ [Additional Monitoring for ETH2 Staking Nodes](https://moody-salem.medium.com/additional-monitoring-for-eth2-staking-nodes-aea05b2f9a86)
+ [Telegram Service for Ethereum 2.0 Staking](https://9elements.com/blog/ethereum-2-0-2/)

## Appendix A — Updating Geth

If you need to update to the latest version of Geth follow these steps.

```
$ sudo systemctl stop lighthousevalidator
$ sudo systemctl stop lighthousebeacon
$ sudo systemctl stop geth
$ sudo apt update && sudo apt upgrade
$ sudo systemctl start geth
$ sudo systemctl status geth # <-- Check for errors
$ sudo journalctl -fu geth # <-- Monitor
$ sudo systemctl start lighthousebeacon
$ sudo systemctl status lighthousebeacon # <-- Check for errors
$ sudo journalctl -fu lighthousebeacon # <-- Monitor
$ sudo systemctl start lighthousevalidator
$ sudo systemctl status lighthousevalidator # <-- Check for errors
$ sudo journalctl -fu lighthousevalidator # <-- Monitor
```

## Appendix B — Updating Lighthouse

If you need to update to the latest version of Lighthouse follow these steps.

First, go [here](https://github.com/sigp/lighthouse/releases) and identify the latest Linux release. Modify the URL in the instructions below to match the download link for the latest version.

> NOTE: There are two types of binaries — portable and non-portable. The difference is explained [here](https://lighthouse-book.sigmaprime.io/installation-binaries.html). Make sure you choose the correct type.

```
$ cd ~
$ sudo apt install curl
$ curl -LO https://github.com/sigp/lighthouse/releases/download/v1.3.0/lighthouse-v1.3.0-x86_64-unknown-linux-gnu.tar.gz
```

Stop the Lighthouse client services.

```
$ sudo systemctl stop lighthousevalidator
$ sudo systemctl stop lighthousebeacon
```

Extract the binary from the archive and copy to the `/usr/local/bin` directory. Modify the URL name as necessary.

```
$ tar xvf lighthouse-v1.3.0-x86_64-unknown-linux-gnu.tar.gz
$ sudo cp lighthouse /usr/local/bin
```

Restart the services and check for errors.

```
$ sudo systemctl start lighthousebeacon
$ sudo systemctl status lighthousebeacon # <-- Check for errors
$ sudo journalctl -fu lighthousebeacon # <-- Monitor

$ sudo systemctl start lighthousevalidator
$ sudo systemctl status lighthousevalidator # <-- Check for errors
$ sudo journalctl -fu lighthousevalidator # <-- Monitor
```

Clean up the extracted files.

```
$ cd ~
$ sudo rm lighthouse
$ sudo rm lighthouse-v1.3.0-x86_64-unknown-linux-gnu.tar.gz
```

## Appendix C — Adding a Validator

If you want to add one or more validators to your existing validator wallet use the following steps.

> NOTE: This appendix assumes that you have already followed the guide and completed all of the steps. DO NOT use this section to add validators to an empty validator wallet/incomplete setup.

### Check Validator Queue Times

There may be a queue to activate a new validator on the ETH 2.0 mainnet. You can check current queue times by heading over to the pending validators page on [beaconcha.in](https://beaconcha.in/validators#pending). The Activation column shows time to activation. If there are 0 pending validators there is no queue.

![image](https://user-images.githubusercontent.com/60827187/115159344-889c5100-a047-11eb-9a92-f46d712ba3c5.png)

### Generate Deposit Data

You will need to generate deposit data for your new validators. This will be used to add the new validators to your existing wallet.

Go [here](https://github.com/ethereum/eth2.0-deposit-cli/releases/) to get the “Latest release” of the deposit command line interface (CLI) app.

Transfer the binary to a USB stick and copy to an air-gapped machine for safety (recommended), or if not available, copy to a machine that is not connected to the net (not recommended).

When ready, run the file in a terminal window (or CMD in windows) to continue using the commands below.

The existing-mnemonic command is used to re-generate or derive new keys from your existing mnemonic. You will need to supply your mnemonic to do this, hence the security requirements.

On Linux/Mac:

```
./deposit existing-mnemonic --validator_start_index <ValidatorStartIndex> --num_validators <NumberOfValidators> --chain mainnet
```

On Windows:

```
deposit.exe existing-mnemonic --validator_start_index <ValidatorStartIndex> --num_validators <NumberOfValidators> --chain mainnet
```

Replace `<ValidatorStartIndex>` with the start index of the new validator(s) you are adding. 

For example: If you have 3 existing validators (#0, #1, #2) and you want to add two more, you would specify `3` (being the next number in the sequence) as the start index. E.g. `--validator_start_index 3`.

> NOTE: Double check the start index. Be sure you use the correct value.

Replace `<NumberOfValidators>` with the number of validators you want to fund. E.g. from the example above, we are adding two new validators: `--num_validators 2`.

![image](https://user-images.githubusercontent.com/60827187/115159419-de70f900-a047-11eb-9ca6-367964e0a3bf.png)

Enter your mnemonic that you should have stored safely after creating your initial validator(s).

![image](https://user-images.githubusercontent.com/60827187/115159428-e6309d80-a047-11eb-8d2c-9249acafbd60.png)

You will be asked to create a **validator keystore** password. This can be the same as your previous keystore password to keep things simple (recommended) or you can create a different password. **Back it up somewhere safe**. You will need this later to load the new validator keys into the Lighthouse validator wallet.
Once you have confirmed your keystore password your validator keys will be created.

Once you have confirmed your keystore password your validator keys will be created.

![image](https://user-images.githubusercontent.com/60827187/115159450-019ba880-a048-11eb-8caa-b0864900ad22.png)

The newly created validator keys and deposit data file are created at the specified location. The contents of the folder are shown below.

![image](https://user-images.githubusercontent.com/60827187/115159455-082a2000-a048-11eb-965a-6423b2ccb8bd.png)

Notes about the files:

- The newer `deposit_data-[timestamp].json` file contains the public keys for the newly added validators and information about the staking deposit. This file will be used to complete the ETH deposit process later on.

- The newly created `keystore-m...json` files contain the encrypted validator signing key. There is one keystore-m per additional validator that you are funding. These will be imported into the Lighthouse validator wallet for use while staking. You will copy these files over to the Ubuntu server (if not already there) later.

**DO NOT DEPOSIT any ETH at this moment.**

It is important to complete and verify your staking setup first. If the ETH deposits become active and your staking setup is not ready you will start receiving penalties for non-activity. Let’s do that next.

### Copy the Validator Keystore Files

If you generated the validator `keystore-m…json` file(s) on a machine other than your Ubuntu server you will need to **copy** the file(s) over to your home directory. You can do this using a USB drive (if your server is local), or via [secure FTP (SFTP)](https://www.maketecheasier.com/use-sftp-transfer-files-linux-servers/).

Place the files here: `$HOME/eth2deposit-cli/validator_keys`. Create the directories if necessary.

### Import Keystore Files into the Validator Wallet

We are now ready to import the `keystore-m…json` file(s) into our existing validator wallet.

First stop the validator. This is may cause your validator to miss some attestations, but it is unavoidable. The loss of income is generally small.

```
$ sudo systemctl stop lighthousevalidator
```

The current user needs access because they will be performing the import. Change `<yourusername>` to the logged in username.

```
$ sudo chown -R <yourusername>:<yourusername> /var/lib/lighthouse/validators
```

Run the validator key import process. You will need to provide the directory where the generated `keystore-m` files are located. E.g. `$HOME/eth2deposit-cli/validator_keys`.

```
$ cd /usr/local/bin
$ lighthouse --network mainnet account validator import --directory $HOME/eth2deposit-cli/validator_keys --datadir /var/lib/lighthouse
```

Enter the keystore password when prompted for each file.

If you have your original and new `keystore-m` files in the directory you will be asked to enter the password for each file.

If you used a different password for the new files, you will need to use the original password for the existing `keystore-m` files and the new password for the new `keystore-m` files.

If you have your original `keystore-m` files in the directory that have been previously added it will skip the import (since they are already in the validator wallet) and inform you:

```
Skipping import of keystore for existing public key: "/home/ethstaker/eth2deposit-cli/validator_keys/keystore-m_12381_3600_2_0_0-1616386570.json"
```

For the new keystore file(s) it will confirm the addition for each:

```
Keystore found at "/home/ethstaker/eth2deposit-cli/validator_keys/keystore-m_12381_3600_4_0_0-1616386614.json":

- Public key: 0x87ad7e70c1e95c287a0666d723cc4575cb1ae3f972cdede9e5c64270229a0918165ecc72c8a2bf01ae49075a63c06681 
- UUID: f38f9224-a95b-4820-b39e-0f1815a49c2a

If you enter the password it will be stored as plain-text in validator_definitions.yml so that it is not required each time the validator client starts.

Enter the keystore password, or press enter to omit it:

Password is correct.

Successfully imported keystore.
Successfully updated validator_definitions.yml.
```

When the operation is complete it will display a summary. In this case, there were 3 keystore files that had previously been imported (skipped) and 2 new files that were imported.

```
Successfully imported 2 validators (3 skipped).
```

Next we restore the permissions to the validator directory.

```
$ sudo chown -R lighthousevalidator:lighthousevalidator /var/lib/lighthouse/validators
```

And then restart the validator and check for errors.

```
$ sudo systemctl start lighthousevalidator
$ sudo systemctl status lighthousevalidator # <-- Check for errors
$ sudo journalctl -fu lighthousevalidator # <-- Monitor
```

> NOTE: although the service output may specify 0 new validators, the listing should contain the full set.

```
INFO Completed validator discovery new_validators: 0
INFO Enabled validator voting_pubkey: 0x969...97e
INFO Enabled validator voting_pubkey: 0x979...a89
INFO Enabled validator voting_pubkey: 0xac6...363
INFO Enabled validator voting_pubkey: 0x87a...681
INFO Enabled validator voting_pubkey: 0xb46...26a
INFO Initialized validators enabled: 5, disabled: 0
```

Now that the validators have been imported into your validator wallet and ready to perform duties, you will need to go to [Step 11 — Fund the Validator Keys](#step-11--fund-the-validator-keys) above and fund the new validators via the Launchpad.

> NOTE: The new validators will not be fully functional until you complete [Step 11](#step-11--fund-the-validator-keys).

## Appendix D — Expanding the Logical Volume

There are cases where Ubuntu is provisioning only 200GB of a larger SSD causing users to run out of disk space when syncing their Eth1 node. The error message is similar to:

`Fatal: Failed to register the Ethereum service: write /var/lib/goethereum/geth/chaindata/383234.ldb: no space left on device`

To address this issue, assuming you have a SSD that is larger than 200GB, expand the space allocation for the LVM by following these steps:

```
$ sudo lvdisplay # <-- Check your logical volume size
$ sudo lvm 
> lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
> lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
> exit
$ sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
$ df -h # <-- Check results
```

That should resize your disk to the maximum available space.

If you need support on this please check with the [Ethstaker](http://invite.gg/ethstaker) Discord.

## Full Disclaimer

This article (the guide) is for informational purposes only and does not constitute professional advice. The author does not warrant or guarantee the accuracy, integrity, quality, completeness, currency, or validity of any information in this article. All information herein is provided “as is” without warranty of any kind and is subject to change at any time without notice. The author disclaims all express, implied, and statutory warranties of any kind, including warranties as to accuracy, timeliness, completeness, or fitness of the information in this article for any particular purpose. The author is not responsible for any direct, indirect, incidental, consequential or any other damages arising out of or in connection with the use of this article or in reliance on the information available on this article. This includes any personal injury, business interruption, loss of use, lost data, lost profits, or any other pecuniary loss, whether in an action of contract, negligence, or other misuse, even if the author has been informed of the possibility.
