# Lightning node

Note the prerequisite to running a lightning node, is that you need to run a bitcoin node. To do this, I recommend you use docker compose which documentated in the readme in the top level of this repository. Below commands are if you want to manually start up a lightning node using the Dockerfile in this folder.  

```sh
#docker build -t lightning:latest .
#docker volume create lnd-vol
#docker run -d --name lightning -v lnd-vol:/var/lib/gopath/.lnd lightning:latest
```

Open another terminal and run this command:
```sh
#docker exec -it lightning lncli create

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

Next we store the password you created above into a file so the node can startup automatically.
```sh
#docker exec -it lightning bash -c "echo '<enter_password_you_used_earlier>' > /var/lib/gopath/lndwallet.txt"
#docker exec -it lightning bash -c "sed -i 's/#wallet-unlock-password-file=/wallet-unlock-password-file=/' /var/lib/gopath/lnd.conf"
```

Check that the node is running and syncing with the bitcoin node. The number should be increasing.
```sh
#docker exec -it lightning bash -c "lncli getinfo | grep block_height"
```

To stop and delete the container.
```sh
#docker stop lightning
#docker rm lightning
```
To delete the volume.
```sh
#docker volume rm lnd-vol
```