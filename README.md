# harness-anchors

External anchors for the ledger of [`faruk-akyol/harness`](https://github.com/faruk-akyol/harness) (private).

The harness ledger is a SHA-256 hash chain. A hash chain detects an edited record, but not a history rewritten from some record onward with every later hash recomputed. To catch that, the runtime periodically publishes its chain head here: `(instance_id, seq, hash, time)`. The runtime can add anchors but cannot change or remove them. The verifier compares the ledger against these anchors. Design: `docs/design/ledger.md` §6.3 and ADR 0005 in the harness repo.

**Only hashes are stored here. No ledger content.** A SHA-256 hash reveals nothing about the records it covers, which is why this repository can be public.

## Format

One file per anchor, never modified after it is added:

```text
anchors/<instance_id>/<seq zero-padded to 20 digits>.json
```

```json
{"hash":"sha256:…","instance_id":"home","seq":52311,"time":"2026-10-12T09:30:00.000000Z"}
```

Files are RFC 8785 canonical JSON with no trailing newline, so keys are sorted: `hash`, `instance_id`, `seq`, `time`. `time` is the anchored record's own envelope time, not the time of publication, so an anchor is a pure function of the record and a retried publish produces identical bytes.

The only other file allowed under `anchors/` is `anchors/.gitkeep`. Anything else under `anchors/` (an unknown file, a non-canonical anchor, a misplaced path) is a format error and fails verification.

## Rules

- **Add only.** Each commit only adds files under `anchors/`.
- **Protected `main`.** A repository ruleset blocks force pushes and branch deletion on `main` with no bypass (classic branch protection is also enabled). The harness checks this at startup via `GET /repos/faruk-akyol/harness-anchors/rules/branches/main`, which must list the `non_fast_forward` and `deletion` rules.
- **Verification reads the git history.** The verifier treats any commit that modifies or deletes an existing anchor file as tampering:

  ```bash
  git log --diff-filter=MDRCT -- anchors/
  git log --merges -- anchors/
  ```

  Both commands must print nothing. The harness verifier is stricter: it compares each commit's `anchors/` tree with its parent's, so the check works for commits of any size.
- **Narrow write access.** The runtime writes with a fine-grained token limited to this repository, with *Contents: read and write* only and no administration permission, so it cannot lift the protection.
- **Detect, not refuse.** A token with *Contents: write* can technically change an existing anchor by sending its blob `sha`. GitHub does not refuse that, but the history cannot be rewritten (no force push, no branch deletion), so the verifier detects any such change.
