---
description: Download Fabric items to local directory
argument-hint: [workspace/item] [output-path]
---

# Download Fabric Items

Download items from Fabric: $ARGUMENTS

## Basic Usage

```bash
fab export "Workspace.Workspace/ItemName.ItemType" -o ./output -f
```

## Item Types

| Type | Format | Open With |
|------|--------|-----------|
| `.Report` | PBIR | Double-click `.pbir` file |
| `.SemanticModel` | TMDL | Power BI Desktop, Tabular Editor |
| `.Notebook` | Python/Spark | Fabric, VS Code |
| `.DataPipeline` | JSON | Fabric |
| `.Lakehouse` | Metadata only | Fabric |

## Download Reports

```bash
fab export "Workspace.Workspace/Report.Report" -o ./output -f
```

**To open:** Double-click the `.pbir` file in the output folder. Requires Power BI Desktop with Developer Mode enabled.

## Download Semantic Models

```bash
# Download as TMDL (default)
fab export "Workspace.Workspace/Model.SemanticModel" -o ./output -f

# Download as complete PBIP project
python3 scripts/export_semantic_model_as_pbip.py \
  "Workspace.Workspace" "Model.SemanticModel" ./output
```

The PBIP script creates a project structure that can be opened directly in Power BI Desktop.

## Download Entire Workspace

```bash
# All items
fab export "Workspace.Workspace" -o ./output -a -f

# Using helper script (with metadata)
python3 scripts/download_workspace.py "WorkspaceName" ./output
```

## Bulk Download by Type

```bash
# All semantic models
fab ls "Workspace.Workspace" | grep ".SemanticModel" | while read item; do
  fab export "Workspace.Workspace/$item" -o ./models -f
done

# All reports
fab ls "Workspace.Workspace" | grep ".Report" | while read item; do
  fab export "Workspace.Workspace/$item" -o ./reports -f
done
```

## Output Structure

```yaml
output/
├── Report.Report/
│   ├── .platform
│   ├── definition.pbir        # <-- Double-click to open
│   └── definition/
│       ├── report.json
│       └── pages/
└── Model.SemanticModel/
    ├── .platform
    └── definition/
        ├── model.tmdl
        ├── database.tmdl
        └── tables/
```

## Options

- `-o, --output` - Output directory (required)
- `-f, --force` - Overwrite without confirmation
- `-a, --all` - Download all items in workspace
