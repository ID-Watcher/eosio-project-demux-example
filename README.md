# eosio-hackathon-starter

## Getting Started

### Prerequisites

#### Nodeos
Follow the instructions on the [Developer Portal for the Docker Quickstart](https://developers.eos.io/eosio-nodeos/docs/docker-quickstart) to run nodeos, the daemon to run a EOSIO node
    
#### Cleos
Make sure to also set the [alias for cleos](https://developers.eos.io/eosio-nodeos/docs/docker-quickstart#section-cleos), the command line interface to interact with the blockchain and manage wallets. The tutorial will be using cleos

### Contract Deployment

1.  Create a wallet
    * Save the password. You will need it to unlock your wallet
    ```bash
    # cleos wallet create -n <Wallet Name>
    cleos wallet create -n blog.platform
    ```
2.  Create keys - owner and active keys
    * Save the private keys. You will need them to add the keys to the wallet
    ```bash
    cleos create key
    cleos create key
    ```
3.  Import keys into wallet
    ```bash
    # cleos wallet import -n <Wallet Name> --private-key <Private Key>
    cleos wallet import -n blog.platform --private-key <Owner Private Key> 
    cleos wallet import -n blog.platform --private-key <Active Private Key>
    ``` 
4.  Import the existing eosio account private key into the wallet, since accounts can only be created with other accounts
    ```bash
    cleos wallet import -n blog.platform --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
    ```
5.  Create an account for the blog smart contract
    * Remember that accounts have no relation with keys in wallets, aside from when you sign transactions to make calls to your contract with keys
    ```bash
    # cleos create account eosio <Account> <Owner Public Key> <Active Public Key>
    cleos create account eosio blog <Owner Public Key> <Active Public Key>
    ```
5.  Compile contract to webassembly with eosiocpp
    * set alias 
    ```bash
    alias eosiocpp='docker exec -it eosio /opt/eosio/bin/eosiocpp'
    ```
    * Create a folder under /tmp/eosio/data/contracts called blog and copy the entire contents of the [contract directory](https://github.com/EOSIO/eosio-hackathon-starter/blob/master/contract) over to this new folder
    * This folder is mounted onto the docker container, which will allow you to compile your contract using the eosiocpp executable in the container
    ```bash
    # eosiocpp -o <Target> <Smart Contract File>
    eosiocpp -o mnt/dev/data/contracts/blog/blog.wast mnt/dev/data/contracts/blog/blog.cpp
    ```
    * Notice the directories are referencing the paths on the docker container to the mounted directory (/tmp/eosio) from your local filesystem
6.  Generate ABI file
    ```bash
    # eosiocpp -g <Target> <Smart Contract File>
    eosiocpp -g mnt/dev/data/contracts/blog/blog.abi mnt/dev/data/contracts/blog/blog.cpp
    ```
7.  Deploy contract
    ```bash
    # cleos set contract <Account> <Path to contract folder> <Path to .wast file> <Path to .abi file>
    cleos set contract blog mnt/dev/data/contracts/blog/ mnt/dev/data/contracts/blog/blog.wast mnt/dev/data/contracts/blog/contract/blog.abi
    ```
8.  Create a blog post
    ```bash
    # cleos push action <Account> <Action name> '<Data>' -p <Account>@active
    cleos push action blog createpost '["blog", "Sample Blog Title", "Sample blog content blah blah", "misc"]' -p blog@active
    ```
9.  Check that the blog post was added to the table
    ```bash
    # cleos get table <Contract> <Scope> <Table>
    cleos get table blog blog post
    ```

#### Frontend Config

1.  Update the root [.env](https://github.com/EOSIO/eosio-hackathon-starter/blob/master/frontend/.env) with your account name, generated private key and chain ID like below
    ```bash
    REACT_APP_EOS_ACCOUNT=
    REACT_APP_EOS_PRIVATE_KEY=
    REACT_APP_EOS_CHAIN_ID=

    REACT_APP_EOS_CONTRACT_ACCOUNT=blog

    REACT_APP_EOS_HTTP_URL=http://localhost:8888
    ```
    * You can get the chainId via the command:
    ```bash
    cleos get info
    ```
2.  Start the react app
    ```bash
    cd frontend
    npm start
    ```