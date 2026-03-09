# Auths

**Decentralized identity for developers. Cryptographic commit signing with Git-native storage.**

Auths replaces GPG, SSH key management, and centralized identity providers with a single, portable developer identity that lives in your Git repository. No servers required. No vendor lock-in. Your identity belongs to you.

---

## What Auths Does

Every commit you sign with Auths is backed by a cryptographic attestation chain rooted in a self-sovereign DID (Decentralized Identifier). Your identity, devices, and trust relationships are stored directly in Git refs — not in a cloud database, not on a third-party server, not in a keyserver you don't control.

**Sign commits.** One command replaces the entire GPG key ceremony. `auths init` creates your identity. `auths sign` signs your work. Done.

**Verify anywhere.** Verification is stateless and offline. No network calls, no certificate authorities, no transparency logs to query. Hand someone your identity bundle and they can verify your signatures on an air-gapped machine.

**Rotate keys without losing your identity.** Built on KERI (Key Event Receipt Infrastructure), Auths supports pre-rotation — commit to your next key before you need it. Rotate compromised keys without losing your commit history or breaking your identity chain.

**Pair devices cryptographically.** Link your laptop, workstation, and CI runners to a single identity with SAS-verified device pairing. Revoke a stolen device without regenerating everything.

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
