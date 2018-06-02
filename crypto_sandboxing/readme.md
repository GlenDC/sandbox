# Introduction
This document describes the details of an atomicswap test environment setup. the main goals of the the document are:
- Architecture of test environment
- How to setup your test environment
- Explains the details of atomicswap process

## Test environment architecture
The following diagram shows the main components of the atomicswap setup
![Atomicswap setup](https://raw.githubusercontent.com/Jumpscale/sandbox/master/crypto_sandboxing/atomicswap_setup.jpg)

The setup shows:
- One zero-os node running on packet.net 
- A node robot container running on top of the ZOS node
- Two VMs created via the node robot each one is running a full node on BTC and TFT networks recpectivly
- A js9 node where the atomicswap SAL can be executed

## How to setup your test environment
We provide an end2end script that can be used to does a complete setup of the environment. The script needs to run from a JS9 node and it needs the following clients to be configured:
- At least one sshkey client to be configured and loaded
- At least one zerotier client to be configured
- At least one packet client to be configured

### Environment Variables
The setup depends on some variables that would be used to control how the setup will deployed and also configur the single VMs running the bockchains. The following environment variables can be set before running the deploy script to set the different values of the variables
- ZT_NET_ID: [REQUIRED] This is the only required environment variables to be set, if it is not set then the script will fail since we will not know which zerotier netowrk to use when setting up the zero-os node. Make sure that your API token configured in the zerotier client have access to this network.
- SSHKEY_NAME: [default: id_rsa] This will be the keyname of the sshkey to be used to authorized the current node to communicate with the remote nodes created during the deployment.
- ZT_CLIENT_INSTANCE: [default: main] Name of the zerotier client instance to be used.
- PACKET_CLIENT_INSTANCE: [default: main] Name of the packet.net client instance to be used.
- TFT_WALLET_PASSPHRASE: [default: pass] Passphrase for initializing the TFT wallet.
- TFT_WALLET_RECOVERY_SEED: [default: None] Recovery seed of TFT wallet, if not provided, new wallet will be initialized.
- ZOS_NODE_NAME: [default: atomicswap.test] Name of the Zero-OS node to be created on top of Packet.net

### Starting the deploy script
If you already have the the pre-requisites satisfied and at least exported the ZT_NET_ID variable then you are ready to run the deploy script.To do that you need to execute the following command on the js9 node where you configured your clients:
```bash
export ZT_NET_ID=<network_id>
js9 'j.clients.git.pullGitRepo("https://github.com/Jumpscale/sandbox.git")'
python3 /opt/code/github/jumpscale/sandbox/crypto_sandboxing/deploy.py
```
The script will do the following steps:
- Create a ZOS node on packet.net with the name of ZOS_NODE_NAME variable
- Create two VMs each containing the bockchains binaries using the flist [https://hub.gig.tech/abdelrahman_hussein_1/ubuntucryptoexchange.flist]
- Forward port 2250 to connect to the TFT node.
- Forward port 2350 to connect to the BTC node.
- Start blockchains on the two VMS
- On the TFT node, it will initialize the wallet using the provided passphrase and recovery seed or use the default values
- Wait for the blockchains to sync with the respective networks.
