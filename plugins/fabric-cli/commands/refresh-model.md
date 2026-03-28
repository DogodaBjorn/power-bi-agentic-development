---
description: Trigger semantic model refresh via Power BI API
argument-hint: [workspace/model]
---

# Refresh Semantic Model

Trigger a refresh for the specified semantic model: $ARGUMENTS

## Steps

1. **Extract workspace and model IDs:**

```bash
# Get workspace ID
WS_ID=$(fab get "WorkspaceName.Workspace" -q "id" | tr -d '"')

# Get model ID
MODEL_ID=$(fab get "WorkspaceName.Workspace/ModelName.SemanticModel" -q "id" | tr -d '"')
```

2. **Trigger the refresh:**

```bash
fab api -A powerbi "groups/$WS_ID/datasets/$MODEL_ID/refreshes" -X post -i '{"type":"Full"}'
```

Refresh types:

- `Full` - Full refresh of all tables
- `Automatic` - Incremental refresh if configured, else full

3. **Monitor refresh status:**

```bash
fab api -A powerbi "groups/$WS_ID/datasets/$MODEL_ID/refreshes?\$top=1"
```

Status values: `Unknown`, `Completed`, `Failed`, `Disabled`, `InProgress`

## Quick One-Liner

```bash
WS="WorkspaceName" MODEL="ModelName" && \
WS_ID=$(fab get "$WS.Workspace" -q "id" | tr -d '"') && \
MODEL_ID=$(fab get "$WS.Workspace/$MODEL.SemanticModel" -q "id" | tr -d '"') && \
fab api -A powerbi "groups/$WS_ID/datasets/$MODEL_ID/refreshes" -X post -i '{"type":"Full"}'
```

## Notes

- Requires workspace contributor or higher permissions
- Enhanced refresh (with `commitMode`, `maxParallelism`) requires Premium/Fabric capacity
- Check refresh history: `fab api -A powerbi "groups/$WS_ID/datasets/$MODEL_ID/refreshes?\$top=5"`
