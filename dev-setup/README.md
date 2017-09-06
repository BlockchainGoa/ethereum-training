# Ethereum dev network setup

## Getting Started
These instructions will allow you to get a ethereum dev network up and running on your local machine for development and testing purposes. 
* **[Testrpc setup](#testrpc-setup)**<br>
* **[Ethereum dev network setup](#ethereum-dev-network-setup-1)**<br>

# Testrpc setup

Install Command
```sh
npm install -g ethereumjs-testrpc
```

Start testrpc command
```sh
$ testrpc --port 8545
```
testrpc: <https://github.com/ethereumjs/testrpc>


# Ethereum dev network setup

* Installing Ethereum client

For Ubuntu
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

## Steps

###  1] New chain directory 
It will create a directory where all the chain data including wallet keys will be stored.

```sh
$ sudo mkdir dev
```




###  2] Create new account
  This command will create new Ethereum Account keystore at `.../dev/keystore/` and store the keys in the keystore directory.
  
  Default directory = `/root/.ethereum/keystore/`

```sh
$ geth --datadir 'dev' account new
Passphrase: xxxxxxxx
Repeat passphrase: xxxxxxxx
Address: {4e6cf0ed2d8bbf1fbbc9f2a100602ceba4bf1319}
```
Copy the displayed account address


###  3] Start Ethereum client
This command will start the Ethereum client process.
```sh
 $ sudo geth --datadir="dev" --dev  --etherbase="0x0000000000000000000000000000000000000000"  --unlock 0 console  
```
 
 -- dev flag - to use the pre-configured settings provided by geth.
 
 -- etherbase flag - set miner account.
 
 --datadir flag - set the location of chain data
 
 --unlock - keep account unlocked for the session. 0 - account [0] 

Enter the passphrase to unlock the wallet


###  4] Start mining
This command will start the miner process. 
```sh
miner.start()
```

Check current balance in Wei
```
web3.eth.getBalance(eth.accounts[0])
```
Check current balance in Ethers

```
web3.fromWei(web3.eth.getBalance(eth.accounts[0]),'ether')
```


###  5] Connect RPC externally (Not Safe)

This command will allow you to connect your Ethereum client from external device on same network. 
 --rpc start rpc server.This is generally enabled by default in Geth.
 
 --rpcport select rpc port. Default port is 8545
 
 --rpcaddr "0.0.0.0"   '(Not Safe)'
 
 --rpccorsdomain specify URLs which can connect to your node in order to perform RPC client tasks '(Not Safe)'
 
Add to start command
```
--rpc --rpcaddr "0.0.0.0" --rpcport 8546 --rpccorsdomain "*"  --rpcapi "db,eth,net,web3,personal"
```
OR
```
 admin.startRPC("0.0.0.0", 8546, "*", "web3,net,eth")
 admin.stopRPC()
```



sample:
```sh
$ sudo geth --datadir="dev" --dev --rpc --rpcaddr "0.0.0.0" --rpcport 8546 --rpccorsdomain "*"  --etherbase="0x0000000000000000000000000000000000000000"  --nodiscover --unlock 0 console
```
