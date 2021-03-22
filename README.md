# 0chain-Blobber-Tutorial


Prerequisite:
Only use if you are connecting via ssh remotely
Guide was built using Command Prompt via Windows, interacting with a server hosted by Teraswitch.

Basis for this Tutorial:
https://zcdn.uk/tutorials/setting-up-a-blobber-on-0chain-network/ 

Part 1: Setting up CLI tools, make, golang & building CLI tools

Log In:
If you are using Windows, open Command Prompt (search command prompt)
Left click in the top left corner, go to Properties, check QuickEdit Mode and Use Ctrl+Shift+C/V as Copy/Paste
Font tab will adjust font size and type if you want something different
If you are using Mac, open Terminal (command + spacebar)
Enter the following command:
	ssh <username>@<server ip address>
When prompted, enter your password
	* Note - your password will not appear but will be entered as you type

Access Root:
	sudo -i
When pcdrompted, enter your password
	* Note - your password will not appear but will be entered as you type

Update/upgrade Ubuntu as a house cleaning activity to begin:
	apt update
apt upgrade

Install Build Tools:
	apt install build-essential
apt install git nano jq htop
Install Go:
wget https://golang.org/dl/go1.16.2.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.16.2.linux-amd64.tar.gz
rm go1.16.2.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version

Create ZCN Folder:
mkdir ~/.zcn
0Box CLI:
git clone https://github.com/0chain/zboxcli.git
	cd zboxcli 
	make install
	cp zbox ~/.zcn
	cd ..
0Wallet CLI:
	git clone https://github.com/0chain/zwalletcli.git
cd zwalletcli
git checkout d929b090dfd694aaeda86731e469c1a6c23f58c1
	make install
	cp zwallet ~/.zcn
	cd ..
Get Network Config:
cp zboxcli/network/one.yaml ~/.zcn/config.yaml
Get Balance/Create Wallets:
Go to ZCN directory: 
cd ~/.zcn

getbalance will show the token balance of the default wallet, ‘wallet.json’. Initially, you will not have a balance since you have not created the wallet names ‘wallet’ yet. It will be automatically created through the getbalance command

./zwallet getbalance 
    
walletname replaced with desired wallet name:
use the following names: delegate.json, wallet.json, blobber1.json

./zwallet getbalance --wallet <walletname>.json 

this will return the balance in the wallet named blobber1:


./zwallet getbalance --wallet blobber1.json

Faucet tokens: (must be in ~/.zcn folder)
This will faucet one (1) token into the default wallet, ‘wallet.json’ for each time you enter the command.

./zwallet faucet --input test --methodName pour
This will faucet one (1) token into the wallet, ‘<walletname>.json’ for each time you enter the command.

./zwallet faucet --input test --methodName pour --wallet <walletname>.json 

Faucet Multiple Tokens: (must be in ~/.zcn folder)
{1..50} will faucet 50 tokens, you can adjust the last number (50) to whatever you want. I suggest 10 as a minimum.

for f in {1..50} ; do ./zwallet faucet --wallet <walletname>.json --methodName pour --input test ; done 
for f in {1..20} ; do ./zwallet faucet --wallet blobber1.json --methodName pour --input test ; done 
for f in {1..20} ; do ./zwallet faucet --wallet delegate.json --methodName pour --input test ; done 
for f in {1..20} ; do ./zwallet faucet --wallet wallet.json --methodName pour --input test ; done 
Part 2: Install Docker tools, Fetch Blobber rep

Setup Docker
apt install docker.io -y
systemctl enable --now docker
docker --version
Installs docker and returns its version
Setup Docker-Compose
curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version

Installs docker-compose and returns its version

Fetch Blobber Rep:
Navigate to root:
	cd ~
git clone https://github.com/0chain/blobber.git

Part 3: Configure Blobber and Delegate
Navigate to:
cd ~/.zcn
If you haven’t already, create the following wallets:
blobber.json 
delegate.json 
wallet.json 
Use the faucet command above and faucet tokens into each wallet.

Extract and display the values of your delegate id, blobber id and blobber private and public keys
DELID=$(jq -r .client_id delegate.json)
B1ID=$(jq -r .client_id blobber1.json)
B1PRIVKEY=$(jq -r .keys[].private_key blobber1.json)
B1PUBKEY=$(jq -r .keys[].public_key blobber1.json)
echo "DELID" $DELID
echo "B1ID" $B1ID
echo "B1PUBKEY" $B1PUBKEY
echo "B1PRIVKEY" $B1PRIVKEY

Copy/paste the above IDs into a Word document or notepad, you will need them in subsequent steps. Do not give out your Private key (B1PRIVKEY)
I like to reduce my copy/paste to the following:
DELID xxxxxxxxxxxxxxxxxxxxx
B1ID xxxxxxxxxxxxxxxx
B1PUBKEY xxxxxxxxxxxxxxxxxxxxxxxx
B1PRIVKEY xxxxxxxxxxxxxxx
Definitions:
DELID = delegate.json ID
B1ID = blobber1.json ID
B1PRIV = blobber1 private key
B1PUB = blobber1 public key)

Navigate to:
	cd ~/blobber/docker.local

Run:
 	nano b0docker-compose.yml
nano  allows you to edit files
ls allows you to see what is in your current directory
Change localhost to your URL, do not include ‘https://’

Run:
nano p0docker-compose.yml
Change localhost (two locations) to your URL, do not include ‘https://’
Navigate to:
cd ~/blobber/config/
For both files (0chain_blobber.yaml and 0Chain_validator.yaml):
	nano 0chain_blobber.yaml 
	nano 0chain_validator.yaml
Change capacity to: 1073741824000 (add 3 0’s at the end)
Change block_worker from https://198.18.0.98:9091 to https://one.devnet-0chain.net/dns
Set delegate price (this can be handled at mainnet)
Increase handlers: rate_limit from 10 to higher number 50 or 100
Configure read_price, write_price, capacity (this can be handled at mainnet)
Set delegate_wallet to the delegate wallet (DELID) you will be using to fund blobber wallet. DELID can be found in your Word document or Notepad, where you save it from the step on the previous page.
Navigate to:
cd ~/blobber/docker.local/keys_config
nano b0bnode1_keys.txt

Enter you Public key and Private key in the order below:
	<PUBKEY>
	<PRIVKEY>
	Localhost
	5051


Setup Config, testnet0, and build blobber(s)
Navigate to:
cd ~/blobber
 
Run:
./docker.local/bin/blobber.init.setup.sh
 
docker network create --driver=bridge --subnet=198.18.0.0/15 --gateway=198.18.0.255 testnet0

./docker.local/bin/build.blobber.sh


Navigate to blobber1 folder:
Run:
	cd docker.local/blobber1


Start the Blobber!
../bin/blobber.start_bls.sh


../bin/p0blobber.start.sh
(Alternatively, there is a p0blobber.start.sh option for remote image)
 
 
Step 4: Service Provider Lock (Delegate) Tokens

Navigate to:
Cd ~/.zcn
If you haven’t already, faucet tokens to delegate.json and wallet.json wallets. See above, step 1, Faucet Multiple Tokens.
Lock Tokens to Service Provider:
./zbox sp-lock --wallet delegate.json --blobber_id $B1ID --tokens 50
Insert your blobber id where it says: $B1ID
Check Locked Tokens of Blobber:
./zbox sp-info --wallet blobber1.json
 
 
 
Create Allocation
./zbox newallocation --lock xx
For now, all you need to do is replace xx with a 1.
 
List Allocation
./zbox listallocations
This command will return a list of the allocations created. You will need your allocation ID in order to upload a file in the next step.
 
Upload File:
./zbox upload --allocation <allocationid> --remotepath /file --localpath file
During testing, files need to be uploaded from the ubuntu server itself. A few examples are below:
Upload zbox:
./zbox upload --allocation $(cat allocation.txt) --remotepath /zbox --localpath zbox --commit
Upload zwallet:
./zbox upload --allocation $(cat allocation.txt) --remotepath /zwallet --localpath zwallet --commit
 
Upload a File to your Blobber:
Navigate to:
~/.zcn
nano config.yaml
Add and uncomment (#) your blobber’s domain under ‘preferred blobbers’:

Close and Save (ctrl+x , y)
Restart docker containers to enable preferred blobbers
docker restart $(docker container ls -aq)
Restart blobber:
Navigate to: blobber/docker.local/blobber
Run:
../bin/blobber.start_bls.sh
Get new allocation:
Return to: cd ~/.zcn
Run:
./zbox newallocation --lock 1
 
Run the Upload zbox/zwallet File commands from above.
 
Step 5: Configure SSL (https)
Navigate to: blobber/https
cd ~/.zcn/blobber/https
Edit docker-compose.yml:
nano docker-compose.yml
Replace <your_email>, <your_domain> with your email and domain. Ensure you have an A type record for your domain and ip address set up with your domain name service provider.

Deploy nginx and certbot using the following command:
docker-compose up -d
Check certbot logs and see if certificate is generated. You will find “Congratulations! Your certificate and chain have been saved at: /etc/letsencrypt/live//fullchain.pem” in the logs if the certificate is generated properly.
docker logs -f https_certbot_1
Edit /conf.d/nginx.conf to uncomment required locations in config for port 80. Uncomment all lines in server config for port 443 and comment locations which are not required. Don’t forget to replace <your_domain> with your domain, 3 different places
 
nano ./conf.d/nginx.conf
	
Restart docker compose and you will be able to access blobbers over https.
docker-compose restart
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
Appendix
Commonly used Commands
ls = returns a list of files within the directory/folder you are in
cd = change directory 
cd .. = returns you to the previous folder you were in, similar to the ‘Back’ button, or move up in folder hierarchy

nano <filename> = “edit” filename
rm = remove
rm <directory/folder> -r = removes all contents of named directory
 
Docker.io Commands
docker ps = returns container IDs, status, ports of your blobber
docker restart $(docker container ls -aq) = restarts all containers
docker-compose restart = restarts docker-compose
 
Restart blobber:
Navigate to 
blobber/docker.local/blobber
Run:
../bin/blobber.start_bls.sh
 
 
 
 
 
 
 
 
