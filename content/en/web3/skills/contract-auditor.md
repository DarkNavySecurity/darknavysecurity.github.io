---
title: "Contract Auditor"
description: "DFS-based multi-agent Solidity audit with adversarial validation"
date: 2026-04-13
draft: false
tags: ["Solidity", "Smart Contract", "Auditing"]
skill_type: "contract-auditor"
tech: "Solidity"
weight: 1
---

A DFS-based AI security auditor for Solidity. The lead auditor reads code, builds a structured context map, extracts value-flow call paths, then delegates each path to a hunt agent for line-by-line depth-first analysis. Findings are merged, deduplicated, and validated.

## Pipeline

1. **Reconnaissance** — discover `.sol` files, resolve skill references, create temp dir
2. **Context & Analysis** — subagent builds context map + threat model + agent allocation plan
3. **Delegated Hunting** — parallel hunt agents do DFS on assigned call paths
4. **Merge & Dedup** — deduplicate findings, assess coverage against entry point census
5. **Adversarial [deep]** — falsifier agent challenges every finding with source verification
6. **Report** — severity-ranked findings + honest coverage summary

## Design Philosophy

- **Coverage, not constraint** — The primary job is structural: build a context map of every entry point, every state variable, every value flow. An agent that never reads a function cannot find a bug in it.
- **Domain knowledge as a reference, not a script** — The checklist is a curated set of Solidity vulnerability patterns that agents consult when they encounter matching code.
- **Validation as discipline** — Every finding passes a 3-gate false-positive check + 6D adversarial scoring.

## Track Record

$21K earned on [Immunefi](https://immunefi.com/profile/DARKNAVY/)

## Install

```bash
# Tell Claude Code:
Install skill https://github.com/DarkNavySecurity/web3-skills/
```

## Usage

```bash
# Scan the full repo
/contract-auditor

# Deep: adds adversarial falsifier after merge
/contract-auditor deep

# Review specific file(s)
/contract-auditor src/Vault.sol
```
