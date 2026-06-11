# Lindblad Protocol — LLM Skill
> Install this skill to build applications on the Lindblad Protocol and Spectral Ledger.

## What is Lindblad Protocol?

Lindblad Protocol is a DePIN Layer 1 blockchain where physical hardware nodes mine PYCO tokens by proving their real-world existence through silicon identity, radio signals, and thermodynamic consensus. Each node derives its unique identity from SRAM PUF at boot, signs with P-256 ECDSA, and participates in consensus via a LoRa radio mesh governed by the Lindblad master equation.

Deployed on **Arbitrum One** mainnet with an EVM bridge to **Polygon**.

**Tagline:** The hardware decides. The physics guarantees. The chain records.

---

## Core Concepts

### The Spectral Ledger
The internal state layer of Lindblad. Gasless, instant settlement for PYCO, USDT, and USDC between LD-addresses. All internal transfers are free and finalized within the current epoch (~60 seconds).

### LD-Address Format
Every participant is identified by a physically unique LD-address derived from SRAM PUF:
```
Full address: LD-484C59B737BB65B2   (used in API calls)
Short ID:     LD484C5               (display only)
```

### PYCO Token
Native token of the Lindblad network. Mined by physical hardware nodes via Physical Coherence Verification (PCV-4). Max supply: 100,000,000 PYCO. Bridge fee: 0.1% on withdrawal (50% burned, 50% to node operators).

- **ERC-20 on Arbitrum One:** `0x16a69CcdA3865a23537d46055dC6564A2813C36B`

### Epochs
The network produces a new block approximately every 60 seconds. Each epoch increments the chain state and distributes mining rewards to active nodes.

---

## API Reference

**Base URL:** `https://lindblad.io`
All endpoints are public. No authentication required for read operations.

---

### Chain State

#### GET /chain
Returns current chain summary.
```json
{
  "blockCount": 25231,
  "latestEpoch": 37054,
  "earliestEpoch": 3,
  "digest": "4d7a8261",
  "totalMined": 1424613875000,
  "totalBurned": 0
}
```
> `totalMined` is in microunits — divide by 1e7 for PYCO display value.

#### GET /blocks?limit=N
Returns last N blocks (default 50).
```json
{
  "blocks": [
    {
      "ep": 37054,
      "ts": 1781135220,
      "prev": "eef41717",
      "curr": "4d7a8261",
      "nodes": 4,
      "txs": [],
      "mined": 1424613875000,
      "att": [
        { "id": "LD95A821D", "puf": "95A821D9AC091073", "cv": 675, "bal": 497218125000, "n": 15460 }
      ]
    }
  ]
}
```

#### GET /block/latest
Returns the most recent block with full attestation data.

---

### Accounts & Balances

#### GET /account/{LD-address}
Returns PYCO balance for a given LD-address.
```
GET /account/LD-EC6C1139FA3CD393
```
```json
{
  "address": "LD-EC6C1139FA3CD393",
  "balance": 48759.2865
}
```

#### GET /token-balance/{LD-address}?token={TOKEN}&network={NETWORK}
Returns balance of USDT, USDC, or PYCO on Arbitrum or Polygon.
```
GET /token-balance/LD-EC6C1139FA3CD393?token=USDT&network=arbitrum
```
```json
{
  "address": "LD-EC6C1139FA3CD393",
  "token": "USDT",
  "network": "arbitrum",
  "balance": 30000000
}
```
> Balances in microunits — divide by 1,000,000 for display value.

---

### Nodes

#### GET /node-list
Returns all registered nodes with status and last epoch.
```json
{
  "nodes": [
    {
      "id": "LD95A821D",
      "puf": "95A821D9AC091073",
      "lastSeen": 1781135284,
      "lastEpoch": 37054,
      "status": "active"
    }
  ],
  "count": 4
}
```
> `status`: `"active"` = last seen < 120 seconds ago, `"idle"` = otherwise.

#### GET /node-status?nodeId={id}
Returns whether a specific node is currently online.
```json
{ "nodeId": "LD95A821D", "online": true, "lastSeen": 1781135284 }
```

---

### Transfers

#### POST /send-token
Transfer tokens between LD-addresses. Free and instant for internal transfers.
```json
{
  "from": "LD-EC6C1139FA3CD393",
  "to": "LD-95A821D9AC091073",
  "token": "USDT",
  "amount": 10000000,
  "sig": "..."
}
```
```json
{ "ok": true, "txId": "abc123" }
```

#### POST /claim-mining
Convert accumulated mining balance to transferable PYCO for a node.
```json
{
  "ldAddress": "LD-EC6C1139FA3CD393",
  "nodeId": "LD95A821D"
}
```
```json
{
  "success": true,
  "ldAddress": "LD-EC6C1139FA3CD393",
  "nodeId": "LD95A821D",
  "claimed": 487342875000,
  "claimedPYCO": 48734.2875,
  "newBalance": 487592865000,
  "newBalancePYCO": 48759.2865
}
```

---

### Bridge

#### GET /node-challenge?nodeId={id}
Returns a signing challenge for hardware-validated withdrawal. Expires in 60 seconds.
```json
{ "challenge": "a3f8c2...", "expiresAt": 1781200060 }
```

#### POST /request-sign
Queues a hardware signing request on a physical node.
```json
{
  "nodeId": "LD95A821D",
  "challenge": "a3f8c2...",
  "amount": 10000000,
  "token": "USDT",
  "network": "arbitrum",
  "toAddress": "0xYourAddress"
}
```
```json
{ "ok": true, "signId": "sign_abc123" }
```

#### GET /get-sign?signId={id}
Poll for the result of a hardware signing request.
```json
{ "status": "complete", "r": "0x...", "s": "0x...", "v": 27 }
```
> `status` values: `pending`, `complete`, `expired`

#### POST /verify-pairing
Authorize a device to a node using a pairing code.
```json
{ "nodeId": "LD95A821D", "code": "KERX84", "ldAddr": "LD-EC6C1139FA3CD393" }
```
```json
{ "ok": true, "nodeId": "LD95A821D", "puf": "95A821D9AC091073" }
```

---

## Smart Contracts

### Arbitrum One (Chain ID: 42161)

| Contract | Address |
|----------|---------|
| PYCO ERC-20 | `0x16a69CcdA3865a23537d46055dC6564A2813C36B` |
| LindblabUSDT v3 | `0x7e0f53f04dDc48dFdc96DFE93606a73f0dCF56A3` |
| LindblabUSDC v3 | `0x1AfC80b30cBBE50E8aBb4585f53ff530c305d416` |
| Real USDT | `0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9` |
| Real USDC | `0xaf88d065e77c8cC2239327C5EDb3A432268e5831` |

### Polygon (Chain ID: 137)

| Contract | Address |
|----------|---------|
| LindblabUSDT v3 | `0x17c6d525A8D809fcBe78aBE4FCaE1F9ddb0b8fa8` |
| LindblabUSDC v3 | `0x9964c63Af739bf8b4702E243f904570b17F33ab4` |
| Real USDT | `0xc2132D05D31c914a87C6611C10748AEb04B58e8F` |
| Real USDC | `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359` |

> PYCO is deployed on Arbitrum One only and functions as the native token of the entire network regardless of which chain a bridge deposit originates from.

---

## Common Build Patterns

### Check wallet balance
```javascript
const res = await fetch('https://lindblad.io/account/LD-{PUF_HEX}');
const { balance } = await res.json(); // PYCO balance
```

### Fetch live network stats
```javascript
const chain = await fetch('https://lindblad.io/chain').then(r => r.json());
const pycoMined = chain.totalMined / 1e7; // convert microunits to PYCO
const epoch = chain.latestEpoch;
```

### List active nodes
```javascript
const { nodes } = await fetch('https://lindblad.io/node-list').then(r => r.json());
const active = nodes.filter(n => n.status === 'active');
```

### Display recent blocks
```javascript
const { blocks } = await fetch('https://lindblad.io/blocks?limit=10').then(r => r.json());
blocks.forEach(b => {
  console.log(`Epoch #${b.ep} | ${b.nodes} nodes | digest: ${b.curr}`);
});
```

---

## Research

| Document | DOI |
|----------|-----|
| Lindblad Protocol: Physics-Enforced Hardware Consensus | [10.5281/zenodo.20620319](https://doi.org/10.5281/zenodo.20620319) |
| PYCO: A Physical Coherence Token Emerging from the Lindblad Protocol | [10.5281/zenodo.20643853](https://doi.org/10.5281/zenodo.20643853) |

---

## Links

- Website: [lindblad.io](https://lindblad.io)
- Wallet: [lindblad.io/wallet](https://lindblad.io/wallet)
- Block Explorer: [lindblad.io/scan](https://lindblad.io/scan)
- Nodes: [lindblad.io/nodes](https://lindblad.io/nodes)
- GitHub: [github.com/lindblad-protocol](https://github.com/lindblad-protocol)
- Twitter/X: [@LindbladIO](https://x.com/LindbladIO)

---

*Lindblad Protocol — The hardware decides. The physics guarantees. The chain records.*
