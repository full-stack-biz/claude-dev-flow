# Installation & Setup

## Install git-flow-next

### macOS (Homebrew - Recommended)

```bash
brew install gittower/tap/git-flow-next
```

Verify installation:
```bash
git-flow --help
```

### Linux/Windows (Manual Download)

1. Download binary from [GitHub Releases](https://github.com/gittower/git-flow-next/releases)
2. Extract to a directory in your PATH (e.g., `/usr/local/bin/`)
3. Make executable:
   ```bash
   chmod +x /usr/local/bin/git-flow
   ```
4. Verify:
   ```bash
   git-flow --help
   ```

### From Source

```bash
git clone https://github.com/gittower/git-flow-next.git
cd git-flow-next
go build -o git-flow
sudo mv git-flow /usr/local/bin/
```

## Initialize in Your Repository

### Option A: Interactive (Recommended)

```bash
git-flow init
```

Prompts you to select preset (classic, github, gitlab) and confirm branch names.

### Option B: With Preset

```bash
# Classic GitFlow (feature/develop/main)
git-flow init --preset=classic --defaults

# GitHub Flow (feature/main)
git-flow init --preset=github --defaults

# GitLab Flow (feature/develop/staging/production)
git-flow init --preset=gitlab --defaults
```

### Option C: Custom Configuration

```bash
git-flow init --custom
```

Then configure branches interactively.

### Option D: All Arguments

```bash
git-flow init --preset=classic --defaults \
  --main=main \
  --develop=develop \
  --feature=feature/ \
  --release=release/ \
  --hotfix=hotfix/ \
  --tag=v
```

## Verify Setup

Check configuration:
```bash
git-flow config list
```

View active branches:
```bash
git-flow overview
```
