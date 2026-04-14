---
title: "Client Auditor"
description: "7-stage orchestrated audit for blockchain node codebases (Go, Rust, C/C++)"
date: 2026-04-13
draft: false
tags: ["Go", "Rust", "C++", "Blockchain", "Auditing"]
skill_type: "client-auditor"
tech: "Go / Rust / C++"
weight: 2
---

A structured 7-stage audit using an orchestrator + subagent architecture for security auditing of blockchain node implementations. Covers execution clients, consensus clients, app-chain SDKs, bridges, and any codebase with P2P networking or consensus logic.

## Pipeline

1. **Setup** — creates output directories, records audit parameters
2. **Recon** — maps codebase structure, entry points, trust boundaries, and applicable patterns
3. **Hunt** — parallel subagents analyze assigned subsystems against 20 vulnerability pattern families
4. **Cross-subsystem** — traces trust boundary mismatches at subsystem call sites
5. **Validation** — deduplicates findings, applies severity override rules
6. **Adversarial review [deep]** — Red Team / Blue Team / Judge protocol for HIGH+ findings
7. **Report** — consolidated report from disk state

## Knowledge Base

- **20 vulnerability pattern families** covering input validation, consensus correctness, resource exhaustion, memory safety, concurrency, serialization, and more
- **7 structured analysis lenses** for systematically examining code at trust boundaries
- **Heuristic strategies** for finding bugs that patterns alone won't catch

## Track Record

- $1K earned on [Immunefi](https://immunefi.com/profile/DARKNAVY/) (1 Medium finding)
- Independently discovered a vulnerability in [rippled](https://github.com/XRPLF/rippled) (XRP Ledger), officially acknowledged and patched

## Scope

- Execution clients (go-ethereum, Erigon, Reth, Nethermind, Besu)
- Consensus clients (Lighthouse, Prysm, Teku, Nimbus, Lodestar)
- App-chain SDKs (Cosmos SDK, Substrate, CometBFT)
- Bridge and relayer codebases

## Install

```bash
Install skill https://github.com/DarkNavySecurity/web3-skills/
```

## Usage

```bash
/client-auditor .           # Audit current repo
/client-auditor ./node      # Audit subdirectory
/client-auditor . deep      # Full deep audit
```
