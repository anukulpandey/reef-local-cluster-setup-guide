# polkadot-local-cluster-setup-guide

Built the `polkadot-sdk` binaries

Generated node-key which is used by lib p2p to assign peer id.

```
./polkadot key generate-node-key
```

got response 

```
12D3KooWBiLBQ2m2hJGvymjLKnxsQtmcgrV83rPuhyShtvoUUmQB
971bc2d4efa1ca6d039b9237ca5cdc9de707362cc17ba8af5242442a7c92b2f9%
```

now generating the custom spec file

for that installing the `custom-spec-builder`:

```
cargo install --git https://github.com/paritytech/polkadot-sdk --force staging-chain-spec-builder
```

used this command to generate `plain.json`:


