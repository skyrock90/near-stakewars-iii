##### 节点信息：

Mainnet：https://near.org/stakewars/

Wallet：https://wallet.shardnet.near.org/

Explorer：https://explorer.shardnet.near.org/



##### 服务器要求：

| 硬件    | 配置       |
| ------- | ---------- |
| CPU     | 4-Core CPU |
| RAM     | 8GB DDR4   |
| Storage | 500GB SSD  |



##### 节点搭建：

- 安转node.js和npm

  ```
  sudo apt update && sudo apt upgrade -y
  curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
  sudo apt install build-essential nodejs
  PATH="$PATH"
  
  node -v
  npm -v
  ```

- 安转NEAR-CLI 

  ```
  sudo npm install -g near-cli
  ```

- 配置默认启动shardnet

  ```
  echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
  ```

- 检查服务器配置是否符合要求

  ```
  lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
    && echo "Supported" \
    || echo "Not supported"
  ```

- 安装开发工具

  ```
  sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
  sudo apt install python3-pip
  USER_BASE_BIN=$(python3 -m site --user-base)/bin
  export PATH="$USER_BASE_BIN:$PATH"
  ```

- 安装Building env

  ```
  sudo apt install clang build-essential make
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  source $HOME/.cargo/env
  ```

- clone nearcore

  ```
  git clone https://github.com/near/nearcore
  cd nearcore
  git fetch
  git checkout 8448ad1ebf27731a43397686103aa5277e7f2fcf
  ```

- 执行build

  ```
  cargo build -p neard --release --features shardnet
  cd target/release/neard
  sudo cp neard /usr/local/bin
  ```

- 初始化near

  ```
  neard --home ~/data/.near init --chain-id shardnet --download-genesis
  ```

- 替换config.json

  ```
  cd /data/.near
  rm config.json
  wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
  ```

- 下载genesis.json

  ```
  sudo apt-get install awscli -y
  cd /data/.near
  wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
  ```

- 运行节点

  ```
  neard --home /data/.near run
  ```





##### 创建Validator：

- 创建钱包：https://wallet.shardnet.near.org/

  ```
  hj-pos.shardnet.near
  ```

- 登录near

  ```
  near login
  ```

- 生成Generate  Key

  ```
  near generate-key hj-pos
  cp ~/.near-credentials/shardnet/hj-pos.json ~/.near/validator_key.json
  ```

- 修改validator_key.json文件

  ```
  ~/.near-credentials/shardnet$ ls
  hj-pos.json  hj-pos.shardnet.near.json
  
  account_id="hj-pos.factory.shardnet.near"
  Change private_key to secret_key
  ```

- 等区块同步完成后创建Validator

  ```
  near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "hj-pos", "owner_id": "hj-pos.shardnet.near", "stake_public_key": "ed25519:HataKZh39EA2qmhsixJ4c4fb8aGKpA7a3f81qKqoC2wG", "reward_fee_fraction": {"numerator": 2, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="hj-pos.shardnet.near" --amount=30 --gas=300000000000000
  ```

- 更新commission

  ```
  near call hj-pos.factory.shardnet.near update_reward_fee_fraction '{"reward_fee_fraction": {"numerator": 2, "denominator": 100}}' --accountId hj-pos.shardnet.near --gas=300000000000000
  ```

- 质押near

  ```
  near call hj-pos.factory.shardnet.near deposit_and_stake --amount 1020 --accountId hj-pos.shardnet.near --gas=300000000000000
  ```

- ping

  ```
  near call hj-pos.factory.shardnet.near ping '{}' --accountId hj-pos.shardnet.near --gas=300000000000000
  ```

  



##### 查看Validator信息：

- logs

  ```
  journalctl -n 100 -f -u neard | ccze -A
  ```

- version

  ```
  curl -s http://127.0.0.1:3030/status | jq .version
  neard --version
  neard (release trunk) (build crates-0.14.0-218-g8448ad1eb) (rustc 1.62.0) (protocol 100) (db 31)
  ```

- Delegators

  ```
  near view hj-pos.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId hj-pos.shardnet.near
  
  View call: hj-pos.factory.shardnet.near.get_accounts({"from_index": 0, "limit": 10})
  [
    {
      account_id: 'hj-pos.shardnet.near',
      unstaked_balance: '2',
      staked_balance: '17030005178700155577121983912',
      can_withdraw: true
    },
    {
      account_id: '0000000000000000000000000000000000000000000000000000000000000000',
      unstaked_balance: '0',
      staked_balance: '2224038335134603430998',
      can_withdraw: true
    },
    {
      account_id: 'zilinear2.shardnet.near',
      unstaked_balance: '1',
      staked_balance: '1999999999999999999999999',
      can_withdraw: true
    }
  ]
  ```

- Kicked Reasion

  ```
  curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("hj-pos"))' | jq .reason
  ```

- Check Blocks Produced 

  ```
  curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id | contains ("hj-pos"))'
  
  {"account_id":"hj-pos.factory.shardnet.near","is_slashed":false,"num_expected_blocks":186,"num_expected_chunks":2681,"num_produced_blocks":159,"num_produced_chunks":2450,"public_key":"ed25519:HataKZh39EA2qmhsixJ4c4fb8aGKpA7a3f81qKqoC2wG","shards":[2],"stake":"15000005503110319595000000000"}
  ```

  





##### Monitor:

我们使用Grafana监控一些节点运行信息，块高、near运行时间、near版本、磁盘空间等等