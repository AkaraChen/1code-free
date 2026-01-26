# Build Free Release

This document describes the automated build process for creating free macOS releases of 1Code.

## Prerequisites

- [Bun](https://bun.sh)
- Python (for native module compilation)
- Xcode Command Line Tools (macOS)
- [GitHub CLI](https://cli.github.com) (`gh`) — authenticated with `gh auth login`

## Remote Configuration

- **upstream**: https://github.com/21st-dev/1code.git
- **origin**: https://github.com/AkaraChen/1code-free.git

## Build Process

> **Specify the upstream tag** (e.g., `v0.0.27`) when running this process.

### 1. Pull from upstream

```bash
# Example: building from tag v0.0.27
git fetch upstream
git stash  # stash bun.lock changes if needed
git checkout v0.0.27
git checkout main && git rebase v0.0.27
```

**Why rebase?** Your local changes are typically in the `free/` directory only. Rebase keeps your commits on top of the upstream tag.

**Note**: `bun.lock` may have local changes after `bun install`. Stash or discard these changes before checkout — do not commit them.

### 2. Install dependencies

See [README](../README.md#installation) for detailed instructions.

```bash
bun install
```

This runs `postinstall` which rebuilds native dependencies (`better-sqlite3`, `node-pty`).

### 3. Download Claude CLI binary

```bash
bun run claude:download
```

**Required!** This downloads the Claude CLI binary to `resources/bin/`. Without this, the app will build but agent functionality won't work.

### 4. Build TypeScript

```bash
bun run build
```

Compiles TypeScript for main, preload, and renderer processes using electron-vite. Output goes to `out/`.

### 5. Build macOS packages (unsigned)

```bash
bun run package:mac
```

**Note**: This build is **unsigned**. Users will see an "unidentified developer" warning on first launch.

### 6. Create release on origin

Check the version in `package.json` (should match the upstream tag).

```bash
# Example: for v0.0.27
gh release create v0.0.27 \
  --repo AkaraChen/1code-free \
  --title "1Code v0.0.27" \
  --notes "Release v0.0.27"
```

### 7. Upload DMG files

List the built files in the `release/` directory:

```bash
ls release/
```

Upload all `.dmg` files to the GitHub release:

```bash
# Example: for v0.0.27
gh release upload v0.0.27 release/1Code-0.0.27-arm64.dmg release/1Code-0.0.27.dmg --repo AkaraChen/1code-free
```
