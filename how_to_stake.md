
## Prerequisites  
- Server: Ubuntu v20.04 (LTS) x64 [Install Guide](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview) 
- Bidao Execution Client [Download](https://github.com/Bidao-Smart-Chain/nethermind)
- Bidao Consensus Client [Download](https://github.com/Bidao-Smart-Chain/teku)
- Crypto wallet: MetaMask [Get Metamask](https://metamask.io/)
  
## Support  
For technical support please join the official [Bidao Telegram channel](https://t.me/bidaoofficial).  
  
## Overview  
The guide consists of the following steps:  
  
1. Generate the staking deposit data and validator keystore.  
2. Prepare the Ubuntu server by installing updates and configuring the firewall.  
3. Set up the Bidao Execution Client node and synchronize it with the Ascend Devnet.  
4. Set up the Bidao Consensus Client and synchronize it with the Ascend Devnet.  
5. Activate the staking Validator by depositing 960 BISC.  
  
## 1. Generate Staking Data  
  
To activate a validator on the Ascend Devnet, a deposit of 960 BISC is required for each. For example, if you intend to run two validators, you need 1920 BISC (960 x 2), along with some additional funds to account for gas fees.  
  
You have to first download the latest version of the Bidao Staking Deposit CLI from [here](https://github.com/Bidao-Smart-Chain/staking-deposit-cli/releases/tag/v1.0.0).  For example like this:

    curl -LO https://github.com/Bidao-Smart-Chain/staking-deposit-cli/releases/download/v1.0.0/bidao-staking_deposit-cli-1_0_0-linux-amd64.tar.gz

  
The Staking Deposit CLI is used to generate the staking validator keystore and deposit file. Therefore, it is crucial to handle the mnemonic securely to prevent any potential exposure (e.g. by using an air gapped machine).  

Once you have extracted the archive (with e.g. `tar xvf bidao-staking_deposit-cli-1_0_0-linux-amd64.tar.gz`), you can execute the tool by using the following command:  
  

    sudo ./deposit new-mnemonic --num_validators <NumberOfValidators> --chain bidao --execution_address <YourWithdrawalAddress>  

By using the flag `--execution_address`, you can designate an Bidao Smart Chain address to which any staking rewards exceeding 960 BISC will be automatically withdrawn to. This same address will also receive your 960 BISC upon exiting the validator.  
  
The CLI flag `--num_validators` is used to specify the number of validators that will be generated for staking.  
  
## 2. Update the Server  
It is highly recommended to ensure that the operating system is running the latest version of software and receiving frequent security updates to maintain optimal security and performance. By regularly updating the system, any known vulnerabilities and bugs can be patched, and newly added features and improvements can be utilized. Neglecting to update the system may result in a less stable and more vulnerable environment, potentially leading to security breaches or system failures.  
  
To accomplish this, please use the following commands:  
  

    sudo apt -y update && sudo apt -y upgrade  
    sudo apt dist-upgrade && sudo apt autoremove  
    sudo reboot  

Now, you should install UFW, which stands for Uncomplicated Firewall, a program that is commonly pre-installed. To confirm its installation, execute the following command:  
  

    sudo apt install ufw  

Now execute the following commands:  
  

    sudo ufw default deny incoming  
    sudo ufw default allow outgoing  
    sudo ufw allow ssh # only if you're connected to the server via ssh
    sudo ufw allow 9000  
    sudo ufw allow 30303  
    sudo ufw enable  

  
Here is a brief explanation of each of the listed commands:  
  
1. `sudo ufw default deny incoming`: This command sets the default behavior for incoming connections to be denied, except for those explicitly allowed by subsequent rules.  
  
2. `sudo ufw default allow outgoing`: This command sets the default behavior for outgoing connections to be allowed, except for those explicitly denied by subsequent rules.  
  
3. `sudo ufw allow ssh`: This command generally allows incoming traffic on port 22 for TCP protocol, which is commonly used for secure shell (SSH) connections.  
  
4. `sudo ufw allow 9000`: This command allows incoming traffic on port 9000, which is used for the validator client.  
  
5. `sudo ufw allow 30303`: This command allows incoming traffic on port 30303, which is used for Bidao peer-to-peer networking.  
6. `sudo ufw enable`: This is used to activate the firewall, applying the defined rules.  
  
To check the status of the firewall run:  
  

    sudo ufw status numbered  

  
## 3. Execution Client Configuration  
First download the latest release of the [Bidao Nethermind](https://github.com/Bidao-Smart-Chain/nethermind/releases/tag/v1.0.0) execution client. e.g. with:

    curl -LO https://github.com/Bidao-Smart-Chain/nethermind/releases/download/v1.0.0/bidao-nethermind-1.0.0-linux-x64.zip
  
Then extract the archive and install the required dependencies:  
  

    sudo apt-get install -y unzip  
    unzip bidao-nethermind-1.0.0-linux-x64.zip -d nethermind  
    sudo cp -a nethermind/bidao-nethermind-1.0.0-linux-x64 /usr/local/bin/nethermind  
    rm bidao-nethermind-1.0.0-linux-x64.zip  
    rm -r nethermind  
    sudo apt-get update  
    sudo apt-get install libsnappy-dev libc6-dev libc6 unzip -y  

To run Bidao-Nethermind in the background, you'll need to configure it as a background service and create an account specifically for it. This type of account won't be able to log into the server.  
  
You can create the account by running the following command:  
  
`sudo useradd --no-create-home --shell /bin/false nethermind`  
  
Next, you'll need to create a data directory to store the Bidao blockchain data. Use this command to create the directory:  
  
`sudo mkdir -p /var/lib/nethermind`  
  
To enable the nethermind user account to modify the data in this directories, you'll need to set the directory permissions as follows:  
  
`sudo chown -R nethermind:nethermind /var/lib/nethermind`  
Now create a systemd service config file using nano.  
  
sudo nano /etc/systemd/system/nethermind.service  
  
Copy and paste the following service configuration into the file.  
  

    [Unit]  
    Description=Bidao Nethermind Execution Client (Ascend Devnet)  
    After=network.target  
    Wants=network.target
    
    [Service]  
    User=nethermind  
    Group=nethermind  
    Type=simple  
    Restart=always  
    RestartSec=5  
    WorkingDirectory=/var/lib/nethermind  
    Environment="DOTNET_BUNDLE_EXTRACT_BASE_DIR=/var/lib/nethermind"  
    ExecStart=/usr/local/bin/nethermind/Nethermind.Runner \  
    -c bidao \ 
    --datadir /var/lib/nethermind \ 
    --JsonRpc.JwtSecretFile /var/lib/jwtsecret/jwt.hex
    
    [Install]  
    WantedBy=default.target  

To save and exit nano, press <CTRL> + X, followed by Y, and then <ENTER>.  
  
Once you've made the changes, you'll need to reload systemd to apply them and start the Nethermind service. To do this, run the following commands:  
  

    sudo systemctl daemon-reload  
    sudo systemctl start nethermind  

  
To confirm that the service is running correctly, check its status by running:  
  
`sudo systemctl status nethermind`  
  
Once you've confirmed that the service is running correctly, you can quit the status screen by pressing Q.  
  
To ensure that the Nethermind service starts automatically on reboot, enable it by running this command:  
  
`sudo systemctl enable nethermind`  
  
## 4. Consensus Client Configuration  
First download the latest release of the [Bidao Teku](https://github.com/Bidao-Smart-Chain/teku/releases/tag/v1.0.0) consensus client. e.g. with:

    curl -LO https://github.com/Bidao-Smart-Chain/teku/releases/download/v1.0.0/bidao-teku-1.0.0.zip
  
Then extract the archive and install the required dependencies:  
  

    unzip bidao-teku-1.0.0.zip -d teku  
    sudo cp -a teku/bidao-teku-1.0.0 /usr/local/bin/teku  
    rm bidao-teku-1.0.0.zip  
    rm -r teku  
    sudo apt-get update  
    sudo apt install -y default-jre  

  
Now you need to copy the in step 1 created validator keystore files to the server. For this create a new folder:  
  

    sudo mkdir -p /var/lib/teku  
    sudo mkdir -p /var/lib/teku/validator_keys  

  
Now create a file containing your in step 1 chosen password using:  
  

    nano password.txt  

  
The following command copies the contents of "password.txt" to a new file for each validator key file found in the "/var/lib/teku/validator_keys" directory. The new file will have the same name as the validator key file, but with a ".txt" extension instead of ".json". This is used to decrypt each validator keystore files.  
  

    for keyFile in /var/lib/teku/validator_keys/*; do cp password.txt "/$(echo "$keyFile" | sed -e 's/\/\(.*\)\.json/\1/').txt"; done
    rm password.txt

or if you need root privileges for this run:
  
    for keyFile in /var/lib/teku/validator_keys/*; do sudo cp password.txt "/$(echo "$keyFile" | sed -e 's/\/\(.*\)\.json/\1/').txt"; done
    rm password.txt
  
To also run Bidao-Teku in the background, you'll need to configure it as a background service and create an account for it and give it permission modify the files in /var/lib/teku.  
  

    sudo useradd --no-create-home --shell /bin/false teku  
    sudo chown -R teku:teku /var/lib/teku  

  
Now create a systemd service config file using nano.  
  

    sudo nano /etc/systemd/system/teku.service  

  
Copy and paste the following service configuration into the file.  Also replace <FeeRecipientAddress> and <yourgraffiti> with your fee recipient (most likely same as your withdrawal address) and your desired graffiti (can be any string or left empty).
  

    [Unit]  
    Description=Bidao Teku Consensus Client (Ascend Devnet)  
    Wants=network-online.target  
    After=network-online.target
    
    [Service]  
    User=teku  
    Group=teku  
    Type=simple  
    Restart=always  
    RestartSec=5  
    Environment="JAVA_OPTS=-Xmx5g"  
    Environment="TEKU_OPTS=-XX:-HeapDumpOnOutOfMemoryError"  
    ExecStart=/usr/local/bin/teku/bin/teku \  
    --network=bidao \  
    --data-path=/var/lib/teku \  
    --validator-keys=/var/lib/teku/validator_keys:/var/lib/teku/validator_keys \  
    --ee-endpoint=http://127.0.0.1:8551 \  
    --ee-jwt-secret-file=/var/lib/jwtsecret/jwt.hex \  
    --validators-proposer-default-fee-recipient=<FeeRecipientAddress> \  
    --validators-graffiti="<yourgraffiti>"
    
    [Install]  
    WantedBy=multi-user.target  

  
To save and exit nano, press <CTRL> + X, followed by Y, and then <ENTER>.  
  
Once you've made the changes, you'll need to reload systemd to apply them and start the Teku service. To do this, run the following commands:  
  

    sudo systemctl daemon-reload  
    sudo systemctl start teku  

  
To confirm that the service is running correctly, check its status by running:  
  
`sudo systemctl status teku`  
  
Once you've confirmed that the service is running correctly, you can quit the status screen by pressing Q.  
  
To ensure that the Teku service starts automatically on reboot, enable it by running this command:  
  
`sudo systemctl enable teku`  
  
## 5. Fund the Validator Keys  
For this visit [https://launchpad-ascend.bidaochain.com](https://launchpad-ascend.bidaochain.com) and upload you in step 1 created deposit file (it should look like this `deposit_data-[timestamp].json`) and click continue.  
  
To connect your wallet, select MetaMask or another supported wallet, log in, and choose the account that holds your BISC. Then, click "Continue" to proceed.  
  
After this, a summary will be displayed, showing the number of validators and the total amount of BISC required. If the information is correct, check the boxes to agree to the terms and conditions, and then click "Continue" to proceed.  
  
When you're ready to deposit your BISC, click on "Initiate All Transactions" or click on each transaction individually.  
  
Congratulations, you have now successfully become an Bidao Staker!
