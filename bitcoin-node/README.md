# Bitcoin node

Use these commands below to build and run a bitcoin node in docker based on the Dockerfile located in this directory.

```sh
docker build -t bitcoin:latest .
docker volume create bitcoin-vol
docker run -d --name bitcoin -v bitcoin-vol:/var/lib/bitcoin bitcoin:latest 
docker exec -it bitcoin bitcoin-cli -getinfo
```

To stop and delete the container.
```sh
docker stop bitcoin
docker rm bitcoin
```

To delete the volume.
```sh
#docker volume rm bitcoin-vol
```