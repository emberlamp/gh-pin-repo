# gh-pin-repo

GitHub CLI extension to pin repositories to your organization profile.

## ⚠️ Important Note

After extensive research and testing, the `updatePinnedItems` mutation **does not exist** in GitHub's API. Here's exactly what I tried:

### Methods Attempted

1. **GraphQL Schema Introspection**
   ```bash
   gh api graphql -f query='{__schema{mutationType{name fields{name}}}}' --jq '.data.__schema.mutationType.fields[].name' | grep -i pin
   ```
   Result: Only found `pinEnvironment`, `pinIssue`, `pinIssueComment` - no repo pinning

2. **Search Pin Types in Schema**
   ```bash
   gh api graphql -f query='{__schema{types{name}}}' --jq '.data.__schema.types[].name' | grep -i pin
   ```
   Result: Found types like `Pinnable`, `PinnedIssue`, but no mutation to use them

3. **Try updatePinnedItems Mutation**
   ```bash
   gh api graphql -f query='mutation{updatePinnedItems(input:{pinnableId:"O_kgDOECK5jw", pinnedItemIds:["R_kgDORvpJ1w"]}){clientMutationId}}'
   ```
   Result: `Field 'updatePinnedItems' doesn't exist on type 'Mutation'`

4. **Try Proposed updateUserPinnedItems**
   ```bash
   gh api graphql -f query='mutation{updateUserPinnedItems(input:{itemIds:["R_kgDORvpJ1w"]}){user{pinnedItems{nodes{name}}}}}'
   ```
   Result: `Field 'updateUserPinnedItems' doesn't exist on type 'Mutation'`

5. **Check User/Org Pinnable Field**
   ```bash
   gh api graphql -f query='query{organization(login:"emberlamp"){pinnable{id}}}'
   gh api graphql -f query='query{viewer{pinnable{id}}}'
   ```
   Result: `Field 'pinnable' doesn't exist on type 'Organization/User'`

6. **REST API Check**
   ```bash
   gh api /orgs/emberlamp | grep -i pin
   ```
   Result: No pin-related endpoints found

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
    itemIds: ["REPO_NODE_ID_1", "REPO_NODE_ID_2"] 
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