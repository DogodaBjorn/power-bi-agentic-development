---
description: Deploy local items to Fabric workspace
argument-hint: [workspace/item] [local-path]
---

# Deploy Fabric Items

Deploy local items to Fabric: $ARGUMENTS

## Basic Usage

```bash
fab import "Workspace.Workspace/ItemName.ItemType" -i ./local/ItemName.ItemType -f
```

## Item Types

| Type | Local Format | Notes |
|------|--------------|-------|
| `.Report` | PBIR folder | Must have `.pbir` file |
| `.SemanticModel` | TMDL folder | Must have `model.tmdl` |
| `.Notebook` | `.py` or folder | Fabric notebook format |
| `.DataPipeline` | JSON folder | Pipeline definition |

## Deploy Reports

```bash
fab import "Workspace.Workspace/Report.Report" -i ./local/Report.Report -f
```

Creates or updates the report in Fabric.

## Deploy Semantic Models

```bash
fab import "Workspace.Workspace/Model.SemanticModel" -i ./local/Model.SemanticModel -f
```

**Note:** Deploying a semantic model triggers a metadata update. Data refresh may be needed after.

## Deploy New Items

```bash
# Create new report
fab import "Workspace.Workspace/NewReport.Report" -i ./local/Report.Report -f

# Create new model
fab import "Workspace.Workspace/NewModel.SemanticModel" -i ./local/Model.SemanticModel -f
```

## Bulk Deploy

```bash
# Deploy all items in directory
for item in ./exports/*; do
  name=$(basename "$item")
  fab import "Workspace.Workspace/$name" -i "$item" -f
done
```

## Environment Migration

```bash
# Download from dev
fab export "Dev.Workspace/Report.Report" -o ./staging -f

# Deploy to prod
fab import "Prod.Workspace/Report.Report" -i ./staging/Report.Report -f
```

## Options

- `-i, --input` - Local path to item (required)
- `-f, --force` - Overwrite without confirmation

## Notes

- Item type must match between local and target
- Semantic model connections may need rebinding after deploy
- Reports deployed without their model need rebinding: `fab set "Workspace.Workspace/Report.Report" -q semanticModelId -i "<model-id>"`
