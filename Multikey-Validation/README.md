# Multiversx - Multikey Validation Installation Guide

Welcome to the Multiversx Multikey Validation Installation Guide! This guide will walk you through the process of setting up Multiversx on your Ubuntu 22.04 LTS server. Ensure you have the necessary prerequisites and follow the steps carefully to establish a secure and efficient multikey validator node setup.


## Requirements

Before you start the installation process, make sure you have the following:

- An Ubuntu 22.04 LTS server with the following specifications:
  - Bare metal setup
  - Secured through SSH key with a password (No direct password connection)
  - 4 Cores / 8 Threads
  - 32 GB RAM
  - 500 GB SSD
  - 1 Gbps Network (Unlimited)

- ValidatorKey.pem file for each node you intend to run on this multikey machine setup.

## Procedure

Follow these steps to install and set up Multiversx Multikey Validation on your Ubuntu server.

### 1. Update & Upgrade Machine

It's crucial to keep your machine up to date and secure. Run the following commands periodically to ensure your packages are aligned with the latest security updates.

```bash
# -- Stepped Update Sequence

# Fetch the latest package information
sudo apt -y update

# Upgrade installed packages to the latest versions
sudo apt -y upgrade

# Perform a distribution upgrade, if needed
sudo apt -y dist-upgrade

# --  Or with a single command
sudo apt -y update && sudo apt -y upgrade && sudo apt -y dist-upgrade
```


### 2. Create new user

We recommend not running nodes as the root user. Create a new user and add it to the sudo group.

```bash
# Create a new user with sudo privileges and a home directory
sudo useradd -s /bin/bash -d /home/phybyte -m -G sudo phybyte

# Allow the new user to run sudo commands without a password
echo 'phybyte ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers.d/myOverrides

# Switch to the new user to continue the setup
sudo su phybyte
```

### 3. Download Node sources

Clone the Multiversx Node sources from the official repository.

```bash
cd ~
git clone https://github.com/multiversx/mx-chain-scripts
```

### 4. Modify the variables.cfg file

Open the variables.cfg file using your preferred text editor and modify the following variables:

```bash
vim ~/mx-chain-scripts/variables.cfg
```

Replace USERNAME with the username you created in step 2.
```
ENVIRONMENT = [“testnet”, “devnet” or “mainnet]”
CUSTOME_HOME = “/home/USERNAME”
CUSTOM_USER = “USERNAME”
```
### 5. Install Multiversx Nodes

Run the following command to install Multiversx Nodes on your machine.

```bash
  cd ~/mx-chain-scripts
  ./script.sh install
```

During the installation, you will be asked to enter the number of observers you want:

Enter **4** as you want one per shard

You will also be ask the names of your 4 observers, for security  reasons you should choose names that are not related to your identity.
Example: Obs-0 Observer-1 Bob-2 or SOMETHING-[0,1,2,meta]

### 6. Configuration of the nodes

We need to edit the prefs.toml files for each node, the files are divide into sections 

```bash
 cd ~/elrond-nodes
 vim node-0/config/prefs.toml
 vim node-1/config/prefs.toml
 vim node-2/config/prefs.toml
 vim node-3/config/prefs.toml
 ```
In these files, 2 sections are of interest for our case:
- **Preferences**: allow to configure the observer.
  - DestinationShardAsObserver: [0,1,2, metachain]
  - NodeDisplayName: Already set to the different names given during the installation.
  - RedundancyLevel: 0 for the main machine, 1 for the backup machine, 2 for the backup of the backup machine...

``` bash
[Preferences]
   # DestinationShardAsObserver represents the desired shard when running as observer
   # value will be given as string. For example: "0", "1", "15", "metachain"
   # if "disabled" is provided then the node will start in the corresponding shard for its public key or 0 otherwise
   DestinationShardAsObserver = ""

   # NodeDisplayName represents the friendly name a user can pick for his node in the status monitor when the node does not run in multikey mode
   # In multikey mode, all bls keys not mentioned in NamedIdentity section will use this one as default
   NodeDisplayName = ""

   # Identity represents the keybase/GitHub identity when the node does not run in multikey mode
   # In multikey mode, all bls keys not mentioned in NamedIdentity section will use this one as default
   Identity = ""

   # RedundancyLevel represents the level of redundancy used by the node (-1 = disabled, 0 = main instance (default),
   # 1 = first backup, 2 = second backup, etc.)
   RedundancyLevel = 0
```
- **NamedIdentity**: allow to configure the list of managed keys and the identity behind the nodes.
  - Identity: Not Mandatory but allows to link more assets of your identity from github. 
  - NodeName: The prefix that will be put for the nodes managed on this shard ( Most likely the same on all nodes of the same machine).
  - BLSKeys: An array containing all the public keys managed by this node. 
``` bash
# NamedIdentity represents an identity that runs nodes on the multikey
# There can be multiple identities set on the same node, each one of them having different bls keys, just by duplicating the NamedIdentity
[[NamedIdentity]]
   # Identity represents the keybase/GitHub identity for the current NamedIdentity
   Identity = "MyIdentityOnGithub"
   # NodeName represents the name that will be given to the names of the current identity
   NodeName = "MyIdentity" # Or MyIdentity-B for the redondant machine
   # BLSKeys represents the BLS keys assigned to the current NamedIdentity
   BLSKeys = [
      "e90f794398bb5fb32c81ce738576ee6c990f4651c0154c2bf0f8e1632b77f3571c771aec1afc2df8dffcdd6e5d1dd03f80a05a0ef376cfbb02f7bdaeb45a3c7fe83626cc7d616789eae5ba49a1f8f769666e9ad7b21f65d58ecf92bf10dff07",
      "6eee026e359f44c3cee9837c583467f1489d4b9ad25a8651e13600a6a1c9643a16e84f0cd83b43e7613c76765a0f40e82507e082c2fe8506927d0481ee76e65a3e10daf635bddd8ffdb2a4600058ff4341799b1fdbd29ba30fdf4f6fa7b9181",
      "dc8a1f0c76320c343bc2c185c25aa68dd431af473c034e20b287970a04ede762f2358a12b135215d7772994fb9304003fd67e1dbe69771f7f853a7c6b62ca397d1a88764fa93e380575fdd2d71f3c082b0f68a8dee241e1a191418d2791640f",
      "1ae0b12bf0066e0e1103fb71c4b3215c1738dfae98e3e97d537ea0b86686f50293ca4b31087d1c1329506d3446997011639d3a253abc2994ae967c7979e1fa5e030e1b31ad9dd23ba50a74e57eb8397ef99f9f87ede87316c80d0d2326fc38e",
      "7917d57307ffda2b10d54a9d00aac6c484058680032a3bc041fcc91428d193a37dec31d86fb1f01fe63370ae9de250ec5ff3078820734f98187f1e1cf5a9dc781c296530c183ab0e4315055338e1af483f1c857ac706bd9228228a213d05787",
      "d1a605e7fef0eb38a0cf09eade0721c71391fd572e8033ae7a3f97fb2e123d4d0832e26bcb5b71e4a1b3f06c564e0042fb4c04bfcd817992f023591b79609c09bb99f9aa5d31acafadb79ae61234ecad567bc71c416c67d525915a7ebf6aa02"
   ]
```
### 6. Syncronize the observer nodes

Before to add the private keys of our nodes, we will start to synchronise our observers with the current epoch.

```bash
#Go to the script folder and start the nodes
 cd ~/mx-chain-scripts
 ./script.sh start
```
We will wait for the nodes to be synchronised with the current epoch, you can check the status of the nodes with the following command:

```bash
  cd ~/elrond-utils
  ./elrond-utils/termui --address localhost:8080
  ./elrond-utils/termui --address localhost:8081
  ./elrond-utils/termui --address localhost:8082
  ./elrond-utils/termui --address localhost:8083
  ```

The synchronization can take a moment, we can continue the configuration of the nodes while waiting.

### 7. Add the validator keys to the nodes

For every BLSKey (nodes that you own) added in step 6, we have a validatorKey.pem file allocated to it, it should be similar to this one:

```bash
-----BEGIN PRIVATE KEY for e296e97524483e6b59bce00cb7a69ec8c0d1ac4227925f07fdd57b3ab4ec2f64b240728a0a3c5be2930aea570bf12c12314e25d942b106472800e51524add26ec9546475c1cfae91dd7e799f256d1b0758e17aaa3898c29d489bd87c86d04498-----
YzJlODM0NTdmOTVmYMDVjZGRiNzdiODc1N2YyZGEx
ZGRhYWY5MTI5Y2NlOWQyOQ==
-----END PRIVATE KEY for e296e97524483e6b59bce00cb7a69ec8c0d1ac4227925f07fdd57b3ab4ec2f64b240728a0a3c5be2930aea570bf12c12314e25d942b106472800e51524add26ec9546475c1cfae91dd7e799f256d1b0758e17aaa3898c29d489bd87c86d04498-----
```

Then then have to create a new file called allValidatorsKey.pem and add the contents of all your validatorKey.pem files in it.

```bash
-----BEGIN PRIVATE KEY for e296e97524483e6b59bce00cb7a69ec8c0d1ac4227925f07fdd57b3ab4ec2f64b240728a0a3c5be2930aea570bf12c12314e25d942b106472800e51524add26ec9546475c1cfae91dd7e799f256d1b0758e17aaa3898c29d489bd87c86d04498-----
YzJlODM0NTdmOTVmYMDVjZGRiNzdiODc1N2YyZGEx
ZGRhYWY5MTI5Y2NlOWQyOQ==
-----END PRIVATE KEY for e296e97524483e6b59bce00cb7a69ec8c0d1ac4227925f07fdd57b3ab4ec2f64b240728a0a3c5be2930aea570bf12c12314e25d942b106472800e51524add26ec9546475c1cfae91dd7e799f256d1b0758e17aaa3898c29d489bd87c86d04498-----
-----BEGIN PRIVATE KEY for 5585ddceb6b7bf0d308162efd895d0717b22bab6b0412f09fb9cee234be73d197bfef8ae10064be5733472c573894015029672b70f63e0b58c7ab2e831ee0aff88b868e4d712bec0baf9a1cd1982e138af9b6cc55e4454b01cb8ad02a064f515-----
MzNlZjQyYTRhZDc3ZDBkZDk1M2JmNGIwNWE2MzczMmYxZWUy
ZWVkNzNiOGQ1ZDQ0NmEzMg==
-----END PRIVATE KEY for 5585ddceb6b7bf0d308162efd895d0717b22bab6b0412f09fb9cee234be73d197bfef8ae10064be5733472c573894015029672b70f63e0b58c7ab2e831ee0aff88b868e4d712bec0baf9a1cd1982e138af9b6cc55e4454b01cb8ad02a064f515-----
-----BEGIN PRIVATE KEY for 791c7e2bd6a5fb1371af18269267ad8ef9e56e264c4c95703c57526b16b84dd8df6347c0cc14f93d595a12316d38ae11264e05d2fa26d80387d12db52c1a98e93064d073d02549c71ec4e352d73724c21c02245b25d3643b532fac25d7580f0b-----
OTcxYjYyNWMzMzlkY2JhNTAyODMwNzZlYjMyY2MxMmYzNThiMjNiNzYz
NTA4YjFjMTVlYTIwNDYyMw==
-----END PRIVATE KEY for 791c7e2bd6a5fb1371af18269267ad8ef9e56e264c4c95703c57526b16b84dd8df6347c0cc14f93d595a12316d38ae11264e05d2fa26d80387d12db52c1a98e93064d073d02549c71ec4e352d73724c21c02245b25d3643b532fac25d7580f0b-----
...
...
```

Then we will copy the allValidatorsKey.pem file into each node configuration folders:

```bash
cd ~/elrond-nodes
cp allValidatorsKeys.pem node-0/config/
cp allValidatorsKeys.pem node-1/config/
cp allValidatorsKeys.pem node-2/config/
cp allValidatorsKeys.pem node-3/config/
```
### 8. Restart the nodes

Now that we have 4 observers **synchronised on their shards**  (wait if its not the case yet), and they all all have access to the private keys of your nodes.

You have to turn them off and on again to take the modifications into consideration, you can do it with the following commands:


```bash
  cd~/mx-chain-script
  ./script.sh stop
  ./script.sh start
```

## Connect with Multiversx Community

Thank you for using the Multiversx Multikey Validation Installation Guide! For more updates, discussions, and support, connect with the Multiversx community through their official channels:

### Telegram Channels:

[Multiversx Validators Channel](https://t.me/MultiversXValidators)

Multiversx Community Chat

### Twitter:

Follow me on Twitter for more guides: [@Phybyte](https://twitter.com/Phybyte)