# Algorand - Build a private network

# 1. Overview

- Origin docs: https://developer.algorand.org/docs/run-a-node/setup/types/
- Protocol: PoS
- Target: build 3 node, include 2 node non-relay + 1 node relay

# 2. Create a private network

- To build docker image, follow [build-docker-image.md](build-docker-image.md)

```
node0 - relay node
node1 - non-relay node
node2 - non-relay node
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
Created new account with address MVYXHKNAOAHY3A37FYMXXTP5Z5XUE5U2D5JFEWNCXWYAZWCGTWBKDZ2AVM
```
- node2:
```
ALGORAND_DATA=/root/node/data goal account new -w algodWallet -f

Please enter the password for wallet 'algodWallet': 
Created new account with address 6QEPMIGFHK36BJMCNC24NFYRFWRQUCCKMELM56ETOMREHSDRCUHXGOZUQE
```

### Generate Participation Keys
- To execute the goal addpartkey command a node needs to be started:
```sh
goal node start
```

- Gen keys:
```sh
goal account addpartkey -a <address-of-participating-account> --roundFirstValid=<partkey-first-round> --roundLastValid=<partkey-last-round> [--keyDilution=<key-dilution-value>]
```

- node1:
```sh
goal account addpartkey -a MVYXHKNAOAHY3A37FYMXXTP5Z5XUE5U2D5JFEWNCXWYAZWCGTWBKDZ2AVM --roundFirstValid=0 --roundLastValid=10000000

Please stand by while generating keys. This might take a few minutes...
Participation key generation successful. Participation ID: HR44MZRXYY6DUYPVD3TP2AWSOE4CCHTQEWZZ5ZMP5SOX7H6RMICA

Generated with goal v3.9.153003
```

- node2:
```
goal account addpartkey -a 6QEPMIGFHK36BJMCNC24NFYRFWRQUCCKMELM56ETOMREHSDRCUHXGOZUQE --roundFirstValid=0 --roundLastValid=10000000

Please stand by while generating keys. This might take a few minutes...
Participation key generation successful. Participation ID: AYACKOMPOQ6HYLGNE5HMXG37RWOR67GMYQGHBPHSZZKUOBWMM2NQ

Generated with goal v3.9.153003
```

### Update the Genesis and Distribute the Initial Stake
- Dump all the information about each participation key that lives on the node.

- node1:
```sh
goal account partkeyinfo

Dumping participation key info from /root/node/data...

Participation ID:          VEHMG5AGPH2ABZKCXDMDOU754M24ZYYQUW2AQKBWWNQ7S55HFV5Q
Parent address:            MVYXHKNAOAHY3A37FYMXXTP5Z5XUE5U2D5JFEWNCXWYAZWCGTWBKDZ2AVM
Last vote round:           N/A
Last block proposal round: N/A
Effective first round:     N/A
Effective last round:      N/A
First round:               0
Last round:                10000000
Key dilution:              3163
Selection key:             DU5tZXjvp/s+6SkWDjg6JBJbUkGSCM51+4KJGEYqkxE=
Voting key:                QBfi5goC6HwMhK6sMeSjITzSXooUxWoTVpfCClsCBP8=
State proof key:           kqYEGPjKZREQ9PreiWnBJNgChDx9QohsJK7u4o1YL5YuUeDbdtYaHowC2PXwJu678nZLmNeGa0dIDhERyktlKg==
```

- node2:
```sh
goal account partkeyinfo

Dumping participation key info from /root/node/data...

Participation ID:          2RAC6IAAYAGPUVAOZKOEHXXDG4LBVF2O6P3P424WGXOJ5KPB6GCA
Parent address:            6QEPMIGFHK36BJMCNC24NFYRFWRQUCCKMELM56ETOMREHSDRCUHXGOZUQE
Last vote round:           N/A
Last block proposal round: N/A
Effective first round:     N/A
Effective last round:      N/A
First round:               0
Last round:                10000000
Key dilution:              3163
Selection key:             gGgSOV4jgl5mYMOAfmJaYv0gpJLjQLat3x7vAL6CX+g=
Voting key:                chFdK2752N6zcwjZTuVAbs/xKwNrbPP2l9itRwC9Ct0=
State proof key:           Lbh/HWF1pG9hOFML+YnGdOdxahR1N/Pgz8YV+vJsHMVcJCt8AijSsyVdFqWZHx+h5W7uKR5vwH2ZHy4zg7uXUw==
```

- [genesis.json](genesis.json)

## c. Nodes Configuration
- `config.json` in data folder.

### Participation nodes (non-relay node) configuration
```json
{
  "Version": 14,
  "GossipFanout": 2,
  "IncomingConnectionsLimit": 0,
  "DNSBootstrapID": "",
  "EnableProfiler": true
}
```

### Relay nodes configuration
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
rm -rf algorand_data/privatenet-v1/crash.*
rm -rf algorand_data/privatenet-v1/ledger.*
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