# How to Change ChainID in BSC Private Networks

## Problem

BSC private networks always return chainId `714` (0x2ca) even when you configure a different chainId in genesis.json or .env files.

## Root Cause

ChainId 714 is **hardcoded** in BSC source code and overrides your configuration.

## Solution: Update 3 Locations

### 🎯 **Location 1: BSC Source Code (Most Important)**

Edit `bsc/params/config.go` line ~251:

```go
// BEFORE
RialtoChainConfig = &ChainConfig{
    ChainID: big.NewInt(714),  // <- Change this!

// AFTER
RialtoChainConfig = &ChainConfig{
    ChainID: big.NewInt(19999),  // Your desired chainId
```

Then rebuild BSC:

```bash
cd bsc && make geth && cp build/bin/geth ../bin/geth
```

### 🎯 **Location 2: Environment File**

Edit `.env`:

```bash
CHAIN_ID=19999  # Must match BSC source code
```

### 🎯 **Location 3: Network Configuration**

Edit `config.toml`:

```toml
NetworkId = 19999  # Must match BSC source code
```

## Deploy Changes

```bash
# Stop and clean everything
bash bsc_cluster.sh stop
rm -rf .local

# Start fresh network
bash bsc_cluster.sh reset
```

## Verify Result

Test the chainId:

```bash
# Wait for network startup
sleep 15

# Check chainId via RPC
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  http://localhost:8545

# Expected: {"jsonrpc":"2.0","id":1,"result":"0x4e1f"}  # 0x4e1f = 19999
```

## Quick Checklist

- [ ] **Modified BSC hardcode** in `bsc/params/config.go` line ~251
- [ ] **Updated `.env`** with `CHAIN_ID=19999`
- [ ] **Updated `config.toml`** with `NetworkId = 19999`
- [ ] **Rebuilt BSC** with `cd bsc && make geth`
- [ ] **Copied binary** with `cp build/bin/geth ../bin/geth`
- [ ] **Cleaned database** with `rm -rf .local`
- [ ] **Restarted network** with `bash bsc_cluster.sh reset`
- [ ] **Verified chainId** with RPC call

## Common Issues

**Still getting 714?**

- Check you're using the new binary: `./bin/geth version`
- Ensure all 3 locations have the same chainId
- Clean database completely: `rm -rf .local`

**Compilation error?**

- Check Go version: `go version` (need 1.21+)
- Clean and rebuild: `cd bsc && make clean && make geth`

---

**Important:** All 3 locations must have the **same chainId value**!
