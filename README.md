# Deploy sunodo DApp in testnet instructions

Create a file containing the network configuration testnet.env:

```shell
#!/bin/sh

export MNEMONIC=""

export NETWORK=
export RPC_URL=
export WSS_URL=
export CHAIN_ID=
```

Source it

```shell
source testnet.env
```

Download file:

```shell
wget https://github.com/lynoferraz/sunodo-deploy-hack/blob/main/testnet-deploy.yml
```

Run the following commands

```shell
docker compose -f testnet-deploy.yml run authority-deployer
docker compose -f testnet-deploy.yml run dapp-deployer
```

Create a .sunodo.env file containing the cartesi node network configurations (change values of everithing in `[]`):

```shell
### contract-address-file
DAPP_CONTRACT_ADDRESS_FILE=/usr/share/sunodo/dapp-[NETWORK].json

### chain-id
CHAIN_ID=[CHAIN_ID]
TX_CHAIN_ID=[CHAIN_ID]

# dispatcher
## uses redis
## uses chain-id (TX_CHAIN_ID acctually)
AUTH_MNEMONIC=[MNEMONIC]
RD_DAPP_DEPLOYMENT_FILE=/usr/share/sunodo/dapp-[NETWORK].json
RD_ROLLUPS_DEPLOYMENT_FILE=/usr/share/sunodo/[NETWORK].json

SC_DEFAULT_CONFIRMATIONS=10
TX_PROVIDER_HTTP_ENDPOINT=[RPC_URL]
TX_DEFAULT_CONFIRMATIONS=11

# state-server
SF_GENESIS_BLOCK=[DAPP_DEPLOY_BLOCK]
SF_SAFETY_MARGIN=20
BH_HTTP_ENDPOINT=[RPC_URL]
BH_WS_ENDPOINT=[WSS_URL]
BH_BLOCK_TIMEOUT=120

# advance-runner
PROVIDER_HTTP_ENDPOINT=[RPC_URL]
```

Check where sunodo is installed on your system. E.g. linux you can check with

```shell
npm list -g 
```

And define the sunodo path. E.g.:

```shell
sunodo_path=~/.nvm/versions/node/v18.18.0/lib/node_modules/@sunodo/cli/dist
```

Since sunodo services depend on each other, start the anvil, database, redis

```shell
SUNODO_BIN_PATH=$sunodo_path docker compose -f $sunodo_path/node/docker-compose-dev.yaml -f $sunodo_path/node/docker-compose-explorer.yaml -f $sunodo_path/node/docker-compose-traefik-config.yaml -f $sunodo_path/node/docker-compose-snapshot-volume.yaml -f $sunodo_path/node/docker-compose-envfile.yaml --project-directory . up -d anvil database redis
```

Find the blockchain-data volume, and inspect it (set dapp variable):

```shell
docker volume ls
dapp=
docker volume inspect ${dapp}_blockchain-data
```

Then copy the rollups.json file generated on a previous command the volume, so the validator node has the correct addresses. In linux the volume path should be something like `/var/lib/docker/volumes/${dapp}_blockchain-data/_data` (it should be run by superuser):

```shell
sudo "NETWORK=$NETWORK" dapp=$dapp cp .deployments/$NETWORK/rollups.json /var/lib/docker/volumes/${dapp}_blockchain-data/_data/$NETWORK.json
sudo "NETWORK=$NETWORK" dapp=$dapp cp .deployments/$NETWORK/dapp.json /var/lib/docker/volumes/${dapp}_blockchain-data/_data/dapp-$NETWORK.json
```

Then start the validador and prompt:

```shell
SUNODO_BIN_PATH=$sunodo_path docker compose -f $sunodo_path/node/docker-compose-dev.yaml -f $sunodo_path/node/docker-compose-explorer.yaml -f $sunodo_path/node/docker-compose-traefik-config.yaml -f $sunodo_path/node/docker-compose-snapshot-volume.yaml -f $sunodo_path/node/docker-compose-envfile.yaml --project-directory . up --attach prompt --attach validator
```

Chech the running services with:

```shell
SUNODO_BIN_PATH=$sunodo_path docker compose -f $sunodo_path/node/docker-compose-dev.yaml -f $sunodo_path/node/docker-compose-explorer.yaml -f $sunodo_path/node/docker-compose-traefik-config.yaml -f $sunodo_path/node/docker-compose-snapshot-volume.yaml -f $sunodo_path/node/docker-compose-envfile.yaml --project-directory . ps
```

Then to stop all services run:

```shell
SUNODO_BIN_PATH=$sunodo_path docker compose -f $sunodo_path/node/docker-compose-dev.yaml -f $sunodo_path/node/docker-compose-explorer.yaml -f $sunodo_path/node/docker-compose-traefik-config.yaml -f $sunodo_path/node/docker-compose-snapshot-volume.yaml -f $sunodo_path/node/docker-compose-envfile.yaml --project-directory . down -v
```