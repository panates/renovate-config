# renovate-config

The shared [Renovate](https://docs.renovatebot.com) policy for panates repositories.

## Usage

Each repository gets one file, and it is a pointer rather than a copy:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>panates/renovate-config"]
}
```

`github>panates/renovate-config` resolves `default.json` from this repository's **default branch**.
Changing the policy is one edit here; no repository has to be touched again.

**That is the whole reason this repository exists.** Dependabot's per-repository file *is* the
policy, so the same setup across ~38 repositories means 38 copies of it, and changing the schedule
later means editing 38 files.

## What the policy does

| | |
| --- | --- |
| `schedule` | one batch, Monday before 6am (Europe/Istanbul) - not a trickle through the week |
| dev dependencies, minor + patch | grouped into a single pull request |
| major updates | held behind the Dependency Dashboard, never opened unprompted |
| github-actions | grouped into a single pull request |
| `lockFileMaintenance` | **weekly regeneration of the lockfile itself** - see below |
| `prConcurrentLimit` | 5 |

## `lockFileMaintenance` is the point

It regenerates `package-lock.json` on a schedule **even when no range in `package.json` changed**,
so transitive dependencies do not silently freeze at whatever they resolved to the day the lockfile
was written.

This is the controlled form of `rm -f package-lock.json && npm install`, which panates CI used to do
in the setup action. That did refresh dependencies - and it cost reproducibility to get there: CI
resolved fresh on every run while the committed lockfile froze forever, so CI, every developer, and
the repository's own record all tested different trees. The vulnerability counts GitHub reports are
measured against that committed file, which the deletion never touched.

`lockFileMaintenance` gets the same refresh as a **reviewable pull request that CI proves before it
merges**, once a week, with a diff someone can read.

## Related

- [`panates/github-actions`](https://github.com/panates/github-actions) - the reusable release and
  QC workflows. From `@v3` the release pipeline also commits a regenerated lockfile after bumping
  versions, because `rman` writes `package.json` and does not manage lockfiles.
- [`panates/gh-setup-node`](https://github.com/panates/gh-setup-node) - from `@v2` it installs with
  `npm ci` and no longer deletes the lockfile.

## Validating a change

```bash
npx --package renovate renovate-config-validator --strict
```

Run it from a directory where the file is named `renovate.json`; passing `default.json` by name
makes the validator check it against the *global* config schema instead, which is looser and will
accept keys a preset cannot use.

## License

MIT
