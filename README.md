<p align="center">
<img src="./.github/logo.svg" width="500px" alt="NeoFS">
</p>
<p align="center">
  <a href="https://fs.neo.org">NeoFS</a> local Development and Testing environment
</p>

---
## Overview

Tools to set up local NeoFS network and N3 privnets. Devenv, for short.

## Prerequisites

Make sure you have installed all of the following prerequisites on your machine:
* docker
* docker-compose
* make (`3.82+`)
* expect
* openssl
* jq
* base64 (coreutils)


## Quick Start

Clone repo: 

```
$ git clone https://github.com/nspcc-dev/neofs-dev-env.git
```

Run next commands from project's root:

```
$ make get
```

This command should be executed for the first run only to execute 
`make hosts`. It is part of the `make up` and, if the hosts have 
been added already, there is no need to run it separately.

```
$ make hosts
192.168.130.10 bastion.neofs.devenv
192.168.130.50 main-chain.neofs.devenv
192.168.130.61 ir01.neofs.devenv
...
192.168.130.74 s04.neofs.devenv
```

This command shows addresses and hostnames of components. Add `make hosts`
output to your local `/etc/hosts` file.

Run all services with command:
``` 
$ make up
```

When all services are up, you need to make GAS deposit for test wallet to be
able to pay for NeoFS operations. Test wallet is located in
`wallets/wallet.json`. The password is empty.

```
$ make prepare.ir
password >
fa6ba62bffb04030d303dcc95bda7413e03aa3c7e6ca9c2f999d65db9ec9b82c
```

Also you should add self-signed node (`s04.neofs.devenv`) certificate to trusted
store (default location might be changed using `CA_CERTS_TRUSTED_STORE`
variable). This step is required for client services (neofs-http-gw,
neofs-s3-gw) to interact with the node:

```
$ make prepare.storage
```

Change NeoFS global configuration values with `make update.*` commands. The
password of inner ring wallet is `one`. See examples in `make help`.

```
$ make update.epoch_duration val=30
Changing EpochDuration configration value to 30
Enter account NNudMSGzEoktFzdYGYoNb3bzHzbmM1genF password > 
Sent invocation transaction dbb8c1145b6d10f150135630e13bb0dc282023163f5956c6945a60db0cb45cb0
Updating NeoFS epoch to 2
Enter account NNudMSGzEoktFzdYGYoNb3bzHzbmM1genF password > 
Sent invocation transaction 0e6eb5e190f36332e5e5f4e866c7e100826e285fd949e11c085e15224f343ba6
```

For instructions on how to set up DevEnv on macOS, please refer [the
guide](docs/macOS.md) in `docs` directory.

## How it's organized

```
.
├── Makefile         # Commands to manage devenv
├── .services        # List of services to work with
├── services         # Services definitions and files
│   ├── basenet
│   ├── chain
│   ├── ir
│   ├── morph_chain
│   └── storage
├── vendor           # Temporary files and artifacts
└── wallets          # Wallet files to manage GAS assets
```

Main commands and targets to manage devenv's services are in `Makefile`.

Each service is defined in it's own directory under `services/` with all
required files to run and scripts to get external artifacts or dependencies.

The list of services and the starting order is defined in `.services` file. You
can comment out services you don't want to start or add your own new services.

You can find more information on each service in `docs` directory.

Maybe you will find the answer for your question in [F.A.Q.](docs/faq.md)

## Using NeoFS Admin Tool in `dev-env`

Devenv supports NeoFS network management via [neofs-adm](https://github.com/nspcc-dev/neofs-node/tree/master/cmd/neofs-adm).
`services/ir` contains the Alphabet wallet in a proper format, specify it
with `--alphabet-wallets` flag.

## Notable make targets

`make help` will print the brief description of available targets. Here we
describe some of them in a more detailed way.

### up

Start all Devenv services.

This target call `pull` to get container images, `get` to download required
artifacts, `vendor/hosts` to generate hosts file and then starts all services in
the order defined in `.services` file.

### down

Shutdowns all services. This will destroy all containers and networks. All
changes made inside containers will be lost.

### hosts

Display addresses and host names for each running service, if available.

### clean

Clean up `vendor` directory.

## How to prepare environment to run neofs-testcases

If necessary, change the IPV4_PREFIX value in the .env file from 192.168.130
to the existing subnet on the host.
Importantly, the subnet mask must be 24.
Use ```ip a``` to see the network configuration on the host.

Make a backup of the ```/etc/hosts``` file.

From the root of the project, run:

```bash
sudo ./bin/update_hosts.sh
```
This script will change the subnet address in the services/coredns/Corefile and
in the services/morph_chain/protocol.privnet.yml configuration files.
And it will add or change ip addresses in the /etc/hosts file.
After running the script, make sure that the contents of these files
are correct.

```bash
make prepare-test-env
```

## Contributing

Feel free to contribute to this project after reading the [contributing
guidelines](CONTRIBUTING.md).

Before starting to work on a certain topic, create an new issue first,
describing the feature/topic you are going to implement.

# License

- [GNU General Public License v3.0](LICENSE)
