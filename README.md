# lens-image

Container image for [BerriAI Lens](https://github.com/BerriAI/lens) (LiteLLM's
tracing and evaluation service), built from a pinned upstream commit until
BerriAI publishes its own image (`ghcr.io/berriai/lens`). Lens's Helm chart
says to do this: "Until a Lens release is published, build from the
repository root and push the image to a registry".

```
ghcr.io/2stacks/lens:<VERSION>          e.g. 0.1.0-dev.0 (upstream's VERSION file)
ghcr.io/2stacks/lens:<VERSION>-<sha7>   e.g. 0.1.0-dev.0-e320cab (records the commit)
```

Pin by digest (`ghcr.io/2stacks/lens:0.1.0-dev.0@sha256:…`). A new upstream
commit with the same `VERSION` rebuilds the `<VERSION>` tag with a new digest.

This repo contains no Lens source. `upstream.env` names the commit;
`.github/workflows/build.yml` builds
`https://github.com/BerriAI/lens.git#<commit>` with upstream's
`deploy/runtime/Dockerfile` (default target `runtime`, build arg
`LENS_VERSION` = upstream's `VERSION`), unchanged.

## Build

Pull requests build and smoke-test only (`litellm-lens --version` must report
the `VERSION`, `node --version`). Pushes to `main` also push both tags and
sign the digest with cosign (keyless). amd64 only. The Rust build uses the
default baseline x86-64 target, so the binaries run on CPUs without AVX2.

## Updating

Renovate watches BerriAI/lens `main` and lists commit bumps on the Dependency
Dashboard; they wait for a tick. Lens is pre-release and pairs with LiteLLM
versions: read the compare link (migrations, ClickHouse schema, API changes)
before merging.

When BerriAI publishes `ghcr.io/berriai/lens`, switch to it and archive this
repo.

## Verify the signature

Needs cosign v3 or newer:

```sh
cosign verify ghcr.io/2stacks/lens@sha256:<digest> \
  --certificate-identity 'https://github.com/2stacks/lens-image/.github/workflows/build.yml@refs/heads/main' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com
```

## License

The build files in this repo are MIT (LICENSE). Lens itself is MIT-licensed
by Berri AI (see the upstream repository); the image contains Lens as built
from upstream.
