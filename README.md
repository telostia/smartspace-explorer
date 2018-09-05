## Smrtspace explorer

## Explorer guide
## ===============

### install required nodejs ver. v6 on 16.04, v8 on 18.04 ubuntu

* sudo apt-get update
* sudo curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
* sudo apt-get install -y nodejs
* sudo npm install
* sudo apt-get install


## Install Mongo DB
## ================

* sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
* echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
* sudo apt-get update
* sudo apt-get install -y mongodb-org
* sudo systemctl start mongod
* sudo npm install forever -g


## Start mongodb cli:
## =================

#db.createUser( { user: "iquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )
#old iquidus

mongo

* use explorerdb
* db.createUser( { user: "ciquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )
* exit


## Install Explorer
## ================

* git clone https://github.com/telostia/smartspace-explorer.git explorer

* cd explorer && npm install --production

* cp ./settings.json.template ./settings.json

### Editing settings.json

**Blockchain api**
Go to windows wallet -> transactions tab > find a transaction and right click showtransactions details, copy address into the settings file. Then copy transaction hash
and go to debug screen and type(ignore the <> signs): gettransaction <paste transaction hash here>

sample transaction hash: 3014b61641fd17e496fe13163e1c5a3f0e36e8db1a6aa66088bb0667c6f84618

sample command: gettransaction 3014b61641fd17e496fe13163e1c5a3f0e36e8db1a6aa66088bb0667c6f84618

past the transaction hash into the table under **txhash**

copy block hash into the table under **blockhash**

In debug screen again, type in command(ignore < > signs): getblock <paste block hash here>

sample: getblock a2dd13140ab068c7339c8f2f9f81485a497ea22696798c90c4e6099bffec9546

copy the height and paste in **blockindex** in the table.
sample block height: 52

sample address: XPP8VCLpFhTDjS2UDshq8k1adDriziRoW7

sample data:
"api": {
    "blockindex": 52,
    "blockhash": "00000000001b8c30360db57b575b3c2bf668b0ed50683f567afd47ae1773efb8",
    "txhash": "3014b61641fd17e496fe13163e1c5a3f0e36e8db1a6aa66088bb0667c6f84618",
    "address": "XPP8VCLpFhTDjS2UDshq8k1adDriziRoW7"
  },

## Setting genesis in table
Go to debug and type command: getblockhash 0. Copy hash into table under **genesis_block**.

sample genesis hash: 9c661341062791ebc601c5082842642df88f6462f6378a51cc5ec0a5e4c37060

type in debug command: getblock <paste your genesis hash here>

sample command: getblock 7f1ed91786d6a81cd2a6c4f32b3a3b1cec0f7f14ac388bc8c72d8dcfc415ff28

sample tx: 7f1ed91786d6a81cd2a6c4f32b3a3b1cec0f7f14ac388bc8c72d8dcfc415ff28

copy tx in table under **genesis_tx**.

"genesis_tx": "7f1ed91786d6a81cd2a6c4f32b3a3b1cec0f7f14ac388bc8c72d8dcfc415ff28",
  "genesis_block": "9c661341062791ebc601c5082842642df88f6462f6378a51cc5ec0a5e4c37060",

### Run the wallet
run command in linux: smrt -reindex -txindex

### To use forever to start (run in directory of explorer):

forever start -c "npm start" ./

### Start reindexing the blocks

node scripts/sync.js index reindex

crontab -e

crontab information:

```
*/1 * * * * cd /root/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
*/2 * * * * cd /root/explorer && /usr/bin/nodejs scripts/masternodes.js > /dev/null 2>&1
*/5 * * * * cd /root/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1
```




----------------------------------------------------------------------------------

Ciquidus Alpha - 1.7.1
================

The Chaincoin block explorer.

This project is a fork of [Iquidus Explorer](https://github.com/iquidus/explorer) so massive thanks go out to Luke Williams for his code! Thank you!!!

### See it in action

*  [explorer.chaincoin.org](https://explorer.chaincoin.org)


### Requires

*  node.js >= 0.10.28
*  mongodb 2.6.x
*  *coind

### Create database

Enter MongoDB cli:

    $ mongo

Create databse:

    > use explorerdb

Create user with read/write access:

    > db.createUser( { user: "ciquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )

*note: If you're using mongo shell 2.4.x, use the following to create your user:

    > db.addUser( { user: "username", pwd: "password", roles: [ "readWrite"] })

### Get the source

    git clone https://github.com/suprnurd/ciquidus explorer

### Install node modules

    cd explorer && npm install --production

### Configure

    cp ./settings.json.template ./settings.json

*Make required changes in settings.json*

### Start Explorer

    npm start

*note: mongod must be running to start the explorer*

As of version 1.4.0 the explorer defaults to cluster mode, forking an instance of its process to each cpu core. This results in increased performance and stability. Load balancing gets automatically taken care of and any instances that for some reason die, will be restarted automatically. For testing/development (or if you just wish to) a single instance can be launched with

    node --stack-size=10000 bin/instance

To stop the cluster you can use

    npm stop

### Syncing databases with the blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorers root directory.

    Usage: node scripts/sync.js [database] [mode]

    database: (required)
    index [mode] Main index: coin info/stats, transactions & addresses
    market       Market data: summaries, orderbooks, trade history & chartdata

    mode: (required for index database only)
    update       Updates index from last sync to current block
    check        checks index for (and adds) any missing transactions/addresses
    reindex      Clears index then resyncs from genesis to current block

    notes:
    * 'current block' is the latest created block when script is executed.
    * The market database only supports (& defaults to) reindex mode.
    * If check mode finds missing data(ignoring new data since last sync),
      index_timeout in settings.json is set too low.


*It is recommended to have this script launched via a cronjob at 1+ min intervals.*

**crontab**

*Example crontab; update index every minute and market data every 2 minutes*

    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/masternodes.js > /dev/null 2>&1
    */5 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1

forcesync.sh and forcesynclatest.sh (located in scripts/) can be used to force the explorer to sync at the specified block heights

### Wallet

The wallet connected to Ciquidus must be running with atleast the following flags:

    -daemon -txindex

### Donate
    
    CHC: CLkWg5YSLod772uLzsFRxHgHiWVGAJSezm
    BTC: 1J8Chi5teDJrvBtSuQhioNCHfTNBCcCrPx

### Known Issues
**Database Querys are slow.**
Index are not working correct you need to create the index in the mongodb

    db.addresses.ensureIndex({"a_id":1})
    db.getCollection('txes').ensureIndex({"txid":1})
    db.getCollection('txes').ensureIndex({"timestamp":1})
    db.txes.collection.createIndex( { timestamp: -1 } )
    db.txes.ensureIndex({"total":1})

**script is already running.**

If you receive this message when launching the sync script either a) a sync is currently in progress, or b) a previous sync was killed before it completed. If you are certain a sync is not in progress remove the index.pid from the tmp folder in the explorer root directory.

    rm tmp/index.pid

**exceeding stack size**

    RangeError: Maximum call stack size exceeded

Nodes default stack size may be too small to index addresses with many tx's. If you experience the above error while running sync.js the stack size needs to be increased.

To determine the default setting run

    node --v8-options | grep -B0 -A1 stack_size

To run sync.js with a larger stack size launch with

    node --stack-size=[SIZE] scripts/sync.js index update

Where [SIZE] is an integer higher than the default.

*note: SIZE will depend on which blockchain you are using, you may need to play around a bit to find an optimal setting*

### License

Copyright (c) 2017, The Chaincoin Community  
Copyright (c) 2015, Iquidus Technology  
Copyright (c) 2015, Luke Williams  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Iquidus Technology nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
