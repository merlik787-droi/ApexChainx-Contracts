# ApexChainx System — Project Context

> **Purpose:** This document describes the high-level system architecture, repository landscape,
> and future contract roadmap for the ApexChainx platform.

## Table of Contents

- [Repository Architecture](#repository-architecture)
- [System Flow](#system-flow)
- [Architectural Rules](#architectural-rules)
- [SC-100: Future Contract Roadmap](#sc-100-future-contract-roadmap)

---

## Repository Architecture

The ApexChainx platform is composed of three repositories:

| Repository | Role | Technology |
|------------|------|------------|
| `apexchainx-fe` | Frontend application | React / TypeScript |
| `apexchainx-be` | Backend API and integration layer | Python / FastAPI |
| `apexchainx-contracts` | Soroban smart contracts (this repo) | Rust / Soroban SDK |

## System Flow

```
 User
  |
  v
┌─────────┐     ┌─────────┐     ┌──────────────┐
│   FE    │ ──→ │   BE    │ ──→ │  Contracts   │
│ (React) │ ←── │ (API)   │ ←── │  (Soroban)   │
└─────────┘     └─────────┘     └──────────────┘
```

## Architectural Rules

1. **Frontend never calls contracts directly** — all contract interactions go through the backend
2. **Backend is the exclusive bridge** — translates contract data to frontend-friendly responses
3. **Contracts are execution-layer only** — pure deterministic computation, no external dependencies

---

## SC-100: Future Contract Roadmap

This section documents the planned evolution of `apexchainx-contracts` based on
current backend integration needs. It distinguishes what exists today from what
is planned, so contributors do not assume missing crates are already present.

### Current State

Only one contract crate exists in this repository:

| Crate | Status | Description |
|---|---|---|
| `apexchainx_calculator` | **Production-ready** | Calculates SLA penalties and rewards; stores config; emits versioned events |

### Planned Additions

The following crates are planned but **not yet implemented**. Do not import or
reference them until they appear in the repository.

| Crate | Status | Depends on | Description |
|---|---|---|---|
| `payment_escrow` | Planned | `apexchainx_calculator` | Locks and conditionally releases Stellar token payments based on SLA results |
| `settlement` | Planned | `payment_escrow` | Splits shared outage costs between multiple parties |
| `governance` | Planned | — | On-chain admin config changes with time-locked execution |

### Integration Expectations

- The backend (`apexchainx-be`) currently integrates only with `apexchainx_calculator`.
- New crates will be introduced incrementally; each must expose a
  `get_result_schema()` equivalent so the backend can version-pin safely.
- Frontend never calls contracts directly — all invocations go through the backend.

### Contribution Guidelines for New Crates

1. Open a tracking issue before creating the crate directory.
2. Follow the `apexchainx_calculator` layout: `src/lib.rs`, `src/tests.rs`, `Cargo.toml`.
3. Add the new crate to any CI matrix in `.github/workflows/`.
4. Export a result schema function so the backend can detect breaking changes.
