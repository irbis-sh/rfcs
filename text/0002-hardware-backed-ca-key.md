---
rfc: 0002
title: Hardware-backed root CA key storage
project: zen
type: feature
authors:
  - anfragment
status: draft
discussion: https://github.com/irbis-sh/rfcs/pull/0000
start-date: 2026-08-24
---

# Hardware-backed root CA key storage

## Summary

This RFC moves Zen's root CA private key from a plaintext file on disk into hardware-backed key storage - the Secure Enclave on macOS, the TPM (via CNG) on Windows - so the key can never be extracted from the machine. Because hardware signing is slow and serialised, the root does not sign leaf certificates directly - it signs a short-lived intermediate CA held in memory, and the intermediate signs the leaves.

Linux is out of scope for this RFC because TPM access there is restricted to a privileged user group. A follow-up RFC will address it.

## Motivation

Zen filters HTTPS traffic by installing a locally generated root CA into the system trust store and generating a certificate for each intercepted host. The root's private key currently lives at `rootCA-key.pem` in Zen's data directory. It's an unencrypted PKCS#8 PEM file that any process running as the user can read, paired with a certificate valid for 32 years.

In practice, an infostealer running under the user's account could copy a CA key that this machine will trust for decades. With any later chance to intercept traffic - a compromised network, a re-infection - it could indefinitely impersonate every HTTPS host the user visits with no warnings. Uninstalling Zen does not always revoke the CA, so the user would have to know to remove it from the trust store themselves.

[Zen's security architecture document](https://github.com/irbis-sh/zen-desktop/blob/master/docs/internal/security-architecture.md) acknowledges this as an open problem ("We're currently exploring ways to encrypt the private key using system APIs"). This RFC addresses that alongside a prototype, [key-protecc](https://github.com/anfragment/key-protecc).

## Impact

**Existing installations**: hardware-backed storage will probably be opt-in, at least initially - as a switch in Settings between a hardware key and today's on-disk key. An existing CA cannot be moved into hardware since a hardware-backed key has to be created inside it. Enabling the feature therefore means generating a fresh CA and seeing the trust-installation prompt once more. Migration is an explicit user choice. Whether hardware storage eventually becomes the default is an open question.

**Platform differences**:

- **macOS**: requires a Secure Enclave, present on all Apple silicon Macs and Intel Macs with a T2 chip. Persisting an Enclave key also requires restricted entitlements (`application-identifier`, `keychain-access-groups`), which macOS only honours in a signed app bundle with a provisioning profile - a new requirement for the release pipeline.
- **Windows**: requires a TPM 2.0, guaranteed on Windows 11 and common but not universal on Windows 10 hardware.
- **Linux**: unchanged by this RFC. TPM device access is limited to the `tss` group by default, and Zen runs as an ordinary user, so the Linux design needs its own approach to permissions. To be addressed in a follow-up RFC.
- **Machines without an Enclave or TPM**: the on-disk key stays available as an option. Selecting hardware keys on a machine without working hardware fails with a visible error.

**Performance**: largely unchanged. Leaf signing stays in software. The hardware is asked for one signature per intermediate rotation.

## Design

### Hardware root

The root CA key is generated inside, and never leaves, the platform's key hardware:

- **macOS**: an ECDSA P-256 key in the Secure Enclave, created with `SecKeyCreateRandomKey` (`kSecAttrTokenIDSecureEnclave`, data protection keychain), signing with `SecKeyCreateSignature`. Its access control is built with `SecAccessControlCreateWithFlags` carrying `kSecAccessControlPrivateKeyUsage` alone - no user-presence flag so the CA can sign unattended.
- **Windows**: an ECDSA P-256 key created with `NCryptCreatePersistedKey` and `NCryptFinalizeKey` under the CNG "Microsoft Platform Crypto Provider" - the TPM-backed provider - signing with `NCryptSignHash`, whose raw `r || s` output is converted to the DER form X.509 expects.

The application holds a key handle and can request signatures. Nothing - including Zen - can read the private key. The root certificate is public and is still written to disk for trust-store installation as today.

The root's certificate template changes in two ways: the key type becomes the one described above, and `MaxPathLen` becomes 1 (today's template sets `MaxPathLenZero`, which forbids any intermediate).

### Short-lived in-memory intermediate

Hardware signing is slow and, more importantly, serial. In the prototype's benchmarks the Secure Enclave signs at ~4.5ms per operation across `-cpu=1,2,4,8,12` on an M2 Max. A browser opening N connections to uncached hosts would pay the single-operation cost × N, queued behind everything else using the chip. Cold-cache bursts - the first load of the day, a proxy restart - would noticeably stall loading.

So the hardware root never signs leaves itself - one extra step sits in between:

1. At proxy start, Zen generates a software ECDSA P-256 key in memory and has the hardware root sign a short-lived intermediate CA certificate for it. The intermediate's key never touches disk.
2. Leaf certificates are generated exactly as today, but signed by the intermediate. The served chain becomes leaf + intermediate.
3. When the intermediate expires, it is regenerated - one hardware signature per rotation rather than one per host.

This accepts a limited amount of risk in exchange for usable performance: an attacker who reads Zen's memory gets a key that stops working at the intermediate's `notAfter`. The intermediate's TTL is deliberately left open (see [Unresolved questions](#unresolved-questions)). Note that it interacts with the 24-hour leaf TTL: a cached leaf must not outlive the intermediate it chains under.

### What changes in code

`certstore` stops parsing a PEM into key material and instead returns a `crypto.Signer` backed by the hardware handle - the interface the prototype's three backends already expose. `x509.CreateCertificate` accepts any `crypto.Signer`, so the signing call sites stay the same. `newCA()` generates in hardware rather than calling `rsa.GenerateKey`, and `loadCA()` opens a handle rather than reading a file. The leaf path in `certgen` gains the intermediate layer described above.

The macOS release pipeline additionally embeds a provisioning profile with the entitlements above. Development builds keep a software-key path behind a build tag. The Enclave path cannot run from a bare `go build` or `go test` binary at all - macOS kills unbundled binaries that claim restricted entitlements.

### The prototype

See [key-protecc](https://github.com/anfragment/key-protecc), which implements all three backends - the Secure Enclave through the Security framework (cgo), CNG/NCrypt on Windows, and (ahead of the follow-up RFC) a TPM 2.0 through `go-tpm` on Linux. To run the full flow locally - create a key, sign a root, issue a leaf, verify the chain - use these [Task](https://taskfile.dev) commands:

```sh
task certtest          # software key - runs anywhere, no hardware or entitlements
task certtest-enclave  # the same flow against the Secure Enclave
```

The Enclave variant has to build a signed `.app` bundle with the provisioning profile embedded (`macos/bundle.sh`, configured through `macos/signing.env`), because of the entitlement rule above. This needs a real developer certificate issued by Apple.

The benchmarks can be run with `task bench` (software baseline) and `task bench-enclave`.

## Drawbacks

- **Release and testing complexity on macOS**. The provisioning profile requirement means the Enclave path only runs in a signed, bundled app. CI and local development need the software fallback, so the production path gets less day-to-day testing.
- **A one-time migration prompt for every user who opts in**, with the confusion and support burden that brings.
- **The intermediate is still a software key**. Exposure is bounded by its TTL, not eliminated.
- **Platform divergence**. We take on two hardware backends behind different platform APIs, and Linux users keep the weak storage until the follow-up RFC.

## Rationale and alternatives

### Why an intermediate, rather than signing every leaf in hardware?

The serialisation numbers above: at ~4.5ms per signature, roughly 220 certificates per second machine-wide. Enough on average but far too slow in bursts. The intermediate removes hardware operations from the per-host hot path entirely, in exchange for one short-lived in-memory key.

### Why hardware keys, rather than encrypting the file at rest?

Encryption at rest is the approach the security architecture document originally suggested (e.g., Electron's `safeStorage`), but it fails against the actual threat. Any process running as the same user can call DPAPI on Windows with no prompt; on macOS the login keychain adds only a per-app consent prompt that could be waved with a careless click. And the key must sit fully decrypted in Zen's memory anyway. Encryption at rest protects against offline disk theft, which is not how CA keys get stolen. Hardware keys, by contrast, are non-extractable even against an attacker with arbitrary code execution as the user.

### The impact of not doing this

The status quo: the root key file remains a target for attackers.

## Prior art

Local TLS interception tools overwhelmingly keep their CA keys on disk: mkcert does, as do mitmproxy and Burp Suite - understandably, since they are developer tools. Consumer software has a worse record (see the Superfish and eDellRoot incidents). We found no mainstream interception, ad-blocking, or privacy tool that keeps its CA in a hardware key store. This proposal is instead adapted from server-side practices (HSM-held roots), rather than from a peer product.

## Security & privacy considerations

This RFC exists to change the security model, so most of the analysis is above. In summary:

- **What improves**: today, reading one file as the user gives an attacker permanent HTTPS traffic interception capability. After this change, the worst an attacker with the same privileges can do is request signatures while staying on the machine (unavoidable for any unattended signer) or steal an intermediate that stops working at the end of its TTL. What this design removes is the ability to carry a lasting capability off the machine.
- **New attack surface**: the keychain/TPM APIs and the entitlement setup. Both are narrower than the file they replace, given the design is correct and implemented properly in code.
- **Failure modes**: when hardware keys are selected, hardware absence or errors must fail loudly - never a silent fallback to the on-disk key.
- No change to what data Zen collects or processes.

## Unresolved questions

To resolve through this RFC's discussion:

- **Design soundness**: whether the hardware root with a short-lived in-memory intermediate is the right architecture for the problem.
- **Prototype correctness**: whether `key-protecc`'s platform-specific backends implement the idea correctly - the right APIs, key attributes, and parameters.
- **The intermediate's TTL**. A shorter TTL limits exposure more tightly. A longer one means fewer rotations and a simpler interaction with the 24-hour leaf TTL and the leaf cache: either a rotation invalidates cached leaves, or the TTL must exceed leaf validity.
- **The default**: whether, and when, hardware keys become the default rather than an opt-in.
- **TPM prevalence**: how common a working TPM 2.0 actually is on Windows 10 hardware.

During implementation: keychain attribute details, how errors are grouped and reported, and the shape of the software-fallback build tag.

Out of scope here: the Linux design, which a follow-up RFC will cover.

## Future possibilities

- The Linux follow-up RFC: the same architecture over a TPM 2.0, once the `tss`-group permissions question is worked out.
- Periodic root rotation becomes practical once CA regeneration is a routine, well-tested path rather than a one-time event.
- The same hardware-key code could later protect other long-lived secrets Zen acquires.
