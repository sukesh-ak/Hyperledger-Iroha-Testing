# How to install Hyperledger Iroha v2 (DLT)
Installing & Testing steps for Blockchain ledger - Hyperledger Iroha v2 on Ubuntu

## What is Iroha?
Iroha is a fully-featured blockchain ledger. With Iroha you can:

* Create and manage custom fungible assets, such as currencies, gold, and others
* Create and manage non-fungible assets
* Manage user accounts with a domain hierarchy and multi-signature transactions
* Use efficient portable smart contracts implemented either via WebAssembly or Iroha Special Instructions
* Use both permissioned and permission-less blockchain deployments

## What about Iroha 2?
Iroha 2 is a complete re-write of Hyperledger Iroha in Rust. These two projects are developed concurrently and compatible with each other.
* Fault Tolerance
* Minimalist Code Base
* Flexibility
* Smart Contracts - Iroha 2 supports two approaches:
  * Iroha Special Instructions (ISI)
  * Web Asembly (WASM)
* Static and Dynamic Linking
* Well Tested 

## Prerequisites
```bash
- Git
- OpenSSL
# Install OpenSSL dev library
$ sudo apt-get install libssl-dev

# To avoid linker errors during rust compile 
$ sudo apt install build-essentials

# To avoid compiler error - unable to find OpenSSL on path
$ sudo apt install pkg-config

```

## Compile Iroha 2 CLI Client
```bash
# Install Rust Language support (Iroha v2 is built using Rust)
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Clone repo - get only iroha2-stable branch
$ git clone https://github.com/hyperledger/iroha.git --branch iroha2-stable

$ cd iroha

# Build Iroha Client CLI (output => ./target/release)
$ cargo build -p iroha_client_cli --release

```

## Running Iroha 2 in Docker
```bash
# Download & Install docker
$ curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh 

# Add current user to docker group
$ sudo usermod -aG docker $USER

# Install docker-compose
$ sudo apt install docker-compose

# Run specific version so that test config.json file works by default
# This creates 4 different containers with ports 8080,8081,8082,8083
$ docker-compose  -f docker-compose.stable.yml up

# If you want to run docker containers in the background, use -d option
$ docker-compose  -f docker-compose.stable.yml up -d

```

## Preparing iroha_client_cli to run from bash shell
```bash

# Prepare base folder for config files etc
$ mkdir test_docker

# Copy sample client config file
$ cp ./configs/client/config.json test_docker/

# Create dummy metadata file
$ echo '{"comment":{"String": "Hello Meta!"}}' > test_docker/metadata.json

# Copy iroha_client_cli binary
$ cp ./target/debug/iroha_client_cli test_docker/

# Build & copy Kagami included to generate Private/Public keys later
$ cargo build --bin kagami --release
$ cp ./target/release/kagami test_docker/

# List all registered domains if any
$ ./iroha_client_cli domain list all

```

# Testing functionality of iroha_client_cli
Several domains and accounts etc are already registered when the docker containers are started.
These settings come from the folder `configs/peer/stable/` which is mapped as a volume.

```bash
# Register a new domain
$ ./iroha_client_cli domain register --id="ghost_town"

# Check if domain was created successfully
$ ./iroha_client_cli domain list all

# Create a new Private/Public Key Pair for creating new account
$ ./kagami crypto

# Register a new account under the new domain
$ ./iroha_client_cli account register --id="sukesh@ghost_town" \
  --key="<public key from previous step>"

# Check if account was created successfully
$ ./iroha_client_cli account list all

# Now we can switch to this new account by making changes in the config.json
# Update config.json with the PUBLIC_KEY, PRIVATE_KEY, and ACCOUNT_ID 

# Register an asset called 'ghostcoin'
$ ./iroha_client_cli asset register \  
  --id="ghostcoin#ghost_town" \
  --value-type=Quantity

# Mint 100 quantity of 'ghostcoin'
$ ./iroha_client_cli asset mint \  
  --account="sukesh@ghost_town" \  
  --asset="ghostcoin#ghost_town" \  
  --quantity="100"

# List and check the assets
$ ./iroha_client_cli asset list all

# How to transfer assets from one account to another
$ ./iroha_client_cli asset transfer \  
  --from "sukesh@ghost_town" \  
  --to "account2@ghost_town" \ 
  --asset-id "ghostcoin#ghost_town" \  
  --quantity 5

# Burning assets
$ ./iroha_client_cli asset burn \  
  --account="sukesh@ghost_town" \  
  --asset="ghostcoin#ghost_town" \  
  --quantity="10"


# Monitor transaction events in another bash shell prompt
$ ./iroha_client_cli events pipeline

```

## References
* Hyperledger Irohi 2 [Documentation](https://hyperledger.github.io/iroha-2-docs/)
* Smart Contracts [Using ISI](https://hyperledger.github.io/iroha-2-docs/guide/blockchain/expressions.html)  
* Smart Contracts [Using WASM](https://hyperledger.github.io/iroha-2-docs/guide/blockchain/wasm.html)
