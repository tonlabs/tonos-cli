# TON OS command line tool

`tonos-cli` is a command line interface utility designed to work with TON blockchain.

## How to install through TONDEV
You can use [TONDEV](https://github.com/tonlabs/tondev) to install the latest version of tonos-cli:

    tondev tonos-cli install


The installer requires [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) to be installed, so it can install packages globally without using sudo. In case of error, manually set environment variable `PATH=$PATH:$HOME./tondev/solidity`

Refer to [TONDEV documentation](https://github.com/tonlabs/tondev#tonos-cli) for information on updating and managing tonos-cli versions.


## Build Prerequisites

- [Rust](https://www.rust-lang.org/tools/install) latest
- OpenSSL
    ```bash
    sudo apt-get install libssl-dev (openssl-devel on Fedora)
    sudo apt-get install pkg-config
    ```

## How to build

    cargo update
    cargo build [--release] [-j8]

## How to test

    cargo test -- --test-threads 1

## How to run

#### Default

cargo run [subcommand args]

#### Linux
`> cd ./target/release`

`> ./tonos-cli [subcommand args]`

## How to use

By default, tonos-cli connects to `https://net.ton.dev` network.

### Crypto commands:

### 1) Generate seed phrase

    tonos-cli genphrase

### 2) Generate pubkey from seed phrase

    tonos-cli genpubkey "<seed_phrase>"

### 3) Generate keyfile from seed phrase

    tonos-cli getkeypair <keyfile.json> "<seed_phrase>"

### Query commands:

### 1) Get global config

    tonos-cli getconfig <index>

### Smart contract commands:

### 1) Generate Contract Address

    tonos-cli genaddr [--genkey|--setkey <keyfile.json>] [--wc <WC>] <tvc> <abi>

Examples:
* `tonos-cli genaddr --genkey wallet_keys.json wallet.tvc wallet.abi.json`
  `wallet_keys.json` file will be created (or replaced) with a new keypair and this keypair will be used for
  address generation. Address will be generated with workchain `0`.

* `tonos-cli genaddr --setkey existing_keys.json --wc -1 wallet.tvc wallet.abi.json`
  Address will be generated using keypair from the existing file `existing_keys.json` and
  workchain id equal to `-1`.

### 2) Deploy Smart Contract

    tonos-cli deploy [--sign <keyfile>] [--wc <WC>] [--abi <abifile>] <tvc> <params>

Example: `tonos-cli deploy --abi wallet.abi.json --sign wallet_keys.json wallet.tvc {param1:0}`

`--wc` parameter should be a signed integer, that fits 8 bit.
If `--abi` or `--sign` parameters are omitted, it must be specified in the config file. See below.

### 3) Call Method

Call contract in blockchain:

    tonos-cli call [--abi <abi_file>] [--sign <keyfile>] <address> <method> <params>

If `--abi` or `--sign` parameters are omitted, it must be specified in the config file. See below.

Alternative command:

    tonos-cli callex <method> [<address>] [<abi>] [<keys>] params...

`params...` - one or more function arguments in the form of  `--name value`.

If `address`, `--abi` or `--sign` parameters are omitted, it must be specified in the config file. See below.

Example:

    tonos-cli callex submitTransaction 0:1b91c010f35b1f5b42a05ad98eb2df80c302c37df69651e1f5ac9c69b7e90d4e SafeMultisigWallet.abi.json msig.keys.json --dest 0:c63a050fe333fac24750e90e4c6056c477a2526f6217b5b519853c30495882c9 --value 1.5T --bounce false --allBalance false --payload ""

Integer and address types can be supplied without quotes.

`--value 1.5T` - suffix `T` converts integer to nanotokens -> `1500000000`. The same as `--value 1500000000`.

Arrays can be specified without `[]` brackets.


Run contract method locally:

    tonos-cli run [--abi <abi_file>] <address> <method> <params>

If `--abi` parameter is omitted, it must be specified in the config file. See below.

Run funC get-method:

    tonos-cli runget <address> <method> [<params>...]

`params` can have multiple values: one for each function parameter. Example:

    tonos-cli runget -1:3333333333333333333333333333333333333333333333333333333333333333 compute_returned_stake 0x4107f968dc3caf85c2aa4e7d1b842d835d743855f62afe87e5862012be3eff4f

    tonos-cli runget -1:3333333333333333333333333333333333333333333333333333333333333333 active_election_id


### 4) Generate signed message

    tonos-cli message [--abi <abi_file>] [--sign <keyfile>] <address> <method> <params> [--lifetime <seconds>] [--raw] [--output <file_name>]

Generates an external inbound message for account and prints it to the terminal or stores to the file.

`--raw` - generates raw boc message. Can be sent later using `sendfile` command.
`--output` - saves generates message to specified file instead of printing it to the terminal.
Other parameters are the same as in `call` command.

### 5) Send prepared message

    tonos-cli send --abi <abi_file> <message>

Allows to send message generated by `message` command without `--raw` flag.

### 6) Send raw boc message

    tonos-cli sendfile <msg_file>

Allows to send external message serialized as bag of cells and stored in specified `msg_file`.

### 7) Store Parameter Values in the Configuration File

tonos-cli can remember one or more parameter values in the config file and use it automatically
in the most part of subcommands.

    tonos-cli config --<parameter1> <value1> ...

List of possible parameters:

      --abi <ABI>                      File with contract ABI.
      --addr <ADDR>                    Contract address.
      --depool_fee <DEPOOL_FEE>        Value added to message sent to depool to cover it's fees (change will be
                                       returned).
      --keys <KEYS>                    File with keypair.
      --lifetime <LIFETIME>            Period of time in seconds while message is valid.
      --local_run <LOCAL_RUN>          Enable preliminary local run before deploy and call commands.
      --no-answer <NO_ANSWER>          FLag whether to wait for depool answer when calling a depool function.
      --retries <RETRIES>              Number of attempts to call smart contract function if previous attempt was
                                       unsuccessful.
      --timeout <TIMEOUT>              Contract call timeout in ms.
      --url <URL>                      Url to connect.
      --delimiters <USE_DELIMITERS>    Use delimiters while printing account balance
      --wallet <WALLET>                Multisig wallet address. Used in commands which send internal messages through
                                       multisig wallets.
      --wc <WC>                        Workchain id.

Also user can clear one or all parameters from the config file (it will set the default value for
them if it is possible.

    tonos-cli config clear [--<parameter1> <value1> ...]

If no parameters were specified, all parameters will be cleared.

Examples:

    tonos-cli config --url https://main.ton.dev --abi wallet.abi.json --keys wallet_keys.json

After that you can omit `--abi` and `--sign` parameters in `deploy`, `call` and `run` subcommands
and cli by default will connect to `main.ton.dev` network.

    tonos-cli config clear --url

After that `url` for tonos-cli commands will change to the default `https://net.ton.dev`.

`config` command creates config file with name `tonos-cli.conf.json` in the current working directory which will be used by cli at every start.
To override searching config file in current dir use the following methods:

 - define environment variable `TONOSCLI_CONFIG` with path to your config file;
 - define global option `--config <path_to_file>` before any other subcommand
   (example: `tonos-cli --config ../config.json call ...`).

 Global option has higher priority than env variable.

 Also you can explicitly define network in every subcommand by using global option `--url <network>`
 (example: `tonos-cli --url https://main.ton.dev account <address>`).

### 8) Get Account Info

    tonos-cli account <address>

Example:

`tonos-cli account 0:c63a050fe333fac24750e90e4c6056c477a2526f6217b5b519853c30495882c9`

### Sample Test Sequence
Task scope: deploy a contract to TON Labs testnet at net.ton.dev.

#### 1) compile contract and get `.tvc` file and `.abi.json`. Lets name it `contract.tvc`.

#### 2) generate contract address.

    tonos-cli genaddr --genkey contract_keys.json contract.tvc contract.abi.json

Save `Raw address` printed to stdout.

#### 3) Ask the testnet giver for Grams.

Note: You have to get giver address, abi and keys.

Let's request 10 Grams to our account.

    tonos-cli call --abi giver.abi.json --sign giver_keys.json <giver_address> sendTransaction {"dest":"<our_address>","value":10000000000,"bounce":false}

#### 4) Get our contract state, check that it is created in blockchain and has the `Uninit` state.

    tonos-cli account <raw_address>

#### 5) Deploy our contract to the testnet.

    tonos-cli deploy --abi contract.abi.json --sign contract_keys.json contract.tvc {<constructor_arguments>}

#### 6) Check the contract state.

    tonos-cli account <raw_address>

The contract should be in the `Active` state.

#### 7) Use the `call` subcommand to execute contract method in blockchain.

    tonos-cli call --abi contract.abi.json --sign contract_keys.json <raw_address> methodName {<method_args>}

## DePool commands

Tonos-cli allows to communicate with depool contract using your multisignature wallet.
All depool commands are started with `tonos-cli depool` .

For all commands:

`--addr` - address of depool.
`--wallet` - address of multisig wallet.
`--sign` - path to keyfile or seed phrase of multisig wallet.
`--no-answer` - Do not wait for depool answer when calling a depool function. (Use for multisig with more than one custodian)

All commands allow to omit `--addr`, `--wallet`,  `--no-answer` and `--sign` options only if this values are defined in config file:

    tonos-cli config --addr <address> --wallet <address> --keys <path_to_keys or seed_phrase> --no-answer true

### Deposit stakes

all `--value` parameters must be defined in tons, like this: `--value 10.5`, it means value is 10,5 tons.

#### Ordinary stake

    tonos-cli depool [--addr <depool_address>] stake ordinary [--wallet <msig_address>] --value <number> [--sign <key_file or seed_phrase>] [--no-answer]


#### Vesting stake

    tonos-cli depool [--addr <depool_address>] stake vesting [--wallet <msig_address>] --value <number> --total <days> --withdrawal <days> --beneficiary <address> [--sign <key_file or seed_phrase>] [--no-answer]


#### Lock stake

    tonos-cli depool [--addr <depool_address>] stake lock [--wallet <msig_address>] --value <number> --total <days> --withdrawal <days> --beneficiary <address> [--sign <key_file or seed_phrase>] [--no-answer]

### Remove stakes

    tonos-cli depool [--addr <depool_address>] stake remove [--wallet <msig_address>] --value <number> [--from-round <number>] [--sign <key_file or seed_phrase>] [--no-answer]

### Transfer stakes

    tonos-cli depool [--addr <depool_address>] stake transfer [--wallet <msig_address>] --value <number> --dest <address> [--sign <key_file or seed_phrase>] [--no-answer]

### Enable/disable withdraw

    tonos-cli depool [--addr <depool_address>] withdraw on | off [--wallet <msig_address>] [--sign <key_file or seed_phrase>] [--no-answer]

### View depool events

    tonos-cli depool [--addr <depool_address>] events [--since <utime>]

Prints depool events since defined unixtime. if `--since` is omitted then prints all depool events.

    tonos-cli depool [--addr <depool_address>] events --wait-one

Waits until new event will be emitted and then prints it to the stdout.


### Replenish contract balance

Transfers funds from the multisignature wallet to the depool contract (NOT A STAKE).

    tonos-cli depool [--addr <depool_address>] replenish --value <number> [--wallet <msig_address>] [--sign <key_file or seed_phrase>]

### View depool answers

    tonos-cli depool [--addr <depool_address>] answers [--wallet <msig_address>] [--since <utime>]

Prints depool answers to the specified multisignature wallet since defined unixtime.
If `--since` is omitted then prints all depool events.

### Set donor

    tonos-cli depool [--addr <depool_address>] donor lock --donor <DONOR> [--wallet <msig_address>] [--sign <key_file or seed_phrase>]
    tonos-cli depool [--addr <depool_address>] donor vesting --donor <DONOR> [--wallet <msig_address>] [--sign <key_file or seed_phrase>]

Sets donor address for the depool participant.

### Call ticktock

    tonos-cli depool [--addr <depool_address>] ticktock [--wallet <msig_address>] [--sign <key_file or seed_phrase>]

Calls depool's `ticktock()` function from the multisignature wallet.

## DeBot commands

### Run DeBot

Run DeBot in terminal browser:


```bash
tonos-cli debot start address
```

or

```bash
tonos-cli debot fetch address
```

`address` - TON address of DeBot.

### Invoke DeBot

Run DeBot with predefined message:


```bash
tonos-cli debot invoke message
```

`message` - internal message BOC encoded in base64.


