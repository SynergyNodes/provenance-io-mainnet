# Provenance.io Mainnet Validator Node Setup

![](https://www.synergynodes.com/youtube/Provenance_Mainnet_Validator_Node.jpg)

## Install Ubuntu 20.04 on a new server and login as root

## Install ``ufw`` firewall and configure the firewall

```
apt-get update
apt-get install ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow 22
ufw allow 26656
ufw enable
```

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
sudo apt install pkg-config build-essential libssl-dev curl jq git libleveldb-dev -y
sudo apt-get install manpages-dev -y

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
cd
```

## Initialise your Validator node
```
#Choose a name for your validator and use it in place of “Provenance_Node” in the following command:
provenanced init Provenance_Node --chain-id pio-mainnet-1
```
## Download latest Snapshot
```
wget https://storage.googleapis.com/provenance-mainnet-backups/latest-data-indexed.tar.gz
```
## Move the downloaded Snapshot to ``.provenanced`` folder and unzip the file
```
mv latest-data-indexed.tar.gz ~/.provenanced
cd ~/.provenanced
rm -rf data
tar -zxvf latest-data-indexed.tar.gz
```

## Download genesis.json, app.toml, config.toml files
```
cd .provenanced/config
rm genesis.json
rm app.toml
rm config.toml

wget https://raw.githubusercontent.com/SynergyNodes/provenance-io-mainnet/main/genesis.json
wget https://raw.githubusercontent.com/SynergyNodes/provenance-io-mainnet/main/app.toml
wget https://raw.githubusercontent.com/SynergyNodes/provenance-io-mainnet/main/config.toml
```
## Start the node and let it Sync
```
provenanced start --home /home/node/.provenanced --p2p.seeds 4bd2fb0ae5a123f1db325960836004f980ee09b4@seed-0.provenance.io:26656, 048b991204d7aac7209229cbe457f622eed96e5d@seed-1.provenance.io:26656 --x-crisis-skip-assert-invariants
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
ExecStart=/home/node/go/bin/provenanced start --home /home/node/.provenanced --x-crisis-skip-assert-invariants
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
sudo systemctl start provenanced

# Stop the service
sudo systemctl stop provenanced

# Restart the service
sudo systemctl restart provenanced


# For Entire log
journalctl -t provenanced

# For Entire log reversed
journalctl -t provenanced -r

# Latest and continuous
journalctl -t provenanced -f
```

## Create a Wallet for your Validator Node

Make sure to copy the 24 words Mnemonics Phrase, save it in a file and store it on a safe location.

```
provenanced keys add validator --home /home/node/.provenanced
```

Send some HASH to this wallet.


## Create and Register Your Validator Node
```
provenanced tx staking create-validator \
  --chain-id pio-mainnet-1 \
  --home /home/node/.provenanced \
  --moniker "Provenance_Node" \
  --pubkey "$(provenanced tendermint show-validator --home /home/node/.provenanced)" \
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
provenanced tx staking delegate <validator address> 10000000000nhash --from validator --fees 381000000nhash --broadcast-mode block --chain-id pio-mainnet-1 --home /home/node/.provenanced -y
```
## Backup Validator node file

Take a backup of the following files after you have created and registered your validator node successfully.

```
/home/node/.provenanced/config/node_key.json
/home/node/.provenanced/config/priv_validator_key.json
/home/node/.provenanced/data/priv_validator_state.json
```
## Upgrade Node to v1.8.2

```
cd provenance
git fetch
git pull
git checkout v1.8.2
make install
cd
provenanced version
# should return v1.8.2

# restart the node
sudo systemctl restart provenanced

# Wait for few minutes and check the status
provenanced status
```




