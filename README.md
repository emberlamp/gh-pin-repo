# gh-pin-repo

GitHub CLI extension to pin repositories to your organization profile.

## Installation

```bash
gh extension install emberlamp/gh-pin-repo
```

## Usage

### Pin repositories
```bash
gh pin-repo emberlamp/general emberlamp/react-template
```

### List pinned repositories
```bash
gh pin-repo --list
```

### Get help
```bash
gh pin-repo --help
```

## Requirements

- GitHub CLI (gh) 2.0+
- Organization admin access to pin repos

## Note

Pinning repositories requires specific GraphQL permissions. If you encounter errors, ensure you have org admin privileges.