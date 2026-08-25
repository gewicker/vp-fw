# fleet/ — the files that go in the OTHER two repos

Templates, pre-validated against the real parser (`t_manifest`), so nothing here can be wrong at
the bench. **Nothing in this directory is used by the firmware build** — it is copied into the two
Service Link repos by hand.

## The two repos, and why they are two

| | `vp-fw` (firmware + manifest) | `vp-logs` (uploads) |
|---|---|---|
| Visibility | **public** | **private** |
| Credential on device | **none** | a fine-grained PAT, `contents:read+write`, **that repo only** |
| Owned by | the project | a **dedicated bot account** |
| If the credential leaks | *there isn't one* | read + rewrite every log from every unit |

**The bot account is the point, and it is not optional.** It must **not be a collaborator on the
firmware repo at all** — then no token it can ever mint, of any type, can touch firmware. That
makes the split *structural* rather than dependent on somebody scoping a token correctly at 11pm.
Disable Actions on the log repo; never grant `workflows`. Set the PAT expiry to the 366-day
maximum and calendar it against a family visit — rotation is necessarily hands-on (§4 Tier 1.6).

**This is why an agent could not finish the setup:** creating a GitHub *account* needs an email
and verification. Creating both repos under `gewicker` instead would look finished while the
security property was absent, which is worse than not starting.

## Layout of `vp-fw` (public)
```
manifest/field.json          <- her channel
manifest/bench.json          <- bench units
device/<SN>.json             <- OPTIONAL per-unit overlay; 404 is the NORMAL case
fw/volcano_puck-<version>.bin
fw/volcano_puck-<version>.elf   <- see §8.1: without it, her panics are undecodable
```
Channels are **`field` / `bench`, never `stable`** — "stable" implies a promise about quality that
a fleet of two cannot make.

## Layout of `vp-logs` (private)
```
logs/<SN>/<zero-padded boot#>/<up_ms>-<seq>.log
fleet/units.tsv              <- the registry; it names a person, so it lives HERE, never public
```
