# getsarde/action

GitHub Action to build [Sarde](https://github.com/getsarde/sarde) static sites for deployment. Installs Sarde, runs the build, and optionally uploads a GitHub Pages artifact.

## Usage

### GitHub Pages

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - uses: actions/configure-pages@v5

      - uses: getsarde/action@v1

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

## Inputs

| Input | Description | Default |
|---|---|---|
| `path` | Path to the Sarde project root (where `sarde.yaml` lives) | `.` |
| `version` | Sarde version to install (e.g. `v1.0.0`) | `latest` |
| `build-args` | Additional arguments passed to `sarde build` | `""` |
| `upload-pages-artifact` | Upload `dist/` as a GitHub Pages artifact | `true` |

## Examples

### With options

```yaml
- uses: getsarde/action@v1
  with:
    path: docs
    version: v1.2.0
    build-args: "--baseURL /my-repo --drafts"
```

### Cloudflare Pages

```yaml
steps:
  - uses: actions/checkout@v7
    with:
      fetch-depth: 0

  - uses: getsarde/action@v1
    with:
      upload-pages-artifact: "false"

  - uses: cloudflare/wrangler-action@v3
    with:
      command: pages deploy dist --project-name=my-site
      apiToken: ${{ secrets.CF_API_TOKEN }}
```

### Vercel

```yaml
steps:
  - uses: actions/checkout@v7
    with:
      fetch-depth: 0

  - uses: getsarde/action@v1
    with:
      upload-pages-artifact: "false"

  - uses: amondnet/vercel-action@v25
    with:
      vercel-token: ${{ secrets.VERCEL_TOKEN }}
      working-directory: dist
```

### Netlify

```yaml
steps:
  - uses: actions/checkout@v7
    with:
      fetch-depth: 0

  - uses: getsarde/action@v1
    with:
      upload-pages-artifact: "false"

  - uses: nwtgck/actions-netlify@v3
    with:
      publish-dir: dist
      production-deploy: true
    env:
      NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
      NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

## Notes

- Set `fetch-depth: 0` on checkout so `last_updated: git` in `sarde.yaml` can read git history for page dates.
- The action only builds. Deployment is handled by platform-specific actions.
- Set `upload-pages-artifact: "false"` when deploying to anything other than GitHub Pages.

## License

MIT
