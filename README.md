# nydus-converter

GitHub action for convertering OCI images to nydus.

**Notes**(TODOs):

- only support push images to ghcr.io
- no auth support.
- only tested on `ubuntu-latest`

## Usage

### Example 1

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

### Example 2

[This example](.github/workflows/test.yaml) converts an image when triggering from an issue:

```yaml
on:
  issues:
    types:
      - opened

env:
  REGISTRY: ghcr.io

jobs:
  converter-job:
    runs-on: ubuntu-latest
    if:
      ${{ !github.event.issue.pull_request }}
      && ${{ github.repository }} == "liubin/nydus-converter"
      && ${{ github.actor }} == 'liubin'
      && startsWith(github.event.comment.body, '/convert ')
    name: Test convert v5 image by issue
    steps:
      - uses: actions/checkout@v3
      - name: Check issue body
        shell: bash
        run: |
          # bless there are no quotas or double quotas
          body='${{ github.event.issue.body }}'
          # ignore this very simple regex
          regex='\/convert (docker.io\/(.*)?\/(.*)?:(.*)?)'
          if [[ $body =~ $regex ]]
          then
              image=$(echo ${BASH_REMATCH[1]} | tr -d '\n')
              echo "got image: $image"
              echo "SOURCE=$image" >> $GITHUB_ENV
          fi
      - name: convert image
        id: nydus-converter
        uses: ./
        with:
          source: ${{ env.SOURCE }}
          fs-version: "5"
          registry: ${{ env.REGISTRY }}
          repository: ${{ github.repository }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Message success
        if: ${{ success() }}
        uses: actions/github-script@v4
        with:
          script: |
            github.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: "Conversion succeeded! ✅\nThe nydus image is ${{ steps.nydus-converter.outputs.nydus-image }}, you can use `docker pull` to use it:\n```bash\n$ docker pull ${{ steps.nydus-converter.outputs.nydus-image }}\n```\n",
            });
```
