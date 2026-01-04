# Homebrew Tap

Homebrew formulae for [grokify](https://github.com/grokify) CLI tools.

## Installation

```bash
brew tap grokify/tap
brew install <formula>
```

## Available Formulae

| Formula | Description |
|---------|-------------|
| `structured-changelog` | CLI for canonical, deterministic changelogs using JSON IR |

## Usage Examples

```bash
# Install structured-changelog
brew tap grokify/tap
brew install structured-changelog

# Use the CLI (both commands work)
sclog version
structured-changelog version

# Update
brew update
brew upgrade structured-changelog
```

## Adding a New Tool to This Tap

To distribute another CLI tool via this tap using GoReleaser:

### 1. Create a Fine-Grained Personal Access Token

- Go to: GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
- **Name:** `HOMEBREW_TAP_GITHUB_TOKEN`
- **Repository access:** Select `grokify/homebrew-tap`
- **Permissions:** Contents → Read and write

### 2. Add Secret to Your Tool's Repository

- Go to: `github.com/grokify/<your-tool>` → Settings → Secrets and variables → Actions
- Add new repository secret:
  - **Name:** `HOMEBREW_TAP_GITHUB_TOKEN`
  - **Value:** (paste the token)

### 3. Add GoReleaser Configuration

Create `.goreleaser.yaml` in your tool's repository:

```yaml
version: 2

project_name: your-tool

builds:
  - binary: your-tool
    main: ./cmd/your-tool
    env:
      - CGO_ENABLED=0
    goos:
      - linux
      - darwin
      - windows
    goarch:
      - amd64
      - arm64
    ldflags:
      - -s -w
      - -X main.version={{.Version}}
      - -X main.commit={{.Commit}}
      - -X main.date={{.Date}}

archives:
  - formats:
      - tar.gz
    format_overrides:
      - goos: windows
        formats:
          - zip
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"

checksum:
  name_template: "checksums.txt"

release:
  github:
    owner: grokify
    name: your-tool

brews:
  - name: your-tool
    repository:
      owner: grokify
      name: homebrew-tap
      token: "{{ .Env.HOMEBREW_TAP_GITHUB_TOKEN }}"
    directory: Formula
    homepage: "https://github.com/grokify/your-tool"
    description: "Your tool description"
    license: "MIT"
    install: |
      bin.install "your-tool"
    test: |
      system "#{bin}/your-tool", "version"
    commit_author:
      name: goreleaserbot
      email: bot@goreleaser.com
```

### 4. Create GitHub Actions Release Workflow

Create `.github/workflows/release.yaml`:

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

permissions:
  contents: write

jobs:
  goreleaser:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: "1.23"

      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v6
        with:
          distribution: goreleaser
          version: "~> v2"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          HOMEBREW_TAP_GITHUB_TOKEN: ${{ secrets.HOMEBREW_TAP_GITHUB_TOKEN }}
```

### 5. Release

```bash
git add .goreleaser.yaml .github/workflows/release.yaml
git commit -m "build: add GoReleaser and Homebrew release workflow"
git push
git tag v1.0.0
git push origin v1.0.0
```

GoReleaser will automatically create/update the formula in this tap repository.

## License

MIT
