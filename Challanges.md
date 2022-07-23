# Stake Wars iii
Welcome to Stake Wars: Episode III A New Validator\
In Validator We Trust\
Mission I - VI

![image](https://user-images.githubusercontent.com/46390405/180593004-b1c77efc-4bc5-4ae2-b215-398d90cd979c.png)

## Recommended Hardware Requirements
* 4x CPUs; the faster clock speed the better
* 8GB RAM
* 500GB of storage (SSD or NVME)
* Permanent Internet connection (traffic will be minimal during devnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Installation
Update, Download Pacakge and Install Python
```
sudo apt update && sudo apt upgrade -y

sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker protobuf-compiler libssl-dev pkg-config clang llvm cargo clang build-essential make

sudo apt install python3-pip 

USER_BASE_BIN=$(python3 -m site --user-base)/bin export PATH="$USER_BASE_BIN:$PATH"
```
Install Node JS and NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash - 
sudo apt install build-essential nodejs 
PATH="$PATH"
```
Install Near CLI
```
sudo npm install -g near-cli
```
Setup Environment
> Be aware that you should input these commands anytime you open a new session! Otherwise testnet environment will be used!
```
export NEAR_ENV=shardnet 
echo ‘export NEAR_ENV=shardnet’ >> ~/.bashrc
```
Install Cargo and Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```
Download and Build Binary
```
git clone https://github.com/near/nearcore 
cd nearcore 
git fetch 
git checkout 8448ad1ebf27731a43397686103aa5277e7f2fcf
cargo build -p neard --release --features shardnet
```
Check Version 
```
~/nearcore/target/release/neard --version
```
Initialize the working directory, delete old files and download new config file and new genesis.json. Since the network was hard forked recently, there is no need to download the snapshot.
```
~/nearcore/target/release/neard --home ~/.near init --chain-id shardnet --download-genesis 
rm ~/.near/config.json ~/.near/genesis.json
```
```
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json 
wget -O ~/.near/genesis.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```
```
mv ~/.near/data ~/.near/data-fork1 && rm ~/.near/config.json && wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json && rm ~/.near/genesis.json && wget -O ~/.near/genesis.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```
Now you will need to create a wallet https://wallet.shardnet.near.org/ WARNING! If you had a wallet previously to the hard fork, you may need to recreate it! Forget the old mnemonic and do everything from scratch.

After that, write in the CLI
```
near login
```
![image](https://user-images.githubusercontent.com/46390405/180593913-9b6d92e7-5392-47bc-a8e6-c362c3047bac.png)

Copy link to your browser and confirm connect wallet

![image](https://user-images.githubusercontent.com/46390405/180593970-232424e6-c379-4bf3-9a91-9c236bc65522.png)

if it looks like the one above, don't panic, because it's already connected successfully

![image](https://user-images.githubusercontent.com/46390405/180594030-fd41ec43-566e-416f-a283-1c9dd453eb94.png)

Enter Your Address

![image](https://user-images.githubusercontent.com/46390405/180594061-2afe871b-09ea-4fdd-afb5-d36b85e27bee.png)

Done Connect your Wallet

### Next Step

See the authorisation link and copy it into the browser window where you wallet is opened. After connecting a wallet, right the wallet name ("accountId".shardnet.near) in the CLI and confirm.

Copy your wallet json and make some changes
```
cd ~/.near-credentials/shardnet/
cp wallet.json ~/.near/validator_key.json
nano validator_key.json
```
Change the account ID and Private Key parameter accordingly, then exit nano editor.
```
{ 
"account_id": "xx.factory.shardnet.near", 
"public_key":"ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G", "secret_key": "ed25519:****" 
}
```
Create a service file (one command)
```
sudo tee /etc/systemd/system/neard.service > /dev/null <<EOF 
[Unit] 
Description=NEARd Daemon Service 
[Service] 
Type=simple 
User=$USER
#Group=near 
WorkingDirectory=$HOME/.near
ExecStart=$HOME/nearcore/target/release/neard run 
Restart=on-failure 
RestartSec=30 
KillSignal=SIGINT 
TimeoutStopSec=45 
KillMode=mixed 
[Install] 
WantedBy=multi-user.target 
EOF
```
Now start the service and see logs, that everything works fine. You should see it downloading headers firstly and the blocks. Waiting for full sync.
```
sudo systemctl daemon-reload 
sudo systemctl enable neard 
sudo systemctl start neard
journalctl -f -u neard
```
```
sudo apt install ccze

journalctl -n 100 -f -u neard | ccze -A
```
![image](https://user-images.githubusercontent.com/46390405/180594278-0760ec9c-bb84-452b-b7b8-eb5384d727eb.png)

> Wait until Download and Sync

Now let’s deploy a contract of our staking pool with 30 NEAR staked!
```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
```
> Change "pool id", "accountId", "public key", "accountId" parameters here!

My example for this
```
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "demigohu", "owner_id": "demigohu.shardnet.near", "stake_public_key": "ed25519:3kmizJ6JowpDKSQzu52XhVNtDSCaK4fHrtxVnCk73HEx", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="demigohu.shardnet.near" --amount=30 --gas=300000000000000
```
If everything is fine, you should see yourself in near proposals command. Let’s look at the seat price in the bottom of near proposals command. And then we will need to stake. Remember to set environmentals for shardnet!
```
near proposals
```
![image](https://user-images.githubusercontent.com/46390405/180594454-1ac784e7-7013-4df5-91b4-aca53325888e.png)

Change parametres for staking accordingly!
```
near call demigohu.factory.shardnet.near  deposit_and_stake --amount 1200 --accountId demigohu.shardnet.near --gas=300000000000000
```
In the few epochs, you will be able to see yourself in the explorer and by typing
```
near validators current 
near validators next
```
Finally, we can set the ping (every 5 minutes)
```
mkdir $HOME/nearcore/logs
```
```
nano $HOME/nearcore/scripts/ping.sh
```
Edit "PoolID" && "AccountID"
```
#!/bin/sh
# Ping call to renew Proposal added to crontab

export NEAR_ENV=shardnet
export LOGS=$HOME/nearcore/logs
export POOLID="PoolID"
export ACCOUNTID="AccountID"

echo "---" >> $LOGS/all.log
date >> $LOGS/all.log
near call $POOLID.factory.shardnet.near ping '{}' --accountId $ACCOUNTID.shardnet.near --gas=300000000000000 >> $LOGS/all.log
near proposals | grep $POOLID >> $LOGS/all.log
near validators current | grep $POOLID >> $LOGS/all.log
near validators next | grep $POOLID >> $LOGS/all.log
```
```
contrab -e
```
change with this and remove the #
```
*/5 * * * * sh $HOME/nearcore/scripts/ping.sh
```
check logs
```
cat $HOME/nearcore/logs/all.log
```
If do you see this you DONE and wait for Next Challenge!

![image](https://user-images.githubusercontent.com/46390405/180594677-5ebbb9ec-fe04-4c44-b1fd-e2646aae4d65.png)

