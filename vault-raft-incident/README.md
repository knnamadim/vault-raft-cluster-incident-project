# Vault Raft Cluster Incident

## Overview

This project documents a real-world incident in a Vault Raft cluster (3-node setup) where leader election, node restarts, and adding a new node caused cluster instability and authentication failures.

The Vault cluster consists of 3 nodes (V1, V2, V3) using Raft storage.

### Problem Summary

- Original leader `V1` restarted while `V2` and `V3` were sealed.
- When `V1` went down, Raft elected `V3` as the new leader.
- Other nodes reported `failed to authenticate` errors.
- Adding a new node caused all pods to temporarily go down until the original 3 nodes were up.

---

## Steps to Reproduce

1. Deploy a 3-node Vault cluster with Raft storage.
2. Restart the leader node while other nodes are sealed.
3. Observe Raft leader election and authentication failures.
4. Add a new node using `vault operator raft join` and observe cluster disruption.

---

## Resolution

1. **Sequential Restarts**: Restart Vault nodes one at a time to maintain quorum.
2. **Auto-Unseal Configuration**: Use a KMS/HSM to automatically unseal nodes after restart.
3. **Verify Raft/TLS Config**: Ensure correct TLS and Raft node IDs.
4. **Pod Distribution**: Use anti-affinity rules to prevent all nodes from landing on the same host.
5. **Logging and Monitoring**: Enable detailed Vault logs and monitor leader elections.
6. **Add Nodes Carefully**: Join new nodes only when a quorum is operational.

---

## References

- [Vault Raft Storage Documentation](https://www.vaultproject.io/docs/concepts/ha/raft)
- [Vault Auto-Unseal Documentation](https://www.vaultproject.io/docs/secrets/auto-unseal)
