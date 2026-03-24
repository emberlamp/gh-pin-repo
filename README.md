# gh-pin-repo

GitHub CLI extension to pin repositories to your organization profile.

## ⚠️ Important Note

After extensive research and testing, the `updatePinnedItems` mutation **does not exist** in GitHub's API. I tried:

- GraphQL schema introspection - no such mutation found
- REST API endpoints - none available for pinning repos  
- Direct GraphQL queries - returns "Field doesn't exist" error
- Various GraphQL mutations for pins - only `pinIssue`, `pinIssueComment` exist (not for repos)

This is a known missing feature - GitHub users have requested this API for years with no resolution.

## Current Status

Repository pinning is only available through GitHub's web interface:
1. Go to your org profile (e.g., https://github.com/emberlamp)
2. Click "Customize your pins" in the right sidebar
3. Select up to 6 repositories

## Installation

```bash
gh extension install emberlamp/gh-pin-repo
```

## Usage

### List pinned repositories (works)
```bash
gh pin-repo --list
```

### Get help
```bash
gh pin-repo --help
```

## Requirements

- GitHub CLI (gh) 2.0+
- Organization owner to manually pin via web interface

## References

- [GitHub Community: Add GraphQL API mutation to manage profile pinned repositories](https://github.com/orgs/community/discussions/184845)
- [GitHub Docs: Pinning repositories to your organization's profile](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile)

## Proposed Solution

Add a mutation like:

```graphql
mutation {
  updateUserPinnedItems(input: { 
    itemIds: ["MDEwOlJlcG9zaXRvcnkxMjM0NTY3ODk=", "..."] 
  }) {
    user { 
      pinnedItems(first: 6) { 
        nodes { 
          ... on Repository { nameWithOwner } 
        } 
      } 
    }
  }
}
```

This would complement the existing read-only queries and make the profile pinning feature fully accessible via the API.