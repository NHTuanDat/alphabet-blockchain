# Table of contents

1. [Introduction](#1-introduction)
2. [How to use](#2-how-to-use)

   2.1 [Install geth](#21-install-geth)

   2.2 [Create nodes](#22-create-nodes)

   2.3 [Setup genesis block](#23-setup-genesis-block)

   2.4 [Initializing the Geth Database](#24-initializing-the-geth-database)

   2.3 [Bootnode](#25-boot-node)

   2.3 [Copy startnode script](#26-copy-startnode-script)

   2.3 [Run the scripts](#27-run-the-scripts)

## 1. Introduction

Window batscript for deploying a local private blockchain network using [`geth`](https://geth.ethereum.org/).
And an tldr on how to use `geth` to set up private server on Window. This is for my own use only so it is quite basic.
If you want more information you can check the [official guide](https://geth.ethereum.org/docs/interface/private-network).

## 2. How to use

### 2.1 Install geth

Go to [geth](https://geth.ethereum.org/) (Go inplementation of Ethereum blockchain) website to download geth for your system.
Then add to system path the installation folder.

### 2.2 Create nodes

Before executing any commands, let's first create an empty folder and enter it. This empty folder will be our project root.
Then create new folders for nodes, it is advisable to has a minimum of 3 nodes.

```
$ mkdir node1
$ mkdir node2
$ mkdir node3
```

Then create an account for each node using `geth`, type in and remember the password.

```
$ geth account new --datadir ./node1/
$ geth account new --datadir ./node2/
$ geth account new --datadir ./node3/
```

Save passwords in a hidden file in each nodes. Here I use `vi`, you can use any text editor you want.
Type in the password for the account in each file then save it.

```
$ vi ./node1/.password
$ vi ./node2/.password
$ vi ./node3/.password
```

### 2.3 Setup genesis block

Now we set up genesis block for our private chain.
To do this we use a tool in geth call `puppeth`, A CLI tool for managing genesis block.

```
$ puppeth
+-----------------------------------------------------------+
| Welcome to puppeth, your Ethereum private network manager |
|                                                           |
| This tool lets you create a new Ethereum network down to  |
| the genesis block, bootnodes, miners and ethstats servers |
| without the hassle that it would normally entail.         |
|                                                           |
| Puppeth uses SSH to dial in to remote servers, and builds |
| its network components out of Docker containers using the |
| docker-compose toolset.                                   |
+-----------------------------------------------------------+

Please specify a network name to administer (no spaces, hyphens or capital letters please)
>
```

Type the name of your blockchain network, then select option `2. Configure new genesis` -> `1. Create new genesis from scratch`.

```
Please specify a network name to administer (no spaces, hyphens or capital letters please)
> tmp

Sweet, you can set this via --network=tmp next time!

INFO [08-09|08:40:37.479] Administering Ethereum network           name=tmp
WARN [08-09|08:40:37.481] No previous configurations found         path=.puppeth\tmp

What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
> 2

What would you like to do? (default = create)
 1. Create new genesis from scratch
 2. Import already existing genesis
> 1

Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
>
```

Here you can choose either PoW or PoA, depend on what you need.
Keep going with the options, `pre-funded` mean that it will start with some amount of eth.
`network id` is also really important, `1` is mainnet network id.

After all the options, enter `2. Manage existing genesis` -> `2. Export genesis configurations`.
And export it into current folder (project root)
You can exit `puppeth` by `Ctrl+C` or `Ctrl+D`.

Now if you check in project root folder, you can see a `json` file, it is the genesis config file.

```
./genesis.json

{
  "config": {
    "chainId": 45434,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0,
    "istanbulBlock": 0,
    "ethash": {}
  },
  "nonce": "0x0",
  "timestamp": "0x62f1c0b4",
  "extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "gasLimit": "0x47b760",
  "difficulty": "0x80000",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "0000000000000000000000000000000000000000": {
      "balance": "0x1"
    },
    "0000000000000000000000000000000000000001": {
      "balance": "0x1"
    },
    "0000000000000000000000000000000000000002": {
      "balance": "0x1"
    },

    ...

    "00000000000000000000000000000000000000fe": {
      "balance": "0x1"
    },
    "00000000000000000000000000000000000000ff": {
      "balance": "0x1"
    }
  },
  "number": "0x0",
  "gasUsed": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "baseFeePerGas": null
}
```

Here you can check the `network id` if you forgot.

### 2.4 Initializing the Geth Database

Start initializing all the nodes with this genesis block by execute

```
$ geth init --datadir ./node1/ genesis.json
$ geth init --datadir ./node2/ genesis.json
$ geth init --datadir ./node3/ genesis.json
```

### 2.5 Boot node

For all the node to find eachother, it need to connect to the boot node, which will help our nodes discover eachother.
Generating `boot.key` first

```
$ mkdir bootnode
$ cd bootnode
./bootnode/$ bootnode -genkey boot.key
./bootnode/$ cd ..
```

Then now we can start our boot node with

```
$ bootnode -nodekey ./bootnode/boot.key
enode://b60c1e...094af0@127.0.0.1:0?discport=30301
Note: you're using cmd/bootnode, a developer tool.
We recommend using a regular node as bootstrap node for production deployments.
INFO [08-09|09:27:33.253] New local node record                    seq=1,660,012,053,252 id=bbf6fc13239b8a6b ip=<nil> udp=0 tcp=0
```

Remember to save the `enode:...` line it is address that other node will try to connect (`<bootstrap-node-record>`).

### 2.6 Copy startnode script

You can try to start the blockchain with the command

```
$ geth --datadir ./node1/ --networkid <network id>
```

But this is quite limited, but also the amount of options and flags is quite much, so we need to write some script file for each nodes.
I already create an script temlplate file called `startnode.cmd`. I'll start to explain all the options

```
./starnode.cmd

geth --datadir <node dir> --networkid <network id> --bootnodes <bootstrap-node-record>^
--mine --miner.threads 1 --port <discovery port> --nat "any" --http --http.port <http port>^
--http.corsdomain "*" --http.api eth,web3,personal,net --allow-insecure-unlock --unlock 0 --password "./<node dir>/.password"^
--authrpc.port <rpc port> console
```

Note that it should only be one line, and each node has different `startnode.cmd` file

```
--datadir <node dir> *
        ./node1/, ./node2/ and so on

--networkid <network id>
        Your network id above
        Ex: 45434

--bootnodes <bootstrap-node-record>
        Your bootstrap node above
        Ex: enode://b60c1e...094af0@127.0.0.1:0?discport=30301

--mine, --miner.threads <number of threads>
        Enable mining for node and how many cpu thread it use
        Ex: 1

--port <discovery port> *
        Discover port for other node to find it
        Ex: "30307"

--http, --http.port <http port> *
        Enable http to be able to interact with blockchain using commands and metamask.
        Ex: "8543"

--authrpc.port <rpc port> *
        RPC port?
        Ex: "8551"

--password "./<node dir>/.password"
        password file in the node folder
        Ex: node1
```

Options with `*` mean each node need be different
More options and explanations can be found in `gethHelp.txt` or `geth --help`.

Copy `startnode.cmd` for each node dir then edit them

```
$ cp ./startnode.cmd ./node1/
$ cp ./startnode.cmd ./node2/
$ cp ./startnode.cmd ./node3/
$ vi ./node1/startnode.cmd
```

### 2.7 Run the scripts

Now you can finally run all the scripts, remember to run bootnode first (on different terminal). And also run each commands on different terminal.

```
[Terminal 1]
$ ./node1/startnode.cmd

[Terminal 2]
$ ./node2/startnode.cmd

[Terminal 3]
$ ./node3/startnode.cmd
```

We are done, for more information check the [official documents](https://geth.ethereum.org/docs/interface/private-network)
