FROM fedora:latest
ENV GOPATH /var/lib/gopath
ENV PATH $PATH:$GOPATH/bin
WORKDIR $GOPATH
COPY lnd.conf ./
RUN dnf install git golang make -y && \
    mkdir $GOPATH/.lnd && \
    git clone https://github.com/lightningnetwork/lnd && \
    cd lnd && \
    make install && \
    ln -s $GOPATH/.lnd /root/.lnd
EXPOSE 9735/tcp 
#ENTRYPOINT ["/bin/sh", "-c", "command 1; command 2"]
ENTRYPOINT [ "/bin/sh", "-c", "/var/lib/gopath/bin/lnd --lnddir=/var/lib/gopath/.lnd --configfile=/var/lib/gopath/lnd.conf" ]
