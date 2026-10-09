# Working on ZeroDay Release Verifier

Verify that public delivery matches operator-approved release metadata and bytes.
This small repository owns a checkout-free receipt workflow, not the client
build, signing system, release authorization or production infrastructure.

Read `README.md` and `.github/workflows/verify.yml` together. The workflow is the
executable contract; `SECURITY.md` defines private reporting for bypasses and
non-public release metadata. Do not import the full ZeroDay build/release process
into routine verifier maintenance.

- Keep manual dispatch, empty token permissions, no checkout and no execution of
  repository scripts. Input values enter shell through environment variables;
  validate all five before network access. Preserve strict version, filename,
  lowercase digest and nonce shapes.
- Expected hashes come from the approved record, never from the endpoint being
  tested. Match version, filename, client digest, patch-note schema and public
  audience. Hash the exact `jq -cS '.patchNotes'` output, including its trailing
  newline; alternate JSON serialization can change the approved digest.
- Verify both metadata and downloaded client bytes. Keep bounded requests,
  fail-on-error behavior, isolated temporary files and exit cleanup. Emit
  `ZERODAY_EXTERNAL_RECEIPT` only after every check passes, with the supplied
  nonce and verified values. Wrong bytes, metadata or a failed fetch must fail.
- A receipt is a dated delivery observation, not approval provenance, signing,
  availability history or transport confidentiality. Preserve the documented
  HTTP transport limitation. Keep private release metadata and credentials out
  of public issues, logs and fixtures.

There is no committed build or local test harness. For workflow behavior changes,
extract and syntax-check the Bash body with `bash -n`, then exercise it against
isolated fixtures/test doubles: matching input; malformed inputs; wrong version,
filename, digest, schema or audience; altered patch notes/client bytes; network
failure; and no success receipt on failure. Preserve canonicalization and cleanup
in those checks. Do not label source inspection or mocked curl as a public probe.

`gh workflow run verify.yml --repo JadenRazo/zeroday-release-verifier` with the
five documented inputs is a live public probe that consumes an Actions run;
`gh run watch --repo JadenRazo/zeroday-release-verifier` follows its result.
Neither is a release operation or necessary for a prose edit. Use the probe only
within authorized verification scope and retain the exact run URL and inputs.
The current workflow has no push/PR trigger, so a guide PR alone runs no verifier.

Lead docs and PRs with the concrete contract change, why it matters, actual checks
and limits. Use descriptive evidence links and concise paragraphs. Follow existing
commit conventions, otherwise `type: concrete change` (preferably under 72
characters). Report local edits, review, push/PR and dispatched receipt separately;
never claim a release or default-branch adoption from a prepared guide.
