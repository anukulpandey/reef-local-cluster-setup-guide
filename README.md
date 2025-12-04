# reef-local-cluster-setup-guide

remove all the dirs

```
rm -rf /tmp/validator1 /tmp/validator2 /tmp/validator3 /tmp/bootnode /tmp/validator1.txt /tmp/validator2.txt /tmp/validator3.txt /tmp/v1_seed.txt /tmp/v2_seed.txt /tmp/v3_seed.txt /tmp/v1_addr.txt /tmp/v2_addr.txt /tmp/v3_addr.txt /tmp/bootnode_peer_id.txt /tmp/bootnode_node_key.txt /tmp/v1_node_key.txt /tmp/v2_node_key.txt /tmp/v3_node_key.txt /tmp/local-chain-spec.json /tmp/local-chain-spec-updated.json /tmp/local-chain-spec-raw.json
```

generate 3 validators

```
./target/release/reef-node key generate --scheme Sr25519 --output-type json > /tmp/validator1.txt
```

```
./target/release/reef-node key generate --scheme Sr25519 --output-type json > /tmp/validator2.txt
```

```
./target/release/reef-node key generate --scheme Sr25519 --output-type json > /tmp/validator3.txt
```

generate keys

```
./target/release/reef-node key generate-node-key --chain local > /tmp/bootnode_node_key.txt
```

```
./target/release/reef-node key generate-node-key --chain local > /tmp/v1_node_key.txt
```

```
./target/release/reef-node key generate-node-key --chain local > /tmp/v2_node_key.txt
```

```
./target/release/reef-node key generate-node-key --chain local > /tmp/v3_node_key.txt
```

generate initial chain spec

```
./target/release/reef-node build-spec --chain testnet-new --disable-default-bootnode > /tmp/local-chain-spec.json
```

insert keys

```
bash -c '
    set -e;
    V1_SEED=$(grep -o "\"secretSeed\": \"[^\"]*\"" /tmp/validator1.txt | cut -d"\"" -f4);
    V1_ADDR=$(grep -o "\"ss58Address\": \"[^\"]*\"" /tmp/validator1.txt | cut -d"\"" -f4);
    V2_SEED=$(grep -o "\"secretSeed\": \"[^\"]*\"" /tmp/validator2.txt | cut -d"\"" -f4);
    V2_ADDR=$(grep -o "\"ss58Address\": \"[^\"]*\"" /tmp/validator2.txt | cut -d"\"" -f4);
    V3_SEED=$(grep -o "\"secretSeed\": \"[^\"]*\"" /tmp/validator3.txt | cut -d"\"" -f4);
    V3_ADDR=$(grep -o "\"ss58Address\": \"[^\"]*\"" /tmp/validator3.txt | cut -d"\"" -f4);

    V1_BABE=$(./target/release/reef-node key inspect --scheme Sr25519 "$V1_SEED//babe" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V1_GRAN=$(./target/release/reef-node key inspect --scheme Ed25519 "$V1_SEED//grandpa" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V1_IMON=$(./target/release/reef-node key inspect --scheme Sr25519 "$V1_SEED//im_online" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V1_AUDI=$(./target/release/reef-node key inspect --scheme Sr25519 "$V1_SEED//authority_discovery" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);

    V2_BABE=$(./target/release/reef-node key inspect --scheme Sr25519 "$V2_SEED//babe" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V2_GRAN=$(./target/release/reef-node key inspect --scheme Ed25519 "$V2_SEED//grandpa" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V2_IMON=$(./target/release/reef-node key inspect --scheme Sr25519 "$V2_SEED//im_online" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V2_AUDI=$(./target/release/reef-node key inspect --scheme Sr25519 "$V2_SEED//authority_discovery" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);

    V3_BABE=$(./target/release/reef-node key inspect --scheme Sr25519 "$V3_SEED//babe" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V3_GRAN=$(./target/release/reef-node key inspect --scheme Ed25519 "$V3_SEED//grandpa" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V3_IMON=$(./target/release/reef-node key inspect --scheme Sr25519 "$V3_SEED//im_online" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);
    V3_AUDI=$(./target/release/reef-node key inspect --scheme Sr25519 "$V3_SEED//authority_discovery" --output-type json 2>/dev/null | grep -o "\"ss58Address\": \"[^\"]*\"" | cut -d"\"" -f4);

    jq "
        .genesis.runtimeGenesis.patch.balances.balances +=
          [[\"$V1_ADDR\", 100000000000000000000000000],
           [\"$V2_ADDR\", 100000000000000000000000000],
           [\"$V3_ADDR\", 100000000000000000000000000]]
        |
        .genesis.runtimeGenesis.patch.session.keys =
          [[\"$V1_ADDR\", \"$V1_ADDR\", {\"authority_discovery\": \"$V1_AUDI\", \"babe\": \"$V1_BABE\", \"grandpa\": \"$V1_GRAN\", \"im_online\": \"$V1_IMON\"}],
           [\"$V2_ADDR\", \"$V2_ADDR\", {\"authority_discovery\": \"$V2_AUDI\", \"babe\": \"$V2_BABE\", \"grandpa\": \"$V2_GRAN\", \"im_online\": \"$V2_IMON\"}],
           [\"$V3_ADDR\", \"$V3_ADDR\", {\"authority_discovery\": \"$V3_AUDI\", \"babe\": \"$V3_BABE\", \"grandpa\": \"$V3_GRAN\", \"im_online\": \"$V3_IMON\"}]]
        |
        .genesis.runtimeGenesis.patch.staking.invulnerables =
          [\"$V1_ADDR\", \"$V2_ADDR\", \"$V3_ADDR\"]
        |
        .genesis.runtimeGenesis.patch.staking.stakers =
          [[\"$V1_ADDR\", \"$V1_ADDR\", 1000000000000000000000000, \"Validator\"],
           [\"$V2_ADDR\", \"$V2_ADDR\", 1000000000000000000000000, \"Validator\"],
           [\"$V3_ADDR\", \"$V3_ADDR\", 1000000000000000000000000, \"Validator\"]]
    " /tmp/local-chain-spec.json > /tmp/local-chain-spec-updated.json
'
```

convert to raw chain spec

```
./target/release/reef-node build-spec --chain /tmp/local-chain-spec-updated.json --disable-default-bootnode --raw > /tmp/local-chain-spec-raw.json
```

insert validator 1 session keys

```
./target/release/reef-node key insert --base-path /tmp/validator1 --chain=/tmp/local-chain-spec-raw.json --scheme Sr25519 --suri "$V1_SEED//babe" --key-type babe

./target/release/reef-node key insert --base-path /tmp/validator1 --chain=/tmp/local-chain-spec-raw.json --scheme Ed25519 --suri "$V1_SEED//grandpa" --key-type gran

./target/release/reef-node key insert --base-path /tmp/validator1 --chain=/tmp/local-chain-spec-raw.json --scheme Sr25519 --suri "$V1_SEED//im_online" --key-type imon

./target/release/reef-node key insert --base-path /tmp/validator1 --chain=/tmp/local-chain-spec-raw.json --scheme Sr25519 --suri "$V1_SEED//authority_discovery" --key-type audi
```

start bootnode

```
./target/release/reef-node \
  --base-path /tmp/bootnode \
  --chain /tmp/local-chain-spec-raw.json \
  --port 30335 \
  --node-key-file /tmp/bootnode_node_key.txt \
  --name Bootnode
```

start validators

```
./target/release/reef-node \
  --base-path /tmp/validator1 \
  --chain /tmp/local-chain-spec-raw.json \
  --port 30333 \
  --rpc-port 9944 \
  --node-key-file /tmp/v1_node_key.txt \
  --bootnodes /ip4/127.0.0.1/tcp/30335/p2p/<BOOTNODE_PEER_ID> \
  --validator --rpc-cors all --rpc-methods Unsafe --rpc-external \
  --name Validator1Node
```

