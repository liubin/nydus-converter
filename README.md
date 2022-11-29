# nydus-converter

GitHub action for convertering OCI images to nydus.

**Notes**(TODOs):

- only support push images to ghcr.io
- no auth support.
- only tested on `ubuntu-latest`

## Usage

[This workflow](examples/on-push.yaml) will convert `docker.io/library/alpine:latest` to `ghcr.io/liubin/alpine:latest-nydus-v5`, where the target image is composed by:

- `ghcr.io`: GitHub container registry
- `liubin`: the organization of current repo(get from `${{ github.repository }}`)
- `alpine`: the repo name, not changed from the source image
- `latest-nydus-v5`: the target image name, where `latest` is the original tag, `nydus-v5` is for nydus image with v5 file format(v5 and v6 are supported).

**NOTE**:
> Suggest to upload target images to ghcr.io, it will be easy and safe to setup username/password.

```yaml
on: [push]

env:
  REGISTRY: ghcr.io

jobs:
  converter_job:
    runs-on: ubuntu-latest
    name: Test convert v5 image
    steps:
      - uses: actions/checkout@v3
      - name: convert image
        uses: ./
        with:
          source: "docker.io/library/alpine:latest"
          fs-version: "5"
          registry: ${{ env.REGISTRY }}
          repository: ${{ github.repository }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
```
