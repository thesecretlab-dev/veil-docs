# VEIL Child Node Bootstrap Runbook

Updated: 2026-02-23

## Goal

Create a repeatable, evidence-tracked dependency bootstrap for every new child validator host before node launch.

## Standard Dependencies (Per Child Host)

1. OS packages:
   - `ca-certificates`
   - `curl`
   - `git`
   - `jq`
   - `build-essential`
   - `pkg-config`
   - `libssl-dev`
   - `unzip`
   - `tar`
   - `xz-utils`
   - `tmux`
2. Go toolchain:
   - `go1.23.7 linux/amd64`
3. Avalanche CLI:
   - `avalanche-cli` (currently `1.9.6`)
4. Persistent shell path:
   - `/usr/local/go/bin`
   - `/root/bin`

## Bootstrap Command Block (Run on Child Host)

```bash
set -euo pipefail
export DEBIAN_FRONTEND=noninteractive

apt-get update
apt-get install -y --no-install-recommends \
  ca-certificates curl git jq build-essential pkg-config libssl-dev \
  unzip tar xz-utils tmux

if ! command -v go >/dev/null 2>&1; then
  curl -fsSL https://go.dev/dl/go1.23.7.linux-amd64.tar.gz -o /tmp/go.tgz
  rm -rf /usr/local/go
  tar -C /usr/local -xzf /tmp/go.tgz
  ln -sf /usr/local/go/bin/go /usr/local/bin/go
  ln -sf /usr/local/go/bin/gofmt /usr/local/bin/gofmt
fi

if ! command -v avalanche >/dev/null 2>&1; then
  curl -fsSL https://raw.githubusercontent.com/ava-labs/avalanche-cli/main/scripts/install.sh | bash
  ln -sf /root/bin/avalanche /usr/local/bin/avalanche
fi

for line in \
  'export PATH=/usr/local/go/bin:/root/bin:$PATH' \
  'export GOROOT=/usr/local/go'
do
  grep -F "$line" /root/.bashrc >/dev/null 2>&1 || echo "$line" >> /root/.bashrc
  grep -F "$line" /root/.profile >/dev/null 2>&1 || echo "$line" >> /root/.profile
done
```

## Verification Checklist

Run and record:

```bash
which go
go version
which avalanche
avalanche --version
gcc --version | head -n1
make --version | head -n1
tmux -V
```

Expected:

- `go` resolves and reports `go1.23.7`
- `avalanche` resolves and reports a valid CLI version
- build toolchain commands succeed

## Operator Reachability Bridge (Required if Operator Is Not Public)

When operator ports are not directly reachable from child hosts, run Cloudflare Access TCP sidecars on the child host.

Bridge command block:

```bash
set -euo pipefail

if ! command -v cloudflared >/dev/null 2>&1; then
  curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o /usr/local/bin/cloudflared
  chmod +x /usr/local/bin/cloudflared
fi

# staking bridge (operator tcp/9651 -> child 127.0.0.1:29651)
nohup cloudflared access tcp \
  --hostname veil-staking.thesecretlab.app \
  --url 127.0.0.1:29651 \
  >/tmp/veil-staking-access-29651.log 2>&1 &

# raw rpc bridge (operator tcp/9660 -> child 127.0.0.1:29660)
nohup cloudflared access tcp \
  --hostname veil-rpc-tcp.thesecretlab.app \
  --url 127.0.0.1:29660 \
  >/tmp/veil-rpc-access-29660.log 2>&1 &
```

Optional helper (first child baseline):

- `/root/veil-operator-bridge.sh` supports `ensure|status|restart` for the two bridge sidecars.

Bridge verification:

```bash
ss -ltnp | grep -E '29651|29660'
timeout 8 openssl s_client -connect 127.0.0.1:29651 -brief < /dev/null
curl -sS -m 8 http://127.0.0.1:29660/ext/health/readiness
```

Expected:

- `127.0.0.1:29651` and `127.0.0.1:29660` are listening under `cloudflared`.
- staking bridge completes TLS handshake on `29651`.
- RPC readiness over `29660` returns healthy JSON from operator node.

## Evidence Requirements (Per Child)

1. Save bootstrap report JSON on host:
   - `/root/veil-child-node-bootstrap.json`
2. Mirror child result into central tracker:
   - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`
3. Update latest pointer:
   - `evidence-bundles/child-node-bootstrap/latest.txt`

## First Child Baseline

- Sandbox ID: `d2fe48a2a6465322e963a0a11c30ead3`
- Report path: `/root/veil-child-node-bootstrap.json`
- Installed versions:
  - `go version go1.23.7 linux/amd64`
  - `avalanche version 1.9.6`
  - `gcc (Ubuntu 11.2.0-19ubuntu1) 11.2.0`
  - `tmux 3.2a`
