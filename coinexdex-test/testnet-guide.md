# Coinex DEX Chain Testnet Guide

<br>

---
## Community and Docs


- [>>>>>Join developer channel<<<<<](https://join.slack.com/t/coinexchain/shared_invite/enQtNzA0NjU5ODc3MjM0LTk3NWUzMDA2YmU0NTc5MDg2NDI3NmRjM2VkNzYzNjIyZWM0NzZhMWIwMWQxNGJjNmI3NjVkZWIxZWUwNjJmYTI)

- [Docs](https://github.com/coinexchain/dex-manual)
- [FAQ](https://github.com/coinexchain/dex-manual/blob/01222ef03f4c94231f851ccd3d82e20cb899bb61/docs/02_faq.md)
  - Please read Doc and FAQ before setup your validators
	
- [REST API Doc](https://dex-api.coinex.org/swagger/)


<br>


## Testnet Parameters
- chain-id: coinexdex-test
- seeds info:
    ```
    seeds = "84aa1158fcc35b236a549ed5ccdec64dc2ef7913@52.27.129.48:26656,b075fe9dba4f49ce361c2401b73aefebf51d69b3@54.193.91.79:26656"
    ```

## Prepare
- About files:
  - Genesis Configuration will hosted in this same repo: https://github.com/coinexchain/testnets
  
    ```
    └── coinexdex-test
	    ├── genesis.json        <-- Genesis Configuration 
	    ├── linux_x86_64
	    │   ├── cetcli          <-- RPC Client
	    │   └── cetd            <-- FullNode Daemon
	    ├── mac_osx
	    │   ├── cetcli
	    │   └── cetd
	    ├── md5.sum
	    └── testnet-guide.md
    ```


- Download binary and genesis.json 
    ```
    curl https://raw.githubusercontent.com/coinexchain/testnets/master/coinexdex-test/linux_x86_64/cetcli > cetcli
    curl https://raw.githubusercontent.com/coinexchain/testnets/master/coinexdex-test/linux_x86_64/cetd > cetd
    curl https://raw.githubusercontent.com/coinexchain/testnets/master/coinexdex-test/genesis.json > genesis.json
    ```

## Init Fullnode
- Init fullnode data dir: 
    > cetd init `moniker_name` --chain-id=`coinexdex-test`
    - args: 
        - `moniker_name`: name of your fullnode
        - --chain-id:  chain id to join

    - example:
        ```
        cetd init MyNodeName --chain-id=coinexdex-test
        ```
    
- Deploy genesis configuration
    > cp ./genesis.json ~/.cetd/config/genesis.json

- Set seeds info in `~/.cetd/config/config.toml`
    - replace default `seeds = ""` with
        ```
        seeds = "84aa1158fcc35b236a549ed5ccdec64dc2ef7913@52.27.129.48:26656,b075fe9dba4f49ce361c2401b73aefebf51d69b3@54.193.91.79:26656"
        ```

- Start your node
    > ./cetd start

    - notes: if you want the node running in background, use nohup or setup it as a service of your OS
        - example: `nohup ./cetd start &`
    - check node status by: `cetcli status`


## Join Testnet as Validator
- Prepare an CoinEx Chain Address
    > cetcli keys add `local_address_name`
    - args: 
        - `local_address_name`: name of address in local keystore
    - example(use fullnode_user1 as name of address):
        ```
        cetcli keys add fullnode_user1
        ```
    - notes: **your mnemonic passphrase will print out by this command, store it safely**
    - notes: your private keystore will be in folder: `~/.cetcli`

    - using following commands to get your address
        > cetcli keys show fullnode_user1 -a
        ```
        coinex1d4c5xcvvt9l6dcm82taqzdsle8qqa8nwx9yrgd
        ```

- Get some testnet cet for your address:
    - Get testnet CET from [>>>>>testnet faucet<<<<<](http://faucet.coinex.org)

    - query account status:
        > cetcli q account $(cetcli keys show fullnode_user1 -a) --chain-id=coinexdex-test
        ```
            Address:       coinex1d4c5xcvvt9l6dcm82taqzdsle8qqa8nwx9yrgd
            Coins:         200000000cet
            AccountNumber: 2
            Sequence:      1
        ```
    - notes for developer: 
        - `All tokens' precision are fixed at 8 decimal digits.`
        - `so in previous example 200000000cet on chain means 2CET`

- Get your node's consensus pubkey
    > ./cetd tendermint show-validator
    ```
    coinexvalconspub1zcjduepqagvj8plupgura2vt08xlm3tpur5u0vw89cw8ut9j8a55xq2jetgswccuwt
    ```

- Send CreateValidator tx to become a validator
    > cetcli tx staking create-validator \\\
    > --amount=1000000000000cet \\\
    --pubkey=coinexvalconspub1zcjduepqagvj8plupgura2vt08xlm3tpur5u0vw89cw8ut9j8a55xq2jetgswccuwt \\\
    --moniker=MyNodeName \\\
    --identity=CF1FAAA36A78BE02 \\\
    --chain-id=coinexdex-test \\\
    --commission-rate=0.05 \\\
    --commission-max-rate=0.2 \\\
    --commission-max-change-rate=0.01 \\\
    --min-self-delegation=1000000000000 \\\
    --from $(cetcli keys show fullnode_user1 -a) \\\
    --gas 122295 \\\
    --fees 2445900cet


    > cetcli tx staking create-validator --help
    ```
    create new validator initialized with a self-delegation to it
    Flags:
        --amount string                       Amount of coins to bond
        --commission-max-change-rate string   The maximum commission change rate percentage (per day)
        --commission-max-rate string          The maximum commission rate percentage
        --commission-rate string              The initial commission rate percentage
        --details string                      The validator's (optional) details
        --from string                         Name or address of private key with which to sign
        --gas string                          gas limit; set to "auto" to calculate required gas automatically
        --identity string                     The optional identity signature (ex. Keybase)
        --memo string                         Memo to send along with transaction
        --min-self-delegation string          The minimum self delegation required on the validator
        --moniker string                      The validator's name
        --pubkey string                       The Bech32 encoded PubKey of the validator
        --website string                      The validator's (optional) website
        --chain-id string                     Chain ID of tendermint node
    ```

    - notes, remember to create your --identity at https://keybase.io
        - then your profile image on keybase.io will show in [CoinEx DEX Chain Explorer](https://testnet.coinex.org).

    - notes, gas can be estimated by using --dry-run (without --gas and --fees parameter)
        - `All tokens' precision are fixed at 8 decimal digits.`
        - `so 200000000cet on chain means 2CET`
        - `current network min gas price is 20cet/gas on chain. `
        - `means 0.0000002CET/gas`
    

## Query validator status
- Check your validator status in [CoinEx DEX Chain Explorer](https://testnet.coinex.org/validators)

- Get your validator operator address
    > cetcli keys show fullnode_user1 --bech val
    ```
    NAME:	TYPE:	ADDRESS:					
    fullnode_user1	local	coinexvaloper1kg3e5p2rc2ejppwts6qwzrcgndvgeyztudujdz	

    #coinexvaloper1kg3e5p2rc2ejppwts6qwzrcgndvgeyztudujdz is your validator operator address
    ```

- Query all validators
    > cetcli q staking validators --chain-id=coinexdex-test
    ```
    Validator
    Operator Address:           coinexvaloper1kg3e5p2rc2ejppwts6qwzrcgndvgeyztudujdz
    Validator Consensus Pubkey: coinexvalconspub1zcjduepqagvj8plupgura2vt08xlm3tpur5u0vw89cw8ut9j8a55xq2jetgswccuwt
    Jailed:                     false
    Status:                     Bonded
    Tokens:                     100000000000000
    Delegator Shares:           100000000000000.000000000000000000
    Description:                {fullnode1   }
    Unbonding Height:           0
    Unbonding Completion Time:  1970-01-01 00:00:00 +0000 UTC
    Minimum Self Delegation:    100000000000000
    Commission:                 rate: 0.050000000000000000, maxRate: 0.200000000000000000, maxChangeRate: 0.010000000000000000, updateTime: 2019-06-23 

    ...
    ```

- Do I in vaidator set?
    > ./cetcli q tendermint-validator-set --chain-id=coinexdex-test | grep $(./cetd tendermint show-validator) && echo "in validator set" || echo "not in validator set"


- How to unjail my validator?
	```
	cetcli tx slashing unjail --from fullnode_user1 --chain-id=coinexdex-test --gas=60000  --fees=1200000cet
	```

---
## Diagnosis

- Check block download status and latest block height
    > cetcli status
    ```
    $ cetcli status | jq  '.' | grep catching_up
        "catching_up": false
         #when node is synced to latest block, the "catching_up" will be false
    
    $ cetcli status | jq  '.' | grep latest_block_height
        "latest_block_height": "23323",
    ```

- Check network status
    > curl http://localhost:26657/net_info


## Server Configurations
- upper the ulimit
  > ulimit -n 8192  

- open ports
    - cetd will listen on `TCP 26656`
        - 26656 needs to open to join p2p and consensus process
    - cetd's rpc port is on `TCP 26657`
        - 26657 do not needs open to world

---
## Using cetcli to connect to remote fullnode

- By default `cetcli` will contact `localhost:26657`, you can configue it to send rpc to remote node
    > cetcli config node 111.222.111.222:26657 
    - 111.222.111.222 is the example ip address of remote node

## How to send tokens to others
- estimate gas amount
    ```
    $ cetcli tx send coinex1kg3e5p2rc2ejppwts6qwzrcgndvgeyzt8zl6rk 200000000000000cet --from fullnode_user1 --chain-id=coinexdex-test --dry-run

    gas estimate: 41315
    ```

- do send.  
    ```
    :~$ cetcli tx send coinex1kg3e5p2rc2ejppwts6qwzrcgndvgeyzt8zl6rk 200000000000000cet --from fullnode_user1 --chain-id=coinexdex-test --gas 60000 --fees 1200000cet

    {"chain_id":"coinexdex-test","account_number":"2","sequence":"1","fee":{"amount":[{"denom":"cet","amount":"1200000"}],"gas":"60000"},"msgs":[{"type":"bankx/MsgSend","value":{"from_address":"coinex1d4c5xcvvt9l6dcm82taqzdsle8qqa8nwx9yrgd","to_address":"coinex1kg3e5p2rc2ejppwts6qwzrcgndvgeyzt8zl6rk","amount":[{"denom":"cet","amount":"200000000000000"}],"unlock_time":"0"}}],"memo":""}

    confirm transaction before signing and broadcasting [Y/n]: y
    Password to sign with 'coinex_foundation':
    Response:
    TxHash: 6DFAD8836F999613D33204C51E1C3DFA51D221F673306B0F1299368B3DE98926
    ```

---
## Rest API
- cetcli has a rest-server build-in, so you can query and send txs through REST API. start rest-server by:
    > nohup cetcli rest-server --chain-id=coinexdex-test  --laddr=tcp://localhost:1317  --node tcp://localhost:26657 --trust-node=false > cetcli.nohup.out &
    - the OpenAPI will serve at:  http://localhost:1317/swagger


---
## About testnet parameters
> For better testing the parameter in testnet are different from mainnet.

```
	UnbondingTime = time.Second * 60 * 60 = 1 hour
	Validator's MinSelfDelegation = 10000e8cet
	Proposal's MinDeposit Amount = 1000e8cet
	Proposal's MaxDepositPeriod = 86400 * time.Second
	IssueTokenFee = 1000e8cet
	IssueRareTokenFee = 10000e8cet
	CreateTradePairFee = 10000e8cet
```
