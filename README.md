# Bitcoin and lightning node using Docker 

## Introduction
This guide is to help you setup a bitcoin and lightning node on any computer that is capable of running docker. You will need to have docker desktop installed. If not, visit the [Docker website](https://www.docker.com/). You will also need Git. Go here to download and install [Git for windows](https://gitforwindows.org/). On linux use your package manager to install git.


## Download the files
Start a windows powershell or bash terminal and run this command.
```sh
#git clone https://github.com/findamak/bitcoin-lightning-node.git
```
Now run this command to change into that newly created directory.
```sh
#cd bitcoin-lightning-node
```

## Installation
We will use docker compose to setup our project. Docker compose prepends the names of your images, containers, volumes, network with the name of your working directory. Use the below command to customise the project name to "lab" and begin the build of the project.

```sh
#docker compose -p lab up -d --build
```

Once the build completes, check that the containers are up in the "STATUS" column.
```sh
#docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS                     PORTS                    NAMES
9330bd46b5c5   lab-lightning   "/bin/sh -c '/var/li…"   2 minutes ago   Up 2 minutes (unhealthy)   0.0.0.0:9735->9735/tcp   lab-lightning-1
c40c9b3f5fa0   lab-bitcoin     "/bin/sh -c '/usr/sb…"   2 minutes ago   Up 2 minutes (healthy)     0.0.0.0:8333->8333/tcp   lab-bitcoin-1
```

Then we need to create a new lightning wallet for our lightning node. Follow the instructions. Note I do not enter a passphrase to encrypt the cipher seed.
```sh
#docker exec -it lab-lightning-1 lncli create
Input wallet password:
Confirm password:

Do you have an existing cipher seed mnemonic or extended master root key you want to use?
Enter 'y' to use an existing cipher seed mnemonic, 'x' to use an extended master root key
or 'n' to create a new seed (Enter y/x/n): n

Your cipher seed can optionally be encrypted.
Input your passphrase if you wish to encrypt it (or press enter to proceed without a cipher seed passphrase):

Generating fresh cipher seed...

!!!YOU MUST WRITE DOWN THIS SEED TO BE ABLE TO RESTORE THE WALLET!!!

---------------BEGIN LND CIPHER SEED---------------
 1. above     2. priority   3. various   4. scrap
 5. like      6. balance    7. honey     8. fox
 9. inquiry  10. truck     11. donkey   12. luxury
13. drive    14. bright    15. beauty   16. great
17. siren    18. absent    19. coast    20. age
21. eye      22. enjoy     23. story    24. awkward
---------------END LND CIPHER SEED-----------------

!!!YOU MUST WRITE DOWN THIS SEED TO BE ABLE TO RESTORE THE WALLET!!!
lnd successfully initialized!
```

Next we check the status of the bitcoin node. Run this command a few times, you should notice that the blocks and header numbers increasing. This means it is syncing with the main blockchain.
```sh
#docker exec -it lab-bitcoin-1 bitcoin-cli -getinfo
```

Next we check the status of the lightning node. The "block_height" number should be increasing which means it is syncing with the bitcoin node.
```sh
#docker exec -it lab-lightning-1 bash -c "lncli getinfo | grep block_height"
```

For the purpose of this lab, we want to store the password you used so that lightning can unlock the wallet automatically at startup. We also update the configuration file to now use this stored password and restart the node.  
```sh
#docker exec -it lab-lightning-1 bash -c "echo '<enter_password_you_used_earlier>' > /var/lib/gopath/lndwallet.txt"
#docker exec -it lab-lightning-1 bash -c "chmod 400 /var/lib/gopath/lndwallet.txt"
#docker exec -it lab-lightning-1 bash -c "sed -i 's/#wallet-unlock-password-file=/wallet-unlock-password-file=/' /var/lib/gopath/lnd.conf"
#docker restart lab-lightning-1
```
If you are planning to use this lightning node in production, I recommend you not store the password in a file but instead manually unlock the wallet everytime you start the node by running this command.
```sh
#docker exec -it lab-lightning-1 bash -c "lncli unlock"
```

Using the follwing commands, do a final check that the containers have a status of "Up" and are "(healthy)" and they are syncing with the main blockchain.
```sh
#docker ps -a
#docker exec -it lab-bitcoin-1 bitcoin-cli -getinfo | grep Blocks
#docker exec -it lab-lightning-1 bash -c "lncli getinfo | grep block_height"
```

Lastly, check that the bitcoin and lightning application ports are exposed on your host so that you can expose these to the internet if you like. If working, you will get a blank screen. Type `Ctrl+]` and then `quit` to return to a prompt.
```sh
#telnet localhost 9735
#telnet localhost 8333
```
If you want to learn more about allowing other bitcoin and lightning nodes to access your nodes, have a look at [Bitcoin Full Node documentation](https://bitcoin.org/en/full-node#network-configuration) and the [Lightning network daemon](https://docs.lightning.engineering/lightning-network-tools/lnd) documentation.

## Maintaining

The commands below help you to easily start, stop or tear down your lab. 

> Note that as of October 2022, syncing the full bitcoin mainnet consumes about **600GB** of disk space. 

Also, the backup of these volumes will be left for another day.

Use this command to stop the containers
```sh
#docker compose -p lab stop
```

Use this to start the containers
```sh
#docker compose -p lab start
```

Use this to tear down the lab.
```sh
#docker compose -p lab down --volumes
```