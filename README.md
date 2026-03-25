# gh-pin-repo

GitHub CLI extension to pin repositories to your organization profile.

## ⚠️ Important Note

After extensive research and testing, the `updatePinnedItems` mutation **does not exist** in GitHub's API. Here's exactly what I tried:

### Methods Attempted

1. **GraphQL Schema Introspection**
   ```bash
   gh api graphql -f query='{__schema{mutationType{name fields{name}}}}' --jq '.data.__schema.mutationType.fields[].name' | grep -i pin
   ```
   Output:
   ```
   pinEnvironment
   pinIssue
   pinIssueComment
   unpinIssue
   unpinIssueComment
   ```
   Result: Only found `pinEnvironment`, `pinIssue`, `pinIssueComment` - no repo pinning

2. **Search Pin Types in Schema**
   ```bash
   gh api graphql -f query='{__schema{types{name}}}' --jq '.data.__schema.types[].name' | grep -i pin
   ```
   Output:
   ```
   CancelSponsorshipInput
   CreateSponsorshipInput
   EnvironmentPinnedFilterField
   IssueCommentPinnedEvent
   IssueCommentUnpinnedEvent
   PinEnvironmentInput
   PinEnvironmentPayload
   PinIssueCommentInput
   PinIssueCommentPayload
   PinIssueInput
   PinIssuePayload
   Pinnable
   PinnableItem
   PinnableItemConnection
   ...
   ```
   Result: Found types like `Pinnable`, `PinnedIssue`, but no mutation to use them

3. **Try updatePinnedItems Mutation**
   ```bash
   gh api graphql -f query='mutation{updatePinnedItems(input:{pinnableId:"O_kgDOECK5jw", pinnedItemIds:["R_kgDORvpJ1w"]}){clientMutationId}}'
   ```
   Output:
   ```
   {"errors":[{"path":["mutation","updatePinnedItems"],"extensions":{"code":"undefinedField","typeName":"Mutation","fieldName":"updatePinnedItems"},"locations":[{"line":1,"column":12}],"message":"Field 'updatePinnedItems' doesn't exist on type 'Mutation'"}]}
   gh: Field 'updatePinnedItems' doesn't exist on type 'Mutation'
   ```
   Result: `Field 'updatePinnedItems' doesn't exist on type 'Mutation'`

4. **Try Proposed updateUserPinnedItems**
   ```bash
   gh api graphql -f query='mutation{updateUserPinnedItems(input:{itemIds:["R_kgDORvpJ1w"]}){user{pinnedItems{nodes{name}}}}}'
   ```
   Output:
   ```
   {"errors":[{"path":["mutation","updateUserPinnedItems"],"extensions":{"code":"undefinedField","typeName":"Mutation","fieldName":"updateUserPinnedItems"},"locations":[{"line":1,"column":12}],"message":"Field 'updateUserPinnedItems' doesn't exist on type 'Mutation'"}]}
   gh: Field 'updateUserPinnedItems' doesn't exist on type 'Mutation'
   ```
   Result: `Field 'updateUserPinnedItems' doesn't exist on type 'Mutation'`

5. **Check User/Org Pinnable Field**
   ```bash
   gh api graphql -f query='query{organization(login:"emberlamp"){pinnable{id}}}'
   ```
   Output:
   ```
   {"errors":[{"path":["query","organization","pinnable"],"extensions":{"code":"undefinedField","typeName":"Organization","fieldName":"pinnable"},"locations":[{"line":1,"column":39}],"message":"Field 'pinnable' doesn't exist on type 'Organization'"}]}
   gh: Field 'pinnable' doesn't exist on type 'Organization'
   ```
   Result: `Field 'pinnable' doesn't exist on type 'Organization/User'`

6. **REST API Check**
   ```bash
   gh api /orgs/emberlamp | grep -i pin
   ```
   Output:
   ```
   (no output - no pin-related endpoints found)
   ```
   Result: No pin-related endpoints found

7. **Direct curl to GitHub API**
   ```bash
   curl -s -H "Authorization: bearer TOKEN" -H "Content-Type: application/json" -X POST -d '{"query":"mutation{updatePinnedItems(input:{pinnableId:\"O_kgDOECK5jw\", pinnedItemIds:[\"R_kgDORvpJ1w\"]}){clientMutationId}}"}' https://api.github.com/graphql
   ```
   Output:
   ```
   {"errors":[{"path":["mutation","updatePinnedItems"],"extensions":{"code":"undefinedField","typeName":"Mutation","fieldName":"updatePinnedItems"},"locations":[{"line":1,"column":10}],"message":"Field 'updatePinnedItems' doesn't exist on type 'Mutation'"}]}
   ```
   Result: Same result - mutation doesn't exist

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