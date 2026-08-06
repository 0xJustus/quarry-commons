# quarry-commons CI

The write path to the commons is a **gate in CI**, not a deployed service with a
shared secret. Three workflows, all built from a pinned quarry core binary.

| Workflow | Trigger | What it does |
|---|---|---|
| `commons-verify.yml` | every PR + push to `main` | The **anti-poisoning gate**. Audits the whole tree and blocks merge unless it is consistent: content-addressed + signature-valid abstracts, no specimen/reproducer, Public tier only, an index bound to real artifacts and correctly sharded, and a Bloom digest byte-identical to the canonical digest of the indexed keys. |
| `hydrate.yml` | weekly + manual | Regenerates the tree from the ARVO catalog (metadata-only, runs no target code), verifies it inline, and commits only if it changed. |
| `autovet.yml` | dispatch only | Re-vets an allowlist of reproducer-bearing entries on the per-PoV **Fly Machine** substrate (each air-gapped on its own ephemeral Machine, never on the runner) and commits the admit/reject verdict ledger. |

The gate is the important one: **`commons.verify` reports a bad tree as
`{"ok": false, …}` with a zero exit code**, so the workflow parses `.ok` and fails
the job explicitly. Trusting the exit code alone would merge a poisoned tree.

## Setup

1. **Pin the core.** Each workflow's `QUARRY_REF` is a quarry core commit sha. It
   must be a ref that exists on the core remote, and the committed tree must pass
   *that* binary's gate. Bump it deliberately — the gate's integrity is the
   binary's. `setup-quarry` clones the core into `RUNNER_TEMP` (outside the
   workspace, so it never lands in a commit) and builds it `CGO_ENABLED=0`.

2. **Secrets.**
   - `QUARRY_REPO_TOKEN` — a read-only fine-grained PAT (or deploy key) for the
     private quarry core repo, so CI can build it. Omit once the core is public.
   - `FLY_API_TOKEN` — required only by `autovet.yml`, to drive the Fly substrate.

3. **Autovet prerequisites.** The `quarry-vetd` Fly app, per-target images pushed
   to `registry.fly.io`, and an allowlist file (default `autovet-allow.yaml`, see
   `autovet-allow.example.yaml`). Untrusted reproducers run on the Fly Machines,
   not on the GitHub runner.

## Notes

- **The default token does not trigger other workflows.** A push made by
  `GITHUB_TOKEN` will not fire `commons-verify.yml`, which is why `hydrate` and
  `autovet` verify inline before they commit. Human PRs trigger the gate normally.
- **`README.md` and `commons.json` at the repo root are machine-generated** by the
  catalog/hydrate path. Do not hand-edit them.
- **Consider SHA-pinning the `actions/*` steps** (`actions/checkout`,
  `actions/setup-go`) in addition to `QUARRY_REF`, for a fully pinned supply chain.
