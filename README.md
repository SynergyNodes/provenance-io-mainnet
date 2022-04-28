### Provenance.io Mainnet Validator Node Setup

## Install Ubuntu 20.04 on a new server and login as root
## Create a new User

```
# add user
adduser node

# add user to sudoers
usermod -aG sudo node

# login as user
su - node
```

## Install Prerequisites

```
sudo apt update
sudo apt install pkg-config build-essential libssl-dev curl jq
sudo apt-get install manpages-dev

# install go
curl https://dl.google.com/go/go1.17.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile

# check go version
go version

# check gcc version
gcc --version
```

## Install Provenance Node

```
export PIO_HOME=~/.provenanced
git clone https://github.com/provenance-io/provenance.git
cd provenance
git checkout tags/v1.8.0 -b v1.8.0
make clean
make install
```

## Initialise your Validator node
```
#Choose a name for your validator and use it in place of “Test_Node” in the following command:
provenanced init Test_Node --chain-id pio-mainnet-1
```
## Download latest Snapshot
```
wget https://storage.googleapis.com/provenance-mainnet-backups/latest-data.tar.gz
```
## Unzip the downloaded Snapshot and transfer data to .provenanced folder
```
mv latest-data.tar.gz .provenanced
rm -rf data
tar -zxvf latest-data.tar.gz
```

## Download genesis.json, app.toml, config.toml files
```
wget https://raw.githubusercontent.com/provenance-io/mainnet/main/pio-mainnet-1/genesis.json
curl https://rpc.provenance.io/genesis > genesis.json
cat genesis.json | jq -r | sha256sum

wget https://raw.githubusercontent.com/provenance-io/mainnet/main/pio-mainnet-1/node-app.toml > app.toml
wget https://raw.githubusercontent.com/provenance-io/mainnet/main/pio-mainnet-1/config.toml
```
## Start the node and let it Sync
```
provenanced start --p2p.seeds 4bd2fb0ae5a123f1db325960836004f980ee09b4@seed-0.provenance.io:26656, 048b991204d7aac7209229cbe457f622eed96e5d@seed-1.provenance.io:26656 --x-crisis-skip-assert-invariants
```

## Running the validator as a systemd unit
```
cd /etc/systemd/system
sudo nano provenanced.service
```
Copy the following content into provenanced.service and save it.
```
[Unit]
Description=Provenance Daemon
#After=network.target
StartLimitInterval=350
StartLimitBurst=10

[Service]
Type=simple
User=node
ExecStart=/home/node/go/bin/provenanced start --home /home/pro/.provenanced --x-crisis-skip-assert-invariants
Restart=on-abort
RestartSec=30

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=1048576
```

```
sudo systemctl daemon-reload
sudo systemctl enable provenanced

# Start the service
sudo systemctl start terrad

# Stop the service
sudo systemctl stop terrad

# Restart the service
sudo systemctl restart terrad


# For Entire log
journalctl -t terrad

# For Entire log reversed
journalctl -t terrad -r

# Latest and continuous
journalctl -t terrad -f
```

## Create a Wallet for your Validator Node
```
provenanced keys add synergy2 --recover  --home /home/pro/.provenanced
```

## Create and Register Your Validator Node
```
provenanced tx staking create-validator \
  --chain-id pio-mainnet-1 \
  --home /home/pro/.provenanced \
  --moniker "Test_Node" \
  --pubkey "$(provenanced tendermint show-validator --home /home/pro/.provenanced)" \
  --amount 10000000000nhash \
  --identity "<Keybase.io ID>" \
  --details "Some description" \
  --from validator \
  --fees 381000000nhash \
  --commission-rate=0.0 \
  --commission-max-rate=0.05 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation 1 \
  --broadcast-mode block
```

## Delegate HASH to Your Node
```
provenanced tx staking delegate <validator address> 10000000000nhash --from validator --fees 381000000nhash --broadcast-mode block --chain-id pio-mainnet-1 --home /home/pro/.provenanced -y
```
## Backup Validator node file


