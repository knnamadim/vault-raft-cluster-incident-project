# Vault Raft Troubleshooting Steps

## 1. Identify Leader
vault status
vault operator raft list-peers

## 2. Check logs 
kubectl logs vault-pod-name

- Look for messages like failed to authenticate or sealed.

## 3. Sequential Node Restart
- Restart nodes one by one.
- Unseal each node before restarting the next.
- Ensure quorum is maintained.

## 4. Verify Raft and TLS Config
- Ensure node_id is unique.
- Validate TLS certs are trusted by all nodes.

## 5. Adding a New Node
vault operator raft join https://leader-node:8200
vault operator raft list-peers

Note: Only join new nodes when quorum is stable.