# Bitcoin and lightning node using Docker 

## Introduction
This guide is to help you setup a bitcoin and lightning node on any computer that is capable of running docker. This should take you no longer than 30 minutes to get this setup. Although this guide is more suited for setting up a lab to let you learn about bitcoin and lightning, there is nothing stopping you from running these containers long term. These books on bitcoin and lightning are  worth exploring. 
* [Lightning Book](https://github.com/lnbook/lnbook)
* [Bitcoin Book](https://github.com/bitcoinbook/bitcoinbook)

**If you plan on putting any funds into the lightning wallet, I highly recommend you read and understand the lightning book before you do so, otherwise you risk loosing your funds.**

## Windows installation
Follow [these](https://www.howtogeek.com/249966/how-to-install-and-use-the-linux-bash-shell-on-windows-10/) instructions to install Ubuntu linux using WSL2. Next, you will need to have docker desktop installed and running. Visit the [Docker website](https://docs.docker.com/desktop/windows/wsl/) to install it.

## Linux installation
On linux use your package manager to install Git and Docker.

## Hardware requirements
In terms of hardware requirements, these are the recommendations:
* 2 or 4 core 64bit CPU.
* At least 4GB of RAM.
* A reliable internet connection.
* Note that as of October 2022, syncing the full bitcoin mainnet consumes about **500GB** of disk space. So a hard drive with at least 1TB of free space is recommended if you plan on running this node for a while.


## Download the files
Start a bash terminal and run this command.
```sh
#git clone https://github.com/findamak/bitcoin-lightning-node.git
```
Now run this command to change into that newly created directory.
```sh
#cd bitcoin-lightning-node
```

## Start the docker builds
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

Next we need to create a new lightning wallet for our lightning node. Follow the instructions. Note you need to remember the password for your wallet and I do not enter a passphrase to encrypt the cipher seed.
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

Now we change some default passwords stored in the configuration files of both the bitcoin and lightning nodes. Ensure you replace `<enter_a_few_words>` with your choice of any words and ensure they are the same across both commands. The command takes a hash of your words to generate a strong password.
```sh
#docker exec -it lab-lightning-1 bash -c 'mypass=$(echo -n "<enter_a_few_words>" | sha256sum | sed "s/\s.*$//") ; sed -i "s/change_me_please/${mypass}/" /var/lib/gopath/lnd.conf'
#docker exec -it lab-bitcoin-1 bash -c 'mypass=$(echo -n "<enter_a_few_words>" | sha256sum | sed "s/\s.*$//") ; sed -i "s/change_me_please/${mypass}/" /root/.bitcoin/bitcoin.conf ; sed -i "s/change_me_please/${mypass}/" /etc/bitcoin/bitcoin.conf'
```
Restart both nodes so the new passwords are put into effect.
```sh
#docker restart lab-bitcoin-1
#docker restart lab-lightning-1
```

Next we check the status of the bitcoin node. Run this command a few times, you should notice that the blocks and header numbers increasing. This means it is syncing with the main blockchain.
```sh
#docker exec -it lab-bitcoin-1 bitcoin-cli -getinfo
```

For the purpose of this lab, we want to store the password you used for your wallet (replace `wallet_password_you_used_earlier` wth your password) so that lightning can unlock the wallet automatically at startup.   
```sh
#docker exec -it lab-lightning-1 bash -c "echo 'wallet_password_you_used_earlier' > /var/lib/gopath/lndwallet.txt"
#docker exec -it lab-lightning-1 bash -c "chmod 400 /var/lib/gopath/lndwallet.txt"
```
We also update the configuration file to now use this stored password and restart the node.
```sh
#docker exec -it lab-lightning-1 bash -c "sed -i 's/#wallet-unlock-password-file=/wallet-unlock-password-file=/' /var/lib/gopath/lnd.conf"
#docker restart lab-lightning-1
```
If you are planning to use this lightning node in production, I recommend you not store the password in a file but instead manually unlock the wallet every time you start the node by running this command.
```sh
#docker exec -it lab-lightning-1 lncli unlock
```

Do a final check that the containers have a status of "Up" and are "(healthy)" and they are syncing with the main blockchain. i.e The block numbers are increasing as you run the below checks a few times.
```sh
#docker ps -a
#docker exec -it lab-bitcoin-1 bitcoin-cli -getinfo | grep Blocks
#docker exec -it lab-lightning-1 lncli getinfo | grep block_height
```

Lastly, check that the bitcoin and lightning application ports are exposed on your host so that you can expose these to the internet if you like. If working, you will get a blank screen. Type `Ctrl+]` and then `quit` to return to a prompt.
```sh
#telnet localhost 9735
#telnet localhost 8333
```
If you want to learn more about allowing other bitcoin and lightning nodes to access your nodes, have a look at [Bitcoin Full Node documentation](https://bitcoin.org/en/full-node#network-configuration) and the [Lightning network daemon](https://docs.lightning.engineering/lightning-network-tools/lnd) documentation. 

## Maintaining

The commands below help you to easily start, stop or tear down your lab. 

Use this command to stop the containers
```sh
#docker compose -p lab stop
```

Use this to start the containers
```sh
#docker compose -p lab start
```

Use this to tear down the lab. **This will destroy all data including your wallet**
```sh
#docker compose -p lab down --volumes
```

## Donations

If you found this guide useful, please consider donating some SATS via lightning to this wallet address makka@fountain.fm.