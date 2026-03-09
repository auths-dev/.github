# Auths

**Cryptographic identity and attestation infrastructure for developers, AI agents, and automated workflows.**

Auths is built on a single observation: the identity that signs a commit, the device that holds the key, and the binary that ships to production are all links in the same chain. Today those links are disconnected — GPG keys that can't rotate, CI secrets with no provenance, AI agents with API keys that never expire. Auths connects them.

One identity primitive — built on [did:keri](https://weboftrust.github.io/ietf-did-keri/draft-pfeairheller-did-keri.html) — handles all of it. The same cryptographic chain that lets you pair a new laptop to your developer identity also lets a CI runner attest that it built a specific binary, on a specific commit, authorized by a specific person. Open the `.auths.json` file attached to any Auths-signed release and you see the full chain: **identity to device to artifact signature**, end to end, verifiable offline.

No servers. No vendor lock-in. Identity and trust relationships stored directly in Git refs. Verification is stateless — hand someone your identity bundle and they can verify everything on an air-gapped machine.

---

## What Auths Does

**One identity, every context.** A developer at a terminal, a GitHub Actions runner, an AI agent executing tool calls, a mobile app signing artifacts — each gets a first-class cryptographic identity backed by a self-sovereign DID. Not a shared secret. Not an API key. A real identity with an auditable, append-only history.

**Full-chain provenance.** A signature proves who signed something. Auths proves more: who authorized them to sign, which device they used, when that authorization expires, whether it has been revoked, and — for artifacts — which commit produced the binary and which CI identity built it. Every release ships with a verifiable chain from human identity through build infrastructure to the artifact in your hands.

**Sign commits.** One command replaces the entire GPG key ceremony. `auths init` creates your identity. `auths sign` signs your work. Done.

**Sign artifacts.** The same identity that signs your commits signs your binaries. `auths artifact sign` produces a `.auths.json` attestation bundle that binds the artifact to your identity, the device that produced it, and the full delegation chain. Anyone can verify it without contacting a server.

**Verify anywhere.** Verification is stateless and offline. No certificate authorities, no transparency logs to query. The verifier runs as a native binary, C FFI library, WASM module, or Python package — same verification logic, every platform.

**Delegate with precision.** Issue scoped, time-bound credentials to CI runners, AI agents, and team members. A build agent gets `sign:artifact` for 24 hours. An AI coding assistant gets `sign:commit` scoped to a single repository. Revoke any credential instantly without disrupting others.

**Rotate keys without losing your identity.** Built on KERI (Key Event Receipt Infrastructure), Auths supports pre-rotation — commit to your next key before you need it. Rotate compromised keys without losing your commit history or breaking your identity chain.

**Pair devices cryptographically.** Link your laptop, workstation, and CI runners to a single identity with SAS-verified device pairing. The same mechanism that links a developer's second laptop also onboards a CI runner — because in Auths, a CI runner is just another device with scoped capabilities.

---

## Architecture

Auths is not a wrapper around existing tools. It is a ground-up cryptographic identity system built in Rust, designed for correctness, portability, and zero-dependency verification.

**24 crates** powering a layered architecture:

- **[auths-crypto](https://github.com/auths-dev/auths/tree/main/crates/auths-crypto)** — Ed25519 signing, DID encoding (did:key, did:keri), pluggable crypto providers (ring for native, WebCrypto for WASM)
- **[auths-verifier](https://github.com/auths-dev/auths/tree/main/crates/auths-verifier)** — Attestation chain verification compiled to native, C FFI, and WASM from a single crate
- **[auths-core](https://github.com/auths-dev/auths/tree/main/crates/auths-core)** — Keychain integration (macOS Keychain, Linux Secret Service, Windows Credential Manager), device pairing protocol, trust model
- **[auths-id](https://github.com/auths-dev/auths/tree/main/crates/auths-id)** — KERI Key Event Log management, Git-native identity storage
- **[auths-sdk](https://github.com/auths-dev/auths/tree/main/crates/auths-sdk)** — Application services layer with dependency injection, no I/O, no prompts — ready for embedding in any host
- **[auths-cli](https://github.com/auths-dev/auths/tree/main/crates/auths-cli)** — The developer-facing tool
- **[auths-mobile-ffi](https://github.com/auths-dev/auths/tree/main/crates/auths-mobile-ffi)** — UniFFI bindings for Swift and Kotlin
- **[auths-python](https://github.com/auths-dev/auths/tree/main/packages/auths-python)** — PyO3 SDK with full type stubs and Stripe-style resource API

**7 cloud crates** (proprietary) for managed services:

- Authentication server, identity registry, OIDC bridge, SCIM provisioning, and more

---

## Repositories

| Repository | Description |
|---|---|
| [auths](https://github.com/auths-dev/auths) | Core identity system — CLI, SDK, verifier, crypto, and 24 Rust crates |
| [auths-cloud](https://github.com/auths-dev/auths-cloud) | Managed services — auth server, registry, OIDC bridge, SCIM |
| [auths-verify-widget](https://github.com/auths-dev/auths-verify-widget) | Drop-in web component for commit verification (TypeScript, WASM) |
| [auths-verify-github-action](https://github.com/auths-dev/auths-verify-github-action) | GitHub Action for CI/CD commit signature verification |
| [auths-site](https://github.com/auths-dev/auths-site) | Documentation and project website |

---

## Why Not Sigstore / GPG / SSH Signing

| | Auths | Sigstore (Gitsign) | GPG | SSH Signing |
|---|---|---|---|---|
| Offline verification | Yes | No (requires Rekor) | Yes | Yes |
| Key rotation | Yes (KERI pre-rotation) | N/A (ephemeral certs) | No | No |
| Multi-device identity | Yes (cryptographic pairing) | No | No | No |
| Self-sovereign identity | Yes (did:keri) | No (OIDC-bound) | Partial (keyservers) | No (platform-bound) |
| Git-native storage | Yes (refs/auths/) | No | No | No |
| Zero infrastructure | Yes | No (Fulcio CA + Rekor log) | Partial (keyservers) | Partial (allowed_signers) |
| CI/CD integration | GitHub Action, any CI | GitHub only | Manual | GitHub only |

---

## Install

```bash
# From crates.io
cargo install auths-cli

# From pre-built binaries (Linux x86_64, Linux ARM64, macOS ARM64)
# See GitHub Releases for latest
```

```bash
# Quick start
auths init        # Create your identity
auths sign        # Sign commits
auths verify      # Verify signatures
auths doctor      # Diagnose issues
```

```python
# Python SDK
pip install auths-python

from auths import Auths

client = Auths()
identity = client.identities.create("my-identity")
client.devices.link(identity.did, capabilities=["sign:commit"])
```

```html
<!-- Verify widget (CDN, no build step) -->
<script type="module" src="https://unpkg.com/@auths-dev/verify"></script>
<auths-verify repo="owner/repo" commit="abc123"></auths-verify>
```

---

## Published Packages

- **crates.io** — All 24 public crates published and versioned
- **npm** — `@auths-dev/verify` verification web component
- **GitHub Actions** — `auths-dev/auths-verify-github-action@v1`
- **PyPI** — `auths-python` (coming soon)

---

## License

The core identity system is licensed under **Apache-2.0**. Cloud services are proprietary.
