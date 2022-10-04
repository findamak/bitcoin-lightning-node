FROM fedora:latest
WORKDIR /tmp
COPY bitcoin.conf bitcoin.rpc.conf ./
RUN dnf install bitcoin-core-server bitcoin-core-utils -y && \
    mv bitcoin.conf /etc/bitcoin && \
    mkdir /run/bitcoind && \
    mkdir /root/.bitcoin && \
    mv bitcoin.rpc.conf /root/.bitcoin/bitcoin.conf
EXPOSE 8333/tcp 
#ENTRYPOINT ["/bin/sh", "-c", "command 1; command 2"]
ENTRYPOINT [ "/bin/sh", "-c", "/usr/sbin/bitcoind -conf=/etc/bitcoin/bitcoin.conf -datadir=/var/lib/bitcoin -pid=/run/bitcoind/bitcoin.pid -debuglogfile=/var/log/bitcoin/debug.log" ]
