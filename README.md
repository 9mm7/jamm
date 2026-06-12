# Jàmm — encrypted voting for collective organizations

> *Jàmm* means "peace" in Wolof.

Jàmm is a **local-first, end-to-end encrypted voting platform** for collective organizations: tontines, diaspora associations, cooperatives, unions, coalitions. It lets communities run verifiable elections and collective decisions without trusting a central server with their ballots.

**This is the public showcase.** The full application is a Tauri desktop admin app plus an opt-in Rust web portal; the World ID proof-of-personhood feature for the *Ship a Feature* track at **ETHGlobal NYC 2026** is **in active development** (see below).

## Screenshots

> Designs from the Jàmm design system (forest green `#1f5a3d` · terracotta `#c25a2d`).

**Member web portal** — voting, dues status, open elections, announcements

![Jàmm member portal] <img width="1280" height="901" alt="member-home" src="https://github.com/user-attachments/assets/ba653edb-0735-4522-b904-b5f06c32fa39" />

**Public audit** — every action sealed on the canonical integrity chain, with an independent Python verifier and public export

![Jàmm public audit] <img width="1280" height="950" alt="web-audit-live" src="https://github.com/user-attachments/assets/8b73c88d-55f7-4bf4-918a-27938cfeb843" />

> More: a full coalition demo (open + revealed ballots with results, finances,
> events) is captured under [`screenshots/`](screenshots/) (i.e. `docs/screenshots/`).

## Why

Collective organizations in Senegal and across the diaspora run on trust — and trust breaks when votes are disputed. Existing tools either require trusting a SaaS vendor with sensitive membership data, or provide no integrity guarantees at all. Jàmm is built on a different premise: **the organization owns its data, cryptography guarantees the count.**

## How it works

<!-- NOTE: review depth of security disclosure before any production/public launch -->

- **Local-first storage** — each organization's data lives in an SQLCipher-encrypted database it controls. No cloud dependency for the core.
- **Argon2id-hardened key** — the database master key is derived with Argon2id (19 MiB, memory-hard), making offline brute-force of a stolen database file impractical, not just PBKDF2-fast.
- **Canonical hash chain** — every ballot and privileged action is appended to a tamper-evident canonical chain (v2, `BLAKE3(SHA-256(...))`) with public JSON export and an **independent Python verifier**: anyone can recompute the count without trusting the app.
- **Member web portal** — a Rust (Axum) HTTP server (opt-in, behind Caddy) serves a member SPA for remote voting, dues, announcements and public audit, with the same integrity guarantees. CSRF double-submit, per-IP rate limiting, TOTP 2FA.
- **Desktop admin** — full organization management: members, finances, votes, elections, events, territory-targeted communication, documents, audit trail, TOTP 2FA, server-side PDF reports. Privileged administration is desktop-only by design: `super_admin` is rejected on the web surface, so the master password never crosses the network.

## In development: one human, one vote (ETHGlobal NYC 2026)

The hardest problem in any voting system open to remote participation is **sybil resistance**. Without proof of unique personhood, one person can register several member accounts and stuff the ballot — and the vote is no longer trustworthy. The feature being built for the *Ship a Feature* track integrates **World ID** proof-of-personhood into the member voting flow as a *real constraint*, not a cosmetic add-on. The design:

- A member proves they are a unique human — without revealing identity documents or biometrics to the organization or to Jàmm.
- The action is scoped per election (`jamm-election-{id}`), so one verified human = one ballot **per election**, while re-verification across different elections stays unlinkable.
- **Proof validation runs in the Rust backend** (never client-side), against the World ID verification endpoint. The nullifier is enforced under a `UNIQUE(election_id, nullifier)` constraint, making duplicate registration cryptographically impossible.
- The verification is appended to Jàmm's **canonical integrity chain** as a `world_id_verified` event — so the personhood proof itself becomes a tamper-evident, independently auditable record: World ID personhood anchored inside an organization's own self-owned ledger.

Privacy is preserved end-to-end: World ID's zero-knowledge proofs mean no biometric or personal data ever touches Jàmm's database.

> Status: design locked, integration in progress. This section describes the
> target architecture, not yet a merged feature.

## Tech

| Layer | Stack |
|---|---|
| Desktop app | Tauri 2 · Rust · React · TypeScript |
| Storage | SQLCipher (encrypted SQLite) · Argon2id key · canonical hash-chain ledger |
| Web portal | Axum (Rust) · React SPA |
| Identity (in dev) | World ID (proof of personhood, ZK) |

**Quality:** v1.1 (functionally complete) · **270 Rust tests** (including 49 HTTP integration tests) + 12 Playwright e2e + 19 web unit tests, zero failures on every commit (fmt + `clippy -D warnings` enforced) · a completed **10-sprint security audit sweep** (gate logs A1–A9 + a consolidated final report for A10), following an earlier multi-pass review — **all confirmed findings to date fixed**, plus an Argon2id key-derivation hardening pass.

## Design

Jàmm ships with a complete locked design system — forest green `#1f5a3d`, terracotta `#c25a2d` — across desktop and web. See [`screenshots/`](screenshots/) (i.e. `docs/screenshots/`).

## AI tool usage

Built with AI-assisted development (Claude Code) under a test-driven, spec-driven workflow. AI assisted code generation, refactoring and test scaffolding across the codebase; all architecture, security design (integrity chain, key derivation, threat model), product decisions, the World ID integration design, and code review were directed and verified by the author.

## Status & roadmap

- ✅ Desktop admin application — v1.1, functionally complete
- ✅ Member web portal (React SPA + Rust HTTP server)
- ✅ Canonical integrity chain v2 with independent public verifier
- ✅ 10-sprint security audit sweep completed — all confirmed findings to date fixed (+ earlier multi-pass review) + Argon2id key hardening
- 🛠 World ID sybil-resistant voting — in development (ETHGlobal NYC, June 2026)
- ⏭ Packaging (.dmg/.msi/.AppImage), pilot deployments with diaspora organizations (NYC) and community organizations (Dakar)

Interested in piloting Jàmm with your organization? Reach out. -- notmo00@protonmail.com
