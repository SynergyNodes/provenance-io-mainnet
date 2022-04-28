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
