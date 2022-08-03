# Algorand - Build a private network

# 1. Overview

- Origin docs: https://developer.algorand.org/docs/run-a-node/setup/types/
- Protocol: PoS
- Target: build 3 node, include 2 node non-replay + 1 node replay

# 2. Create a private network

- To build docker image, follow [build-docker-image.md](build-docker-image.md)

```
node0 - replay node
node1 - non-replay node
node2 - non-replay node
```

## a. Generate token

- Run command in folder `node0`:
  ```sh
  docker run --rm -v $(pwd)/algorand_data:/root/node/data go-algorand bash -c "ALGORAND_DATA=/root/node/data goal node generatetoken"
  ```
  - The `algod.token` API token will be generated and installed into the directory `algorand_data`.
  - Do the same with `node1`, `node2` folder.

## b. Create the Genesis File
- A final file of `genesis.json` in this repo.

### Create the Wallets
- On both node1 & node2:
```sh
mkdir algorand_data/privatenet-v1
docker run --rm -it -v $(pwd)/algorand_data:/root/node/data go-algorand bash
ALGORAND_DATA=/root/node/data goal kmd start -t 3600 && goal wallet new <wallet_name>
```
- node1:
```sh
ALGORAND_DATA=/root/node/data goal kmd start -t 3600 && goal wallet new algodWallet

Successfully started kmd
Please choose a password for wallet 'algodWallet': 
Please confirm the password: 
Creating wallet...
Created wallet 'algodWallet'
Your new wallet has a backup phrase that can be used for recovery.
Keeping this backup phrase safe is extremely important.
Would you like to see it now? (Y/n): Y
Your backup phrase is printed below.
Keep this information safe -- never share it with anyone!

air cannon milk fiction tongue anxiety grid artefact shove home return glove hotel toast ring clay shed start liquid poverty marriage bachelor dress above off
```
- node2:
```sh
ALGORAND_DATA=/root/node/data goal kmd start -t 3600 && goal wallet new algodWallet

Successfully started kmd
Please choose a password for wallet 'algodWallet': 
Please confirm the password: 
Creating wallet...
Created wallet 'algodWallet'
Your new wallet has a backup phrase that can be used for recovery.
Keeping this backup phrase safe is extremely important.
Would you like to see it now? (Y/n): Y
Your backup phrase is printed below.
Keep this information safe -- never share it with anyone!

reunion check page item business reward air potato february fade guard cereal churn assist river electric above table limit exchange casino ready slogan absorb announce
```

### Create the Accounts
```sh
docker run --rm -it -v $(pwd)/algorand_data:/root/node/data go-algorand bash
ALGORAND_DATA=/root/node/data goal account new -w <wallet_name> -f
```
- node1:
```
ALGORAND_DATA=/root/node/data goal account new -w algodWallet -f

Please enter the password for wallet 'algodWallet': 
Created new account with address MADSFSIWEGIPMS53C7AXIPAGQGBQL2DGPJIMFARDOJPDXY4BIZMV3QJNA4
```
- node2:
```
ALGORAND_DATA=/root/node/data goal account new -w algodWallet -f

Please enter the password for wallet 'algodWallet': 
Created new account with address 7G6Q746DX5UE7T7RGJLLYA7BZ7OBAUGW64QCABZHGAME7O6MKHO6VFHM6I
```

### Generate Participation Keys
```sh
goal account addpartkey -a <address-of-participating-account> --roundFirstValid=<partkey-first-round> --roundLastValid=<partkey-last-round> [--keyDilution=<key-dilution-value>]
```
```sh
# node1
goal account addpartkey -a MADSFSIWEGIPMS53C7AXIPAGQGBQL2DGPJIMFARDOJPDXY4BIZMV3QJNA4 --roundFirstValid=<partkey-first-round> --roundLastValid=<partkey-last-round>
# node2
goal account addpartkey -a 7G6Q746DX5UE7T7RGJLLYA7BZ7OBAUGW64QCABZHGAME7O6MKHO6VFHM6I --roundFirstValid=<partkey-first-round> --roundLastValid=<partkey-last-round>
```

### Update the Genesis and Distribute the Initial Stake
- [genesis.json](genesis.json)

## c. Nodes Configuration
- `config.json` in data folder.

### Participation nodes (non-replay node) configuration
```json
{
  "Version": 14,
  "GossipFanout": 2,
  "IncomingConnectionsLimit": 0,
  "DNSBootstrapID": "",
  "EnableProfiler": true
}
```

### Replay nodes configuration
```json
{
  "Version": 14,
  "GossipFanout": 2,
  "NetAddress": ":4161",
  "DNSBootstrapID": "",
  "EnableProfiler": true
}
```

## d. Connect Participation and Relay nodes
- `phonebook.json` in `/go/bin` folder.

## e. Start the Network
- Remove data init in step `Generate Participation Keys`:
```sh
rm algorand_data/privatenet-v1/crash.*
rm algorand_data/privatenet-v1/ledger.*
```

- Start nodes:
```sh
goal node start
```

- Monitor
```sh
goal node status -w 1000

Last committed block: 2
Time since last block: 3.5s
Sync Time: 0.0s
Last consensus protocol: https://github.com/algorandfoundation/specs/tree/4a9db6a25595c6fd097cf9cc137cc83027787eaa
Next consensus protocol: https://github.com/algorandfoundation/specs/tree/4a9db6a25595c6fd097cf9cc137cc83027787eaa
Round for next consensus protocol: 3
Next consensus protocol supported: true
Has Synced Since Startup: false
```