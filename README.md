# c-lightning Docker Setup

The docker-compose images and the instructions in this repository are to setup a local Bitcoin network (regtest mode) and run a Lightning Network (c-lightning nodes) over it.

## Setup

There is one docker-compose file for each node in our Lightning Network. Each docker-compose file has two services each, one that runs `bitcoind` and another that runs `lightningd`. We will need an instance of both for each node. We can connect all the c-lightning daemons to the same `bitcoind` instance, but for the sake of completeness we will set up an instance of `bitcoind` for each node.

The bitcoind containers will be named `bitcoind1`, `bitcoind2` and so on, and the corresponding lightningd containers `lightningd1`, `lightningd2` and so on. We will be opening multiple terminal windows to interact with the daemons via CLI. A single terminal can also be used but I find it convenient to have terminals open to each container. I believe using a terminal multiplexer to handle multiple terminals is possible, but I am not an expert at it and is left to you to try :)

We will be setting up a topology as follows:
![topology](images/topology.png "Network Topology")

You can create more nodes by duplicating the docker-compose files and set up a topology of your choice.

All the containers are connected via a `clightning_network` docker network.

Note that there may be a better way to do this, but I am new to docker and this was one way I found to go about this. If there is a more optimal way, feel free to raise a pull request.

## Instructions to build the images and run the containers
Clone the repository onto your PC:
```bash
git clone https://github.com/aaaaa
```

Create the `lightningd` and `bitcoind` containers. Run this for all the docker-compose files in different terminal windows/tabs.
```bash
docker-compose -f "docker-compose-1.yml" up
```
Now, our 4 lightning nodes are set-up. Open 2 terminals each for every node, one to interact with `bitcoind` and one for `lightningd`. Repeat the following for every node in different terminals:

```bash
docker exec -i -t bitcoind1 bash
```
```bash
docker exec -i -t lightnind1 bash
```
We will be using `bitcoin-cli` to interact with `bitcoind` and `lightning-cli` to interact with `lightningd`. Aliases can be used to make your life easier:
```bash
bitcoind1$ cat >> /home/.bashrc
alias bc='bitcoin-cli'

bitcoind1$ source /home/.bashrc
```
```bash
lightningd1$ cat >> /home/.bashrc
alias lc='lightning-cli'

lightningd1$ source /home/.bashrc
```

But using bitcoin-cli runs into a `rpcuser` error where it says credentials are not found. We will have to manually edit the `bitcoin.conf` file present at `/root/.bitcoin/`:
```bash
bitcoind1$ cat > /root/.bitcoin/bitcoin.conf
printtoconsole=1
rpcallowip=::/0
regtest=1
whitelist=0.0.0.0/0
server=1
rpcuser=rpcuser
rpcpassword=rpcpass
```
To ensure that all the nodes have a copy of the same blockchain, establish connections within themselves by appending a `addnode=bitcoind1` to the config file for the subsequent `bitcoind` instances. (`addnode=bitcoind2` can also be added to `bitcoind3` and so on)

I believe this can also be done via the docker-compose file without having to manually doing it after container deployment. Will update the docker-compose files with the same soon.

_in progress_
