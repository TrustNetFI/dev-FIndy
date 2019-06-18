# Monitor Node Setup

FIndy monitoring node setup is based on the [monitoring node implementation made available by the VON network](https://github.com/bcgov/von-network/blob/master/server/anchor.py).
Instructions for running the VON Network can be found from their [Github project](https://github.com/bcgov/von-network).

## Pre-Requisites

Following information is needed in order to run the monitoring node:

* Access to pool transaction genesis file `pool_transactions_genesis` stored in a known location.

## AWS EC2 Instance Setup

Following is an example setup for monitoring node in AWS EC2:

* Instance type: `t2.micro`
* OS: Ubuntu 18.04
* Disk space: 8Gb

## Creation of DID for Monitoring Node

Following commands need to be run in a FIndy Steward Node in order to create DID for monitor node with the correct role:

```
pool create findy-dev-ledger gen_txn_file=<location of pool_transactions_genesis file>
pool connect findy-dev-ledger
did new seed=******* metadata="network monitor"
 
Did "<monitor node DID>" has been created with "<monitor node verkey>" verkey
Metadata has been saved for DID "<monitor node DID>"
 
did use <steward DID>
ledger nym did=<monitor node DID> role=NETWORK_MONITOR verkey=<monitor node verkey>
ledger get-nym did=<monitor node DID>
```

Seed from the above commands should be stored to monitor node `/home/ubuntu/network_monitor_seed`.

## Running Monitor Node

Monitor node can be run with following commands:

```
git clone https://github.com/bcgov/von-network.git
cd von-network/
 
 
virtualenv --python=python3.6 venv
source venv/bin/activate
pip install libnacl python3-indy
 
sudo apt-get install libsodium23 libsodium-dev
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 68DB5E88
sudo add-apt-repository "deb https://repo.sovrin.org/sdk/deb xenial stable"
sudo apt-get update
sudo apt-get install -y libindy
 
GENESIS_FILE=/home/ubuntu/pool_transactions_genesis LEDGER_SEED=`cat /home/ubuntu/network_monitor_seed` python -m server.server &> ~/web-server.log &
```

Restarting the instance requires removing `/home/ubuntu/.indy-client`.