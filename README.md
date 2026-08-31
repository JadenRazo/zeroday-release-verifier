# ZeroDay release verifier

A checkout-free GitHub Actions receipt that verifies an exact public ZeroDay release against operator-supplied metadata and SHA-256 digests.

This repository is deliberately small. Its single manual workflow provides an independently visible answer to one question: **does the public delivery endpoint currently serve the exact release bytes and public patch-note object that were approved?**

## Verification contract

The workflow accepts five required values:

| Input | Required shape | Purpose |
| --- | --- | --- |
| `version` | `X.Y.Z-rN` | Exact public release version |
| `client_filename` | `zeroday-*.jar` | Expected public client filename |
| `client_sha256` | 64 lowercase hex characters | Approved client artifact digest |
| `public_patch_notes_sha256` | 64 lowercase hex characters | Digest of canonical public patch-note JSON |
| `probe_nonce` | 32 lowercase hex characters | Unique identifier for the public receipt |

It then:

1. validates every input before making a request;
2. fetches the public version document with bounded timeouts and retries;
3. requires the exact version, filename, client digest, patch-note schema, and public audience;
4. canonicalizes the patch-note object with `jq -cS` and verifies its SHA-256;
5. downloads the public client and verifies the artifact SHA-256; and
6. emits one `ZERODAY_EXTERNAL_RECEIPT` line containing the verified values and nonce.

Any mismatch or network failure makes the run fail.

## Trust boundary

- The job declares `permissions: {}` and receives no repository token permissions.
- It does not check out this repository or execute repository scripts.
- Expected hashes are explicit workflow inputs; the public endpoint is not trusted to declare its own integrity.
- Temporary response files are isolated with `mktemp` and removed on exit.
- The default branch is protected against direct pushes, force-pushes, and deletion.

This is a delivery verifier, not a build or signing system. It does not create the release, establish who approved the expected hashes, prove artifact provenance, or provide an availability history. The endpoint currently uses plain HTTP; the pre-approved SHA-256 values protect artifact and metadata integrity, while transport availability and confidentiality remain outside this verifier's scope.

## Run it

The operator supplies values from the approved release record:

```bash
gh workflow run verify.yml \
  --repo JadenRazo/zeroday-release-verifier \
  -f version=1.2.3-r1 \
  -f client_filename=zeroday-client.jar \
  -f client_sha256=REPLACE_WITH_64_LOWERCASE_HEX \
  -f public_patch_notes_sha256=REPLACE_WITH_64_LOWERCASE_HEX \
  -f probe_nonce=REPLACE_WITH_32_LOWERCASE_HEX
```

Follow the run and preserve its URL as the external receipt:

```bash
gh run watch --repo JadenRazo/zeroday-release-verifier
```

## Repository contents

| Path | Purpose |
| --- | --- |
| `.github/workflows/verify.yml` | Pinned public-delivery verification logic |
| `SECURITY.md` | Private vulnerability reporting and scope |

## Security reporting

Do not publish a working verification bypass or non-public release metadata in an issue. Use the private route described in [SECURITY.md](SECURITY.md).
