# VEIL Documentation

Technical documentation, specifications, runbooks, and architecture guides for the VEIL protocol.

## Structure

```
docs/
├── architecture/     System design and architecture overviews
├── specs/            Protocol specifications and technical designs
│   ├── VEIL_V1_NATIVE_PRIVACY_SPEC.md          Privacy layer specification
│   ├── VEIL_ZK_CONSENSUS_4_6S_TRIAL_PROFILE.md ZK consensus trial results
│   ├── VEIL_WHITEPAPER_ALIGNMENT_MATRIX.md     Whitepaper alignment tracking
│   ├── VEIL_FOUNDATION_FLYWHEEL_LAUNCH_PLAN.md Flywheel economics launch plan
│   ├── VEIL_EXECUTION_PACKAGE.md               Execution roadmap
│   ├── VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md  Companion EVM feature matrix
│   ├── VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md  Validator readiness
│   ├── VEIL_GEMINI_SIMULATION_MODEL_INPUT_2026-02-25.md     Simulation parameters
│   └── BOND_MARKETS_V2.md                      Bond market V2 design
│
├── runbooks/         Operational procedures
│   ├── VEIL_MASTER_RUNBOOK.md                  Master operations guide
│   ├── VEIL_PRODUCTION_LAUNCH_CHECKLIST.md     Production launch checklist
│   ├── VEIL_PRODUCTION_KEY_CEREMONY_RUNBOOK.md Key ceremony procedure
│   ├── VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md  Mempool privacy hardening
│   ├── VEIL_EVM_INTENT_RELAY_RUNBOOK.md        EVM intent relay operations
│   ├── VEIL_CHILD_NODE_BOOTSTRAP_RUNBOOK.md    Child node bootstrapping
│   ├── VEIL_MAINNET_CHAINLINK_BRIDGE_RUNBOOK.md   Chainlink bridge setup
│   ├── VEIL_STANDALONE_RPC_RUNBOOK.md          Standalone RPC setup
│   ├── VEIL_NATIVE_FLYWHEEL_EXECUTION_PLAYBOOK.md Flywheel execution
│   ├── EVMBENCH_AUDIT_RUNBOOK.md               EVM benchmarking & audit
│   └── CRITICAL_PHASE_CONTROL_TOWER.md         Phase control operations
│
├── companion-evm/    Companion EVM documentation
│   ├── README.md                               Companion EVM overview
│   ├── OPAQUE_EVM_INTENT_MIGRATION_CHECKLIST.md   Intent migration checklist
│   └── VEIL_EVM_RAILS_BEHIND_THE_SCENES.md    EVM rails internals
│
├── handoffs/         Development session handoffs and status updates
│   └── (session continuity documents, blocker burndowns, recovery notes)
│
└── devlog/           Development log
    └── LIVE_DEVLOG.md                          Running development journal
```

## Protocol Overview

**VEIL** is a privacy-native prediction market protocol built on a custom Avalanche L1.

### Key Technical Decisions

- **Custom VM**: VEILvm built on HyperSDK (Go), not Subnet-EVM. Purpose-built for privacy-preserving market operations.
- **Dual Chain**: HyperSDK primary chain + Companion EVM for Solidity contract interop
- **Chain-Owned Liquidity**: The L1 itself holds liquidity positions — not protocol-owned, *chain*-owned
- **MakerDAO-style Stability**: VAI stablecoin backed by adapted Maker modules (Vat/Jug/Spot/Dog)
- **Intent Architecture**: Users express intent; system routes optimally across chains
- **ANIMA Agents**: Autonomous agents with full lifecycle — provision compute, deploy validators, trade markets

### Chain Details

| Property | Value |
|----------|-------|
| Chain ID | `22207` |
| Token | VEIL |
| Stablecoin | VAI |
| VM | Custom HyperSDK (Go) |
| Consensus | Avalanche Snowman |
| Companion | EVM-compatible sidechain |

## Related Repos

| Repo | Layer | Description |
|------|-------|-------------|
| [`veilvm`](https://github.com/thesecretlab-dev/veilvm) | Chain | Custom HyperSDK VM |
| [`veil-contracts`](https://github.com/thesecretlab-dev/veil-contracts) | Contracts | 34 Solidity contracts |
| [`veil-frontend`](https://github.com/thesecretlab-dev/veil-frontend) | Application | veil.markets frontend |
| [`zeroid`](https://github.com/thesecretlab-dev/zeroid) | Identity | ZK-SNARK identity |
| [`anima-runtime`](https://github.com/thesecretlab-dev/anima-runtime) | Agents | Sovereign agent framework |
| [`veildb`](https://github.com/thesecretlab-dev/veildb) | Data | IPFS-backed data layer |

## Research

- **[LatentSync](https://thesecretlab.app/research/latentsync/)** — Agent-to-agent communication via continuous latent vectors

## License

MIT
