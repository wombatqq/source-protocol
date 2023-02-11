# RPC node & State sync endpoint for Source Protocol Testnet
# RPC node (hosted by wombat#7690)

RPC endpoint with "default" pruning: http://207.180.212.166:26657/

API endpoint: http://207.180.212.166:1317/

gRPC endpoint: http://207.180.212.166:9090/

# State sync guide

1. Follow official documentation in order to setup a full node in Source Protocol Testnet: https://docs.sourceprotocol.io/nodes-and-validators/sourced-installation-and-setup.
Make sure you use the actual version of ``sourced``.
2. Set up a service to run ``sourced`` binary.
3. Make sure it is stopped and reset database:
```
sudo systemctl stop sourced.service
sourced unsafe-reset-all
```
4. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
RPC_PEER="529d5e582ebf176a84df9698314037112d6061cc@207.180.212.166:26656"
SNAP_RPC="http://207.180.212.166:26657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.sourced/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$RPC_PEER\"/" $HOME/.sourced/config/config.toml
```
5. Run the service and wait for a few minutes until snaphost is fetched and applied:
```
sudo systemctl restart sourced.service && sudo journalctl -u sourced.service -f -o cat
```
