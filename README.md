Remix made thanks to [Awning](https://github.com/giovantenne/awning)

# ⚠️ This README is not yet complete and the repo still needs a few touchups.
The repo is set up to work with Traefik for generating clearnet certs, however its configuration is not here as it is not assumed you have a public IP

for LND wallet creation, for the time being, you'll want to comment out lines 16&17 (shown below), then docker exec -it btcstack_lnd and create wallet/password. Make sure you

LNbits is currently set to use sqlite

```
# comment lines 16,17 of lnd.conf, set up password which you'll put into config/lnd/password.txt, then uncomment these lines
 # wallet-unlock-password-file=/data/.lnd/password.txt
 # wallet-unlock-allow-create=true
``````

# A dockerized Bitcoin + LND + RTL node
## Options for BTCPay Server, Static Channel Backup, nginx, traefik

Something like [Umbrel](https://umbrel.com) but lighter and portable.
Something like [RaspiBolt](https://raspibolt.org/) but easier and automated. Bitcoin/Lightning-Network oriented with no frills.

This is a plain/vanilla Docker setup.

## Disclaimer
This open-source project is provided 'as-is' without any warranty of any kind, either expressed or implied. The developers are not liable for any damages or losses arising out of the use of this software.

Please read the [full disclaimer](DISCLAIMER.md) before using this project.

# Prerequisites
- git
- docker
- docker-compose (or [`docker compose plugin`](https://docs.docker.com/compose/install/linux/))

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```


# Run your BTC / LN / RTL node:

Follow the steps below:

Please follow [this guide](https://docs.docker.com/engine/install/linux-postinstall/) if you don't want to preface the `docker` and the `docker-compose` commands with `sudo`.
In this guide  `sudo` will be always omitted.
All the commands need to be run from the project **root** directory.

Here is the steps:

1. [Clone this repository](#1)
2. [Edit and customize the `.env` file](#3)
3. [Start the Docker containers](#4)
4. [Create or restore a LND wallet](#6)

> ![TODO] 
>
> Lnbits
>
> BTCPay

You can also [add your BTCPay Server](#7) eventually.

<a name="1"></a>
# Before you begin

Clone or download this repository and enter the project directory.
```sh
$ git clone https://github.com/PatMulligan/btcstack-docker.git
$ cd btcstack-docker
```

<a name="1"></a>
## Edit and understand the .env file

> [!NOTE] Modify .env vars
> The `.env` file contains some setup parameters that you can/need to customize. Just make a copy of the sample file and edit it.
```sh
$ cp .env.sample .env
```


> [!TODO] Add any other parameters related to traefik, mempool, etc
> 
> | Parameter | Description |
> | --- | --- |
> | `UID` | The uid (user id) of your current user. Use this command to retrieve it: `id -u`. |
> | `GID` | The gid (group id) of your current user. Use this command to retrieve it: `id -g`. |
> | `NODE_ALIAS` | The alias your LND node will use |
> | `BITCOIN_ARCH` | Here you need to choose your computer CPU architecture. Write `aarch64` for ARM (Raspberry Pi, etc) or `x86_64` for Intel or AMD. |
> | `LND_ARCH` | Write `arm64` for ARM (Raspberry Pi, etc) or `amd64` for Intel or AMD. |
> | `RTL_PASSWORD` | Choose the password for accessing the *"Ride The Lightning"* web interface. You can change it any time but don't forget to restart the RTL container afterwards with `docker-compose restart rtl`. |
> | `RTL_VERSION`   | RTL [version](https://github.com/Ride-The-Lightning/RTL/releases) |
> |`LND_PASSWORD` | Choose the password to automatically protected and unlock the LND wallet (write `moneyprintergobrrr` if you are migrating from **Umbrel**). You will need to use this password again [here](#6). Changing this after the first setup will have no effect. |
> | `BITCOIN_CORE_VERSION` | Bitcoin Core [version](https://github.com/bitcoin/bitcoin/releases) |
> | `LND_VERSION` | LND [version](https://github.com/lightningnetwork/lnd/releases) |
> | `ELECTRS_VERSION` | Electrs [version](https://github.com/romanz/electrs/releases) |

<a name="3"></a>
# How to begin
The `docker-compose.yml` file contains the docker services that your node will run.

Make a copy of the sample file
```sh
$ cp docker-compose-sample.yml docker-compose.yml
```


Run the following command:
```sh
$ docker-compose up -d --build
```
This will spin-up the following services/containers in background:
- [Bitcoin Core](https://github.com/bitcoin/bitcoin)
- [Electrs](https://github.com/bitcoin/bitcoin)
- [LNbits](https://github.com/lnbits/lnbits)
- [LND](https://github.com/lightningnetwork/lnd)
- [RTL](https://github.com/Ride-The-Lightning/RTL) (Ride The Lightning)
- [TOR](https://www.torproject.org/)

The first time it will take some time to build all the images from scratch (especially compiling the Electrs binary can take up to one hour).

After all the images are built, “bitcoind” should start, begin to sync and validate the Bitcoin blockchain. If you already downloaded the blockchain somewhere else, you can just copy the data to the `./data/bitcoin` directory before the `docker-compose up -d --build` command.

Check the status of the bitcoin daemon that was started with the following command. Exit with Ctrl-C

```sh
$ docker logs -f btcstack_bitcoin
```

Those services open the following TCP ports on your host:

| Port | Service | SSL |  Description |
| --- | --- | --- | --- |
| `50002` | Electrs | :white_check_mark: | Electrs  |
| `8080` | LND | :white_check_mark: | Rest API for LND  |
| `8081` | RTL | :white_check_mark: | RTL web interface|
| `8082` | RTL | :white_large_square: | RTL web interface  |
| `8083` | BTCPay Server (optional)| :white_check_mark: | BTCPay server web interface|
| `8084` | BTCPay Server (optional)| :white_large_square: | BTCPay server web interface |

# Finish the setup

Once you first start the containers there are still a couple of steps to complete:

<a name="4"></a>
### Create or restore the LND wallet

If you are migrating from **Umbrel** or from an existing LND node just copy your data to the `./data/lnd` directory before the `docker-compose up -d --build` command and skip the rest of this step, otherwise run this command:

```sh
$ docker exec -it btcstack_lnd lncli create
```

Enter your password as wallet password (it must be exactly the same you stored in `.env` as [LND_PASSWORD](#3)).

To create a a new wallet, select `n` when asked if you have an existing cipher seed. Just press enter if asked about an additional seed passphrase, unless you know what you’re doing. A new cipher seed consisting of 24 words is created.

These 24 words is all that you need to restore the Bitcoin on-chain wallet. The current state of your channels, however, cannot be recreated from this seed.

🚨 This information must be kept secret at all times.

**Write these 24 words down manually on a piece of paper and store it in a safe place.**

# Use Electrs (via TOR)

Run the following command from the project **root** directory to retrieve the Electrs TOR address to use in your wallet:
```sh
$ echo `cat ./data/tor/hidden_service_electrs/hostname`:50001
```

# Accessing RTL web interface (local)

Ride The Lightining is accessible on both `http` and `https` with a self signed SSL certificate (so expect a warning message from your browser) using the [password](#3) choosen on the `.env` file.

If you are running on this machine you can access the web interface through these URLs:
- [https://localhost:8081](https://localhost:8081)
- [http://localhost:8082](http://localhost:8082)

Replace `localhost` with the IP of your node if you are running on a different PC.

# Connect Zeus to your node (via TOR)
- Download the Zeus app for your mobile phone.
- Open Zeus and tap on “GET STARTED”
- Tap on “Connect a node” and then tap on the “+” at the top right to add your node
- Enter a Nickname for your node (e.g., LndNinja)
- Click on “SCAN LNDCONNECT CONFIG” and, if prompted, allow Zeus to use the camera
- Scan the QR code generated with the following command. It will be a big QR code, so maximize your terminal window and use CTRL+- to shrink the code further to fit the screen
```sh
$ URI=`cat ./data/tor/hidden_service_lnd_rest/hostname` && docker exec btcstack_lnd lndconnect --host $URI --port 8080
```

- Click on “SAVE NODE CONFIG”. Zeus is now connecting to your node, and it might take a while the first time.

# Useful comands
| Command | Description |
| --- | --- |
| `docker ps` |  Lists the containers that are running on your host |
| `docker logs -f btcstack_bitcoin` | Stream the logs for the *bitcoin/lnd/electrs* container |
| `docker exec -it btcstack_lnd bash` |  Connect to the *lnd* container so that you can use the `lncli` command (eg. `lncli getinfo`) |
| `docker-compose restart bitcoin` | Restart the *bitcoin/lnd/electrs* container |
| `docker-compose build --no-cache` | Rebuild all the containers from scratch (eg. after changing bitcoin version in `.env`)|
| `docker-compose down` | Stop all the containers |
| `docker-compose up -d ` | Start all the containers |
| `docker-compose up -d --build` | Rebuild and start all the containers |

# Directories structure
```sh
├── configs
│   ├── bitcoin.conf
│   ├── electrs.toml
│   ├── lnbits.env
│   ├── lnd.conf
│   ├── rtl.json
│   └── torrc
├── data
│   ├── bitcoin
│   ├── btcpay
│   ├── electrs
│   ├── lnbits
│   ├── lnd
│   ├── mempool
│   │   ├── api
│   │   └── mysql
│   ├── nbx
│   ├── postgres
│   ├── rtl
│   ├── scb
│   └── tor
├── docker-compose-sample.yml
├── dockerfiles
│   ├── Dockerfile.bitcoin
│   ├── Dockerfile.electrs
│   ├── Dockerfile.lnbits
│   ├── Dockerfile.lnd
│   ├── Dockerfile.nbx
│   ├── Dockerfile.nginx
│   ├── Dockerfile.rtl
│   ├── Dockerfile.scb
│   ├── Dockerfile.tor
│   └── entrypoints
│       ├── lnd.sh
│       └── scb.sh
├── .env.sample
├── fragments
│   ├── bitcoin.yml
│   ├── btcpay.yml
│   ├── electrs.yml
│   ├── lnbits.yml
│   ├── lnd.yml
│   ├── mempool.yml
│   ├── nginx.yml
│   ├── rtl.yml
│   ├── scb.yml
│   └── tor.yml
├── LICENSE
├── README.md
└── run.sh

```

| Directory | Description |
| --- | --- |
| `configs` | Here you can find all the configuration files. Feel free to edit them as you like, but please be carefull to not mess-up with authentication method: Currently uses cookies authentication between services instead of RPC. |
| `data` | Here is where the data are persisted. The Bitcoin Blockchain, the Electrs indexes, the LND channels, etc. are all stored here. |
| `dockerfiles` | Here you can find and inspect all the files used to build the images. **Don't trust, verify**! |
| `fragments` | All the services that can be added to the `docker-compose.yml` during the setup tutorial with the `run.sh` utility script |


<a name="5"></a>
# BTCPay Server (optional)

You can easily run your own self-hosted instance of [BTCPay Server](https://btcpayserver.org/) with just a few slight modification to a couple of files provided with this repository:

| File | Modification |
| --- | --- |
| `docker-compose.yml` | <ul><li> Replace the `depends_on` directive under the `nginx` section with the one provided.</li><li>Uncomment the `ports` **8083** and **8084** under the `nginx` section. </li> <li>Uncomment the `btcpay`, `nbx` and `postgres` services blocks.</li></ul>|
| `configs/nginx-reverse-proxy.conf` | Uncomment the `upstream` btcpay and **8083**, **8084** `server` blocks. |

Run `docker-compose down` and then `docker-compose up -d` again.

BTCPay server will run 3 additionals containers (requred files and directories are already present on this repository):
- [Postgres](https://github.com/btcpayserver/dockerfile-deps/tree/master/Postgres/13.13)
- [NBXplorer](https://github.com/dgarage/NBXplorer)
- [BTCPay-server](https://btcpayserver.org/)

# How to update Bitcoin/LND/Electrs version

If you wish to update Bitcoin/LND/Electrs version just edit the `.env` file and run the following commands to stop and rebuild the containers:

```sh
$ docker-compose down
$ docker-compose up -d
```
# How to update

If you wish to update to the last version just run the following commands:

```sh
$ docker-compose down
$ git stash
$ git pull
$ git stash apply
```
Resolve any `git` conflicts and run the following command:
```sh
$ docker-compose up -d --build
```
# Support

For any questions or issues you can open a [Github issue](https://github.com/PatMulligan/btcstack-docker/issues).

# Donations/Project contributions
> [!TODO]
> Add Donation link with split to giovantenne
If you would like to contribute and help dev team with this project you can send a donation to the following LN address ⚡``⚡ or on-chain   ``

Enjoy!
