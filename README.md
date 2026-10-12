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
{"instance_id":"home","seq":52311,"hash":"sha256:…","time":"2026-10-12T09:30:00.000000Z"}
```

Files are RFC 8785 canonical JSON with no trailing newline.

## Rules

- **Add only.** Each commit only adds files under `anchors/`.
- **Protected `main`.** Branch protection forbids force pushes and deleting the branch, and applies to admins too.
- **Verification reads the git history.** The verifier treats any commit that modifies or deletes an existing anchor file as tampering:

  ```bash
  git log --diff-filter=MD -- anchors/
  ```

  This command must print nothing.
- **Narrow write access.** The runtime writes with a fine-grained token limited to this repository, with *Contents: read and write* only and no administration permission, so it cannot lift the protection.
