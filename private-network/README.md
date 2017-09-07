# Ethereum private network setup

* **[Pre-requisites](#pre-requisites)**
* **[Setup](#setup)**

## Getting Started

These instructions will allow you to get a ethereum private network up and running on your local machine for development and testing purposes. 

* VM Specification

**OS version :** 

Ubuntu 14.04.

RAM: 4GB

CPU: 2 cores

## Pre-requisites

* How to install go-ethereum

```sh
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```
Check version
```sh
$ geth version
```


Other Operating systems: <https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum>

## Setup


### Steps for setting up 2 peer network
###  1]  Create new account on both peers

* Open new terminal 

This step will create a new keystore file at location `.../peer1/keystore/`

Default directory = `/root/.ethereum/keystore/`


```sh
$ sudo mkdir peer1
$ geth --datadir="peer1" account new
Passphrase: xxxxxxxx
Repeat passphrase: xxxxxxxx
Address: {4e6cf0ed2d8bbf1fbbc9f2a100602ceba4bf1319}
```
 Copy the displayed account address

* `Open second terminal and follow above steps with folder name as peer2`


###  2]  Setup genesis.json
 The genesis block is the start of the blockchain, and the genesis.json is the file that defines it.

sample here:
```javascript
{
    "config": {
        "chainId": 56,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },

    "alloc": {
        "0x1517866f9935de27fea9da34e5a89c2de39855a2": {
            "balance": "1000000000000000000"
        }
    },

    "coinbase": "0x0000000000000000000000000000000000000000",
    "difficulty": "0x0400",
    "extraData": "",
    "gasLimit": "0x8000000",
    "nonce": "0x0000000000000042",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "timestamp": "0x00"
}
```


###  3]  Add network details to genesis.json

* Add network id
```javascript
chainid = networkId 
```
* Add Ethereum address of accounts created and balance in Wei (1 ethers=1000000000000000000 wei)
```javascript
alloc:{
"[ Address here with 0x ]": { "balance": "1000000000000000000"}
}
```

###  4] Create genesis block in both peers:
 This command initializes the genesis block for the network. 
```sh
 $ sudo geth init "PathToGenesis.json"
```

###  5] Start first peer:
This command starts the ethereum client process
 --datadir -data directory that your private chain data will be stored in.
 -- networkid flag - select network id.
 --port - network listening port. Default port is 30303
 --rpc start rpc server.This is generally enabled by default in Geth.
 --rpcport select rpc port. Default port is 8545
 --rpcaddr "0.0.0.0"   '(Not Safe)'
 --rpccorsdomain specify URLs which can connect to your node in order to perform RPC client tasks '(Not Safe)'
 --nodiscover Dont connect to other peers automatically
 --unlock - keep account unlocked for the session. 0 - account [0] 

```sh
$ sudo geth --datadir="peer1" --networkid 56 --port 30303 --rpc --rpcport 8545 --rpcaddr "0.0.0.0"  --rpccorsdomain "*"  --nodiscover --unlock 0 console  
```


### 6] Start Second peer at different port : 

Start second peer as Miner
 -- etherbase flag - set miner account.
 --mine - Start mining

```sh
$ sudo geth --datadir="peer2"  --networkid 56 --port 30304 --rpc  --rpcport 8546 --rpcaddr "0.0.0.0" --rpccorsdomain "*" --etherbase "0x0000000000000000000000000000000000000000" --mine --nodiscover --unlock 0 console
```
stop mining:
```sh
miner.stop()
miner.start()
```

###  7] Connect peers

* get enode id
```javascript
admin.nodeInfo.enode
"enode://704899633efe7dc6178dad6e66c318bebb8633bc8fe141b2f53e7104b5feee3f0c5eb153e634d9939e73b75c268d36a16bd5f7b15d91090452f60508455cc61d@[::]:30303"
```
* add peer with enode id
```
 admin.addPeer("enode://704899633efe7dc6178dad6e66c318bebb8633bc8fe141b2f53e7104b5feee3f0c5eb153e634d9939e73b75c268d36a16bd5f7b15d91090452f60508455cc61d
@33.4.2.1:30303")
```
format: "enode://pubkey@ip:port"

Geth supports a feature called static nodes if you have certain peers you always want to connect to. Static nodes are re-connected on disconnects. You can configure permanent static nodes by putting something like the following into `datadir/static-nodes.json` (this should be the same folder that the chaindata and keystore folders are in)

```javascript
[
  "enode://f4642fa65af50cfdea8fa7414a5def7bb7991478b768e296f5e4a54e8b995de102e0ceae2e826f293c481b5325f89be6d207b003382e18a8ecba66fbaf6416c0@33.4.2.1:30303",
  "enode://pubkey@ip:port"
]
```
* Check peer connection
```
admin.peers
[{
  ID: 'a4de274d3a159e10c2c9a68c326511236381b84c9ec52e72ad732eb0b2b1a2277938f78593cdbe734e6002bf23114d434a085d260514ab336d4acdc312db671b',
  Name: 'Geth/v0.9.14/linux/go1.4.2',
  Caps: 'eth/60',
  RemoteAddress: '5.9.150.40:30301',
  LocalAddress: '192.168.0.28:39219'
}]
```
###  Additional Steps

 new connection to the geth process
  ```sh
  $ geth attach ipc:./geth.ipc
  ```
  Check balance in account
  ```sh
  primary = eth.accounts[0];
  web3.fromWei(eth.getBalance(primary), "ether");
  ```


private network: <https://github.com/ethereum/go-ethereum>

## Contract Deployment


Compile Deploy contracts using browser 
browser-solidity:  <http://ethereum.github.io/browser-solidity/>

