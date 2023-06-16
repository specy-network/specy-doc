## QuickStart
This document introduces how to quickly implement deployment testing for NFT Rewards scenarios.

### 1. Related Components
The operation of specy-network relies on the following components. Please refer to the relevant project documentation before deployment to complete the installation.
- [firehose-cosmos](https://github.com/graphprotocol/firehose-cosmos/tree/v0.5.0)
- [graphnode](https://github.com/graphprotocol/graph-node/tree/v0.27.0) 
- [specy-engine](https://github.com/specy-network/specy-engine)
- node >= 16.0.0
- npm >= 8.0.0
- docker
- docker-compose
- go >= 1.20.4


Copy the compiled executable file above to the /usr/local/bin/ directory.

### 2. irishub-specy install

Download and install irihub with spec-network module integration.
```shell
git clone https://github.com/specy-network/irishub.git -b feature/hackathon-specy
cd irishub && make install
```

### 3. scheduler install

```shell
git clone https://github.com/specy-network/specy-scheduler

cd specy-scheduler 
```

According to the actual situation, configure the config.yaml content in the main directory. In this test case, fill in its content as follows.

```yaml
chain_id: ibc-2
chain_binary_location: /usr/local/bin/iris
engine_node_address: 127.0.0.1:50051
home_dir: $pwd/examples/iris-demo/data/ibc-2
```
Compile specy-scheduler

```shell
make install
```

### 4. Interchain nft rewards test
- Create a basic environment

    Start two irishub chains (the chain id is ibc-1 and ibc-2 respectively), and establish the ability of nft cross Chain transfer transfer between chains based on ibc.
    ```shell
    cd examples/iris-demo
    bash dev-env.sh
    ```
- Deploy data indexing service

    After waiting for the establishment of the inter chain IBC channel to be completed and generating over 100 blocks, use a script to deploy the graphnode service and install the corresponding subgraph.

    ```shell
    bash use-case-test.sh graphnode
    bash use-case-test.sh manifest
    ```
- Deploy specy-engine service

    Reference https://github.com/specy-network/specy-engine Complete engine deployment

- Interchain nft transfer

    - nft issue and mint

    Create an nft denom on the ibc-1 chain using a script, while also creating two nft tokens
    ```shell
    bash use-case-test.sh issue
    base use-case-test.sh mint
    #Modify the use-case-test.sh script to modify the token id
    ```

    - Nft cross chain transfer
    
    Transfer the nft created on the ibc-1 chain to the ibc-2 chain using scripts based on the nft transfer capability

    ```shell
    bash use-case-test.sh ibc-nft-transfer
    #Modify the recipient of token id and ibc-2 chain as user and validator, respectively
    ```

- Add rewad List

    Which NFT categories will be rewarded after cross chain addition on the chain

    ```shell
    bash use-case-test.sh set-reward-list
    ```

- Create executor

    Creating an executor on the ibc-2 chain using a script
    ```shell
    bash use-case-test create-executor
    ```
- Create task

    Creating tasks on the ibc-2 chain using scripts
    ```shell
    bash use-case-test create-task
    ```

After waiting for the specified time of the task, the spec network will execute the corresponding task on the chain and add the record of cross Chain transfer transfer rewards to the rewards module.