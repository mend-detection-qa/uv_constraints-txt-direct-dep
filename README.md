# uv_constraints-txt-direct-dep

**What this probe proves:** `constraints.txt` enforcement is not just for transitive deps — Mend also applies pins to **direct** deps.

## Real-world scenario

A team declares `requests` as a direct dep in `pyproject.toml` without a version specifier. They want their security baseline to pin it to a specific version (e.g. `2.31.0` because 2.32.x had a yanked release). Dropping that pin into `constraints.txt` should resolve `requests` to `2.31.0` at scan time.

## Files

- `pyproject.toml` — direct dep `requests` (no version specifier — intentional)
- `uv.lock` — uv's unconstrained resolution (latest requests)
- `constraints.txt` — `requests==2.31.0`
- `whitesource.config` — `applyConstraints=true`

## Expected scan behavior (post SCA-5154)

Mend's dep tree reports:
- Direct: `requests` at **`2.31.0`** (NOT the latest version uv otherwise resolves)
- Transitives consistent with `requests` 2.31.0's release-time deps.

## Pairing with other probes

| Probe | constraint target | proves |
|---|---|---|
| `uv_constraints-txt-overlay` | transitive (urllib3) | basic auto-discovery + transitive pinning |
| `uv_constraints-txt-direct-dep` (this one) | **direct** (requests) | direct deps are also constrained |
| `uv_constraints-txt-range-exclude` | transitive range | range operator (not just `==`) works |
| `uv_constraints-txt-multi-pin` | two transitives | multiple entries in one file |
| `uv_constraints-txt-flag-off` | (none — flag off) | feature flag gates behavior |