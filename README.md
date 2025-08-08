# Aztec-Prover-Node-Guide 
This is step to step on how to run prover Node on Aztec


A step by step to run *Prover Node* on Aztec Network Testnet.
Prover is intended to generates ZK Proofs that attest to rollup integrity that is pivotal to the protocol.

### NB: 
- If you are here for *rewards/incentive/airdrop* this is not your place. Either it's a Prover node or Sequencer node. Do it as a Hobby!
- Use different wallet if you are running a Validator, it's not recommended to use the same wallet because there might be a Nonce Issue if both Prover and Sequencer node submits Txs at the same time.
- Also, use different server if you running Sequencer/Validator Node already.
## YOU WILL NEED 2 VPS TO HAVE A SUCCESSFUL EXPERIENCE FOR THIS PROVER NODE
# One small and a bigger one for the agents
### Hardware Minimum Requirements
# bigger spec
**RAM** 256 GB+

**CPU** 96 Cores

**DISK** 1TB+ SSD

# small spec vps for prover node and broker
**RAM** 256 GB+

**CPU** 64 Cores

**DISK** 500gb+ SSD

The Prover node uses a really high resource than the Secuencer Node, you can go with Higher machine specs than my recommended one but this is what work for me.

If you running on low specs machine, you will likely face an **Error Stopping job due to deadline hit** and **Error: Epoch proving failed: Proving cancelled**. 

Which means your Provers failing to submit Proof on Epoch because your Hardware can't catch the Epoch deadline.

Errors from agent vps not able to fetch job is caused if the prover node/broker are **shut** , **not running** or **port not properly exposed**


*Rent servers from HostKey ([Hostkey](https://hostkey.com/dedicated-servers/high-ram/)). it's  the one that works for me, others options may work too* 

*Or see Dedicated server from Hetzner (https://www.hetzner.com/dedicated-rootserver/). Good for quality machine*

## INSTALL ON BOTH VPS

### 1. Install Dependencies

```sudo apt-get update && sudo apt-get upgrade -y```

```sudo apt install curl build-essential wget lz4 automake autoconf tmux htop pkg-config libssl-dev tar unzip  -y```
```
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable docker
sudo systemctl restart docker
```

## 2. Install Aztec tools

```bash -i <(curl -s https://install.aztec.network)```

```echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc```

```source ~/.bashrc```

Check if you installed successfully, run: aztec -V

## 3. Allow some ports

```
sudo ufw allow 22
sudo ufw allow ssh
sudo ufw enable
sudo ufw allow 8080
sudo ufw allow 8081
sudo ufw allow 40400
sudo ufw allow 40400/udp
```
## ON THE SMALLER VPS HERE, RUN PROVER AND BROKER HERE :
**smaller vps on HostKey image:* <img width="1080" height="564" alt="image" src="https://github.com/user-attachments/assets/dbab819f-df20-4104-a93c-2a9a2f5ec24a" />


## 4. Run Prover/Broker Node
```mkdir Broker_prover```

```cd Broker_prover```

### Create .env file

```nano .env```

Paste this inside the .env and fill your data

```
P2P_IP=Your_smaller_VPS_IP
ETHEREUM_HOSTS=Your_Execution_Layer_RPC_Endpoint
L1_CONSENSUS_HOST_URLS=Your_Consensus_Layer_RPC_Endpoint
PROVER_PUBLISHER_PRIVATE_KEY=0xYourPrivatekey
PROVER_ID=0xYourAddress
```

Save it, CTRL + XY

### Using Docker Compose

```nano docker-compose.yml```

Paste this into the docker-compose.yml :

```
name: aztec-prover
services:
  broker:
    image: aztecprotocol/aztec:latest
    command:
      - node
      - --no-warnings
      - /usr/src/yarn-project/aztec/dest/bin/index.js
      - start
      - --prover-broker
      - --network
      - alpha-testnet
    environment:
      DATA_DIRECTORY: /data-broker
      LOG_LEVEL: info
      ETHEREUM_HOSTS: ${ETHEREUM_HOSTS}
      P2P_IP: ${P2P_IP}
    volumes:
      - ./data-broker:/data-broker
    ports:
      - "8081:8080"   # Expose broker's internal 8080 as external 8081
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet --prover-broker'

  prover-node:
    image: aztecprotocol/aztec:latest
    depends_on:
      - broker
    environment:
      P2P_ENABLED: "true"
      DATA_DIRECTORY: /data-prover
      P2P_IP: ${P2P_IP}
      DATA_STORE_MAP_SIZE_KB: "134217728"
      ETHEREUM_HOSTS: ${ETHEREUM_HOSTS}
      L1_CONSENSUS_HOST_URLS: ${L1_CONSENSUS_HOST_URLS}
      LOG_LEVEL: info
      PROVER_BROKER_HOST: http://broker:8080  # within docker network
      PROVER_PUBLISHER_PRIVATE_KEY: ${PROVER_PUBLISHER_PRIVATE_KEY}
    volumes:
      - ./data-prover:/data-prover
    ports:
      - "40400:40400"
      - "40400:40400/udp"
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet --archiver --prover-node'

```
**NB: I expose the broker to port 8081 , you can use a different port if 8081 is taken already on your device**

- Save it by CTRL + XY

- Then, run the Node Docker

```docker compose up -d```

- See the Docker Status

```docker ps```

- This should output below

- <img width="1347" height="152" alt="image" src="https://github.com/user-attachments/assets/d4cb2c2a-0c69-4c7f-a82e-823415baee16" />

- Optional: Stop and kill node

```docker compose down -v```

- Restarting the docker

```docker compose down -v && docker compose up -d```


## Monitoring the Prover and broker logs

```docker logs -f aztec-prover-prover-node-1```
```docker logs -f  aztec-prover-broker-1```

# Once the logs are working fine with no error, go ahead to the bigger VPS.

## Make sure you can see the prover and broker run for atleast 1 minute before moving to the Bigger VPS .

## ON THE BIGGER VPS HERE, RUN AGENT :
**Bigger vps on HostKey image:* <img width="1080" height="489" alt="image" src="https://github.com/user-attachments/assets/ffe3ba02-2347-4fbd-b5d9-c219849374d5" />

## 1. Run minimum of 30 agents here
```mkdir agents```

```cd agents```

### Create .env file

```nano .env```

Paste this inside the .env and fill your data

```
P2P_IP=Your_Bigger_VPS_IP
ETHEREUM_HOSTS=Your_Execution_Layer_RPC_Endpoint
L1_CONSENSUS_HOST_URLS=Your_Consensus_Layer_RPC_Endpoint
PROVER_PUBLISHER_PRIVATE_KEY=0xYourPrivatekey
PROVER_BROKER_HOST=http://<your_SMALLvps_IP_Here>:8081
PROVER_ID=0xYourAddress
```

Save it, CTRL + XY

### Using Docker Compose

```nano docker-compose.yml```

Paste this into the docker-compose.yml :

```
services:
  agent:
    image: aztecprotocol/aztec:latest
    command:
      - node
      - --no-warnings
      - /usr/src/yarn-project/aztec/dest/bin/index.js
      - start
      - --prover-agent
      - --network
      - alpha-testnet
    environment:
      PROVER_AGENT_COUNT: "30"
      PROVER_AGENT_POLL_INTERVAL_MS: "10000"
      PROVER_BROKER_HOST: ${PROVER_BROKER_HOST}
      PROVER_ID: ${PROVER_ID}
    volumes:
      - ./data-prover:/data-prover
    entrypoint: >
      sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet --prover-agent'

```

- Save it by CTRL + XY

- Then, run the Node Docker

```docker compose up -d```

- See the Docker Status

```docker ps```

**confirm the container is runing up to a minute**

**you can change agent number from 30 to any other number by changing it this to any figure based on your device   *PROVER_AGENT_COUNT: "30"**

## Useful Command

- Monitoring the Agent logs on Bigger VPS

```docker logs -f aztec-agent-only-agent-1```


### Check if your prover are working on the smaller VPS

```docker logs -f aztec-prover-prover-node-1  2>&1 | grep --line-buffered -E 'epoch proved|epoch'```

You will get logs like this:

<img width="1366" height="368" alt="image_2025-08-08_19-24-35" src="https://github.com/user-attachments/assets/1e2eaae7-edb7-4ee0-84bc-eec006e7d25a" />


### Check if you succesfully Submitted Proof of an Epoch

```docker logs -f aztec-prover-prover-node-1  2>&1 | grep --line-buffered -E 'Submitted'```

You will get logs like this:

<img width="1354" height="364" alt="image_2025-08-08_19-18-17" src="https://github.com/user-attachments/assets/4ae9439c-4edd-4b3e-a7d8-872b675b2d43" />


### Also you can check your Prover Address on Sepolia Etherscan

Paste your Prover address on: https://sepolia.etherscan.io

<img width="1345" height="292" alt="image" src="https://github.com/user-attachments/assets/3658b5f5-f3b1-4294-9ea5-3f1bb2d83976" />


## To Earn a Prover Role on Aztec Discord, You need to submit 750 epoch and  with roughly 30epoch/day, it could take you 25days or more.

**This is FAQ from the DC on it:**

1. How do I get the Prover Role on Discord?
*To qualify, you must:*

  • Run a Prover node that has submitted 750+ epochs

  • Then, DM @dindhdat.eth on aztec discord with the following:

      • Wallet Address

      • Discord Username

      • Link Transaction Hash proving ownership (must be signed from the prover wallet)
*To generate a transaction hash:*
*Send 0 eth (sepolia network) from your wallet to your self and put the message like discord name and wallet address on Rabby wallet*

2. How can I check how many epochs my Prover has submitted?
*You can track your Prover's performance here:*

    • [Etherscan (Sepolia)](https://sepolia.etherscan.io/)


    •  Dune Analytics Dashboard : [Aztec Prover Stats](https://dune.com/rhum/aztec-nb-proofs-tx-new-rollup)
     Paste your address to check how many epoch you have submitted.

**Once you meet up, you get the prove role like below**
<img width="1080" height="627" alt="image" src="https://github.com/user-attachments/assets/a2c05592-8238-4904-945b-f7fac5b122fe" />

   

*This Guide will be updated here and on my X handle on [HallenjayArt](https://x.com/HallenjayArt) !*

