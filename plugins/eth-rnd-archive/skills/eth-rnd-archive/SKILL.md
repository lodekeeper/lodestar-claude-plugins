---
name: eth-rnd-archive
description: Search and track the Ethereum R&D Discord Archive. Use when looking up protocol discussions, finding what researchers said about a topic, tracking channel activity, or monitoring Ethereum development conversations across consensus-dev, execution-dev, networking, and 100+ other R&D channels.
---

# Eth R&D Archive Tracker

Search and track discussions from the [Ethereum R&D Discord Archive](https://github.com/ethereum/eth-rnd-archive) — a machine-readable archive of all Eth R&D Discord channels, updated hourly by EF DevOps.

## Setup

Clone the archive repository locally:

```bash
git clone https://github.com/ethereum/eth-rnd-archive.git ~/eth-rnd-archive
```

The repo is ~600MB and contains daily JSON exports of every channel.

## Archive Structure

```
eth-rnd-archive/
├── consensus-dev/
│   ├── 2026-02-25.json
│   ├── 2026-02-24.json
│   └── _threads/
│       └── thread-name/
│           └── 2026-02-25.json
├── execution-dev/
│   └── ...
├── epbs/
│   └── ...
└── ... (115+ channels)
```

### Message Format

```json
{
  "author": "username",
  "category": "Discord category",
  "parent": "thread parent (empty if top-level)",
  "content": "message text",
  "created_at": "ISO8601 timestamp",
  "attachments": [...]
}
```

## Key Channels

### Core Protocol Development
- `consensus-dev` — CL protocol development
- `execution-dev` — EL protocol development
- `allcoredevs` — cross-client coordination
- `specifications` — spec discussions
- `apis` — Beacon/Engine API discussions
- `client-development` — client team discussions

### Active Research Topics
- `epbs` — enshrined PBS design
- `inclusion-lists` — inclusion list design
- `shorter-slot-times` — slot time reduction research
- `data-availability-sampling` — DAS / PeerDAS
- `l1-zkevm` — L1 zkEVM / CK EVM
- `l1-zkevm-protocol` — zkEVM protocol details

### Networking & Testing
- `networking` — general networking
- `libp2p` — libp2p protocol
- `peerdas-testing` — PeerDAS test coordination
- `peerdas-devnet-alerts` — PeerDAS devnet status

### Other Notable Channels
- `cryptography` — crypto research
- `post-quantum` — post-quantum transition
- `formal-methods` — formal verification
- `account-abstraction` — AA/ERC-4337
- `light-clients` — light client protocol
- `beacon-network` — beacon chain networking
- `portal-network` — portal network

## How to Search

### Search a specific channel for a topic

```bash
cd ~/eth-rnd-archive

# Search recent files in a channel
grep -rl "keyword" consensus-dev/ | sort -r | head -10

# Read a specific day's messages
cat consensus-dev/2026-02-25.json | python3 -c "
import json, sys
msgs = json.load(sys.stdin)
for m in msgs:
    if 'keyword' in m.get('content', '').lower():
        print(f\"[{m['created_at']}] {m['author']}: {m['content'][:200]}\")
"
```

### Search across all channels

```bash
# Find which channels discussed a topic
grep -rl "PeerDAS" --include="*.json" ~/eth-rnd-archive/ | \
  sed 's|.*/eth-rnd-archive/||' | cut -d'/' -f1 | sort -u
```

### Get recent activity in a channel

```bash
# Last 3 days of a channel
for d in $(seq 0 2); do
  DATE=$(date -u -d "$d days ago" +%Y-%m-%d 2>/dev/null || date -u -v-${d}d +%Y-%m-%d)
  FILE="$HOME/eth-rnd-archive/consensus-dev/$DATE.json"
  if [ -f "$FILE" ]; then
    COUNT=$(python3 -c "import json; print(len(json.load(open('$FILE'))))")
    echo "$DATE: $COUNT messages"
  fi
done
```

## Checking for New Messages

### Using the check script

The `scripts/check-updates.sh` script pulls latest changes and reports new messages in tracked channels:

```bash
# Check for new messages since last check
bash scripts/check-updates.sh

# Check a specific date
bash scripts/check-updates.sh 2026-02-25
```

The script:
1. Pulls the latest archive data
2. Diffs against the last-checked commit (stored in `scripts/state.json`)
3. Outputs changed files in tracked channels as JSON
4. Updates the state file

### Configure tracked channels

Edit `scripts/config.json` to set which channels to monitor:

```json
{
  "channels": [
    "epbs",
    "consensus-dev",
    "allcoredevs",
    "specifications",
    "data-availability-sampling"
  ],
  "checkIntervalMinutes": 60,
  "repoPath": "~/eth-rnd-archive"
}
```

## Thread Support

Threads are stored in `_threads/` subdirectories within each channel:

```
epbs/_threads/make it two clients/2026-02-23.json
l1-zkevm/_threads/Proof orchestration/2026-02-23.json
```

Thread messages have `"parent": "<thread name>"` set. The check script scans these automatically.

## Tips

- **Pull before searching** — `cd ~/eth-rnd-archive && git pull` to get the latest messages
- **Threads contain the good stuff** — many detailed technical discussions happen in threads, not the main channel
- **Author search** — find everything a specific researcher said: `grep -rl '"author": "vbuterin"' --include="*.json"`
- **Date range search** — use `find` with `-newer` to limit to recent files
- **Large channels** — `consensus-dev` and `execution-dev` are the most active; start searches there
- **Archive lag** — the archive is updated hourly, so there's up to 1 hour delay from live Discord
