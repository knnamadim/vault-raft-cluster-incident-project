# Vault Raft Cluster Postmortem

## Incident Summary

**Date:** [Insert Date]  
**Duration:** [Insert Duration]  
**Impact:** Temporary downtime of Vault cluster; authentication failures reported; leader elections caused instability.

**Nodes Involved:** V1, V2, V3 (Raft cluster)  
**Leader at Start:** V1  
**Leader After Restart:** V3

---

## Root Cause

1. Vault nodes restart in a **sealed state** and cannot participate in Raft communication until unsealed.
2. The leader (`V1`) attempted to connect to nodes that were still sealed.
3. Authentication failures occurred due to missing/unsealed tokens and potential TLS misconfiguration.
4. Adding a new node caused cluster disruption as all pods temporarily became unavailable, affecting quorum.

---

## Timeline

| Time | Event |
|------|-------|
| T0   | Vault leader V1 restarts |
| T1   | V1 tries to connect to V2 and V3 (sealed) |
| T2   | Authentication failures reported on V2 and V3 |
| T3   | V1 goes down, Raft elects V3 as leader |
| T4   | New node added; all pods temporarily unavailable |
| T5   | Original 3 nodes restarted sequentially, cluster recovered |

---

## Resolution

- Nodes restarted **sequentially** to maintain quorum.
- Auto-unseal enabled via KMS to prevent sealed state on restart.
- Verified Raft and TLS configuration to ensure authentication works between nodes.
- Implemented pod anti-affinity to prevent single-node outages.
- Monitoring enabled for leader elections and sealed states.

---

## Lessons Learned

1. Never restart all Vault nodes simultaneously.
2. Auto-unseal is critical for HA Vault clusters.
3. Always verify Raft/TLS configuration when adding nodes.
4. Staging test is essential before production changes.

---

## Action Items

- [ ] Enable auto-unseal on all Vault clusters.
- [ ] Implement sequential rolling restarts in deployment pipelines.
- [ ] Add Prometheus/Grafana monitoring for Vault leader and seal states.
- [ ] Document node addition procedures and anti-affinity rules.
