# MiLaboratories Build of AppWrapper

This fork of [project-codeflare/appwrapper](https://github.com/project-codeflare/appwrapper)
builds the AppWrapper controller that Platforma deployments run. Branch `milab` holds our
release line: an upstream release plus the changes below.

## Changes on Top of Upstream

| Change | Why | Upstream |
|---|---|---|
| Confirm a missing component with an uncached read before failing | The controller can read a Job it has just created from a stale cache and fail the workload at once (`MissingComponent: Only found 0 deployed components`) | PR project-codeflare/appwrapper#417 (open) |
| `appwrapper.maxConcurrentReconciles` in the operator config | One reconcile worker can fall behind during a burst of submissions and create Jobs late; the delay counts against the admission grace period | not proposed yet |

## Configuration

The operator ConfigMap `appwrapper-operator-config` (key `config.yaml`) takes, in addition to the
upstream settings:

```yaml
appwrapper:
  maxConcurrentReconciles: 8   # unset or 0 = 1 (upstream behaviour)
```

Durations in `faultTolerance` are integers in nanoseconds (Go `time.Duration` in JSON), for example
`admissionGracePeriod: 300000000000` for 5m. The controller refuses to start when
`admissionGracePeriod` is longer than `warmupGracePeriod` (default 5m).

## Release

1. Merge into `milab`. CI (`.github/workflows/milab.yaml`) runs the unit tests and builds the image.
2. Tag the merge commit `vX.Y.Z-milab.N` (X.Y.Z = the upstream release it is based on) and push the tag.
3. CI pushes `quay.io/milaboratories/pl-containers:appwrapper-vX.Y.Z-milab.N` (linux/amd64,
   linux/arm64; also served as `containers.pl-open.science/milaboratories/pl-containers:...`) and
   creates a GitHub release with `install.yaml`.

To move to a new upstream release: branch from the upstream tag, cherry-pick the changes listed
above, open a PR into `milab`.
