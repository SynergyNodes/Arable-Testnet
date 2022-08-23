
# Arable Cosmos Testnet bamboo_9000-1 Validator Node Setup

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
curl https://dl.google.com/go/go1.18.3.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -

```

```
# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

```

```
source $HOME/.profile

# check go version
go version

```

## Install Arable Testnet Node

```
git clone https://github.com/ArableProtocol/acrechain.git
cd acrechain
git checkout testnet_bamboo
make install
cd
acred version --long
```

You should get the following output:

```
name: acre
server_name: acred
version: ""
commit: 01482d6deddda2b0b4a399857857dc2a0dd38555
```

## Initialise your Validator node
```
#Choose a name for your validator and use it in place of “<moniker-name>” in the following command:
acred init <moniker-name> --chain-id bamboo_9000-1

#Example
acred init Synergy_Nodes --chain-id bamboo_9000-1
```

## Download genesis.json file
```
cd ~/.acred/config
rm genesis.json
cd

wget https://raw.githubusercontent.com/ArableProtocol/acrechain/testnet_bamboo/networks/bamboo/genesis.json -O $HOME/.acred/config/genesis.json
```

## Update configuration and peers list

```
cd

PEERS="44dd124ca34742245ad906f9f6ea377fae3af0cf@168.100.9.100:26656,6477921cdd4ba4503a1a2ff1f340c9d6a0e7b4a0@168.100.10.133:26656,9b53496211e75dbf33680b75e617830e874c8d93@168.100.8.9:26656,c55d79d6f76045ff7b68dc2bf6655348ebbfd795@168.100.8.60:26656,6d41af54405fa98073b178262ec9d083b3f12e67@46.4.81.204:36656"

sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.acred/config/config.toml
```

## Start the node and let it Sync
```
acred start

# Press Ctrl+c to exit

```

## Running the validator as a systemd unit
```
cd /etc/systemd/system
sudo nano acred.service
```
Copy the following content into ``acred.service`` and save it.

```
[Unit]
Description=Acred Daemon
#After=network.target
StartLimitInterval=350
StartLimitBurst=10

[Service]
Type=simple
User=node
ExecStart=/home/node/go/bin/acred start
Restart=on-abort
RestartSec=30

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=1048576
```

```
sudo systemctl daemon-reload
sudo systemctl enable acred

# Start the service
sudo systemctl start acred

# Stop the service
sudo systemctl stop acred

# Restart the service
sudo systemctl restart acred


# For Entire log
journalctl -t acred

# For Entire log reversed
journalctl -t acred -r

# Latest and continuous
journalctl -t acred -f
```

## Execute the folloiwng command to get the node id

```
acred tendermint show-node-id
```

## Create a Wallet for your Validator Node

Make sure to copy the 24 words Mnemonics Phrase, save it in a file and store it on a safe location.

```
acred keys add <wallet-name>

#Example
acred keys add my_wallet

```

## The following applys after the network goes live on 13th June 2022

Get some ACRE tokens to the above created wallet.


## Create and Register Your Validator Node
```
acred tx staking create-validator -y \
  --chain-id bamboo_9000-1 \
  --moniker <moniker-name> \
  --pubkey "$(acred tendermint show-validator)" \
  --amount 5000000000000000000uacre \
  --identity "<Keybase.io ID>" \
  --website "<website-address>" \
  --details "Some description" \
  --from <wallet-name> \
  --gas=300000 \
  --commission-rate=0.0 \
  --commission-max-rate=0.05 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation 1
  
#Example

acred tx staking create-validator -y \
  --chain-id bamboo_9000-1 \
  --moniker "Synergy Nodes" \
  --pubkey "$(acred tendermint show-validator)" \
  --amount 5000000000000000000uacre \
  --identity "D74433D32938F013" \
  --website "http://www.mywebsite.com" \
  --details "Some description" \
  --from my_wallet \
  --gas=300000 \
  --commission-rate=0.0 \
  --commission-max-rate=0.05 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation 1
  
```

## Get Validator Operator Address (Valoper Address)

Make sure to change ``<wallet-name>`` to correct values.

```
acred keys show <wallet-name> --bech val --output json | jq -r .address

#Example
acred keys show my_wallet --bech val --output json | jq -r .address
```

## Delegate ``ACRE`` to Your Node
```
acred tx staking delegate <validator-address> 1000000uacre --from <wallet-name> --chain-id bamboo_9000-1 -y

#Example
acred tx staking delegate acrevaloper1y4pfpkwpy6myskp7pne256k6smh2rjtay37kwc 1000000uacre --from my_wallet --chain-id bamboo_9000-1 -y
```
## Backup Validator node file

Take a backup of the following files after you have created and registered your validator node successfully.

```
/home/node/.acred/config/node_key.json
/home/node/.acred/config/priv_validator_key.json
/home/node/.acred/data/priv_validator_state.json
```

## Withdraw Rewards

Make sure to change ``<validator-operator-address>``, ``<wallet-name>`` to correct values.

```
acred tx distribution withdraw-rewards <validator-address> --from <wallet-name> --chain-id=bamboo_9000-1 -y

#Example
acred tx distribution withdraw-rewards acrevaloper1y4pfpkwpy6myskp7pne256k6smh2rjtay37kwc --from synergy --chain-id=bamboo_9000-1 -y
```

## Check Balance of an Address

```
acred query bank balances <wallet-address>

#Example:
acred query bank balances acre1y4pfpkwpy6myskp7pne256k6smh2rjtaaf0xv9
```
