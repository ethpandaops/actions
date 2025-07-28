# Claude PR Assistant Composite Action

This composite action provides a standardized Claude PR assistant configuration for ethPandaOps repositories.

## Benefits of This Approach

1. **Centralized Configuration**: All Claude PR logic is maintained in one place
2. **Easy Updates**: Update the composite action, and all workflows automatically get the changes
3. **Consistent Behavior**: Ensures all repos use the same Claude configuration
4. **Simple Installation**: Users just need to add the workflow template

## Usage

This action is designed to be used through the workflow template. Users should:

1. Go to Actions → New workflow
2. Select "Claude PR Assistant" from the templates
3. Commit the workflow file

The workflow will automatically use this composite action.

## Direct Usage (Advanced)

If you need to use this action directly:

```yaml
- uses: ethpandaops/workflow-templates/actions/claude-pr-assistant@main
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    timeout_minutes: '60'  # optional
    node_version: '23'     # optional
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `claude_code_oauth_token` | OAuth token for Claude Code | Yes | - |
| `timeout_minutes` | Timeout for Claude operations | No | `60` |
| `node_version` | Node.js version to use | No | `23` |
| `custom_instructions` | Additional instructions for Claude | No | Default PR instructions |
| `allowed_tools` | List of allowed tools | No | Standard tool set |

## Updating the Action

To update the composite action:

1. Make changes to `action.yml`
2. Commit and push to main
3. All workflows using `@main` will automatically get the update
4. For versioned releases, create a tag (e.g., `v1.0.0`)

## Versioning Strategy

- `@main`: Always uses latest