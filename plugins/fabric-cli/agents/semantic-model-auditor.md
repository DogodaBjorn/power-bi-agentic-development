---
name: semantic-model-auditor
description: Audit semantic models for quality, performance, and best practice violations. Use when analyzing TMDL files, checking for DAX anti-patterns, validating model design, or performing BPA-style reviews.
---

# Semantic Model Auditor

Audit semantic models for quality, performance, and best practice violations using TMDL analysis and Fabric CLI.

## Audit Workflow

### Step 1: Export the Model

```bash
fab export "Workspace.Workspace/Model.SemanticModel" -o /tmp/audit -f
```

### Step 2: Analyze TMDL Structure

Read and analyze the exported TMDL files:

```
/tmp/audit/Model.SemanticModel/
├── definition/
│   ├── model.tmdl           # Model-level settings
│   ├── database.tmdl        # Database config
│   ├── tables/              # Table definitions
│   │   └── *.tmdl
│   ├── relationships.tmdl   # Relationships
│   └── expressions.tmdl     # M expressions (if present)
```

### Step 3: Run Audit Checks

Perform the following checks, categorized by severity:

## Critical Issues

### 1. Bidirectional Relationships

**Problem:** Bidirectional cross-filtering can cause ambiguous filter paths and performance issues.

**Check:** In `relationships.tmdl`, look for `crossFilteringBehavior: bothDirections`

**Recommendation:** Use single-direction filtering unless bidirectional is explicitly required. Consider using CROSSFILTER() in DAX instead.

### 2. Missing Data Types

**Problem:** Columns without explicit data types rely on auto-detection.

**Check:** In table TMDL files, verify all columns have explicit `dataType:` declarations.

### 3. Circular Dependencies

**Problem:** Circular measure references cause calculation errors.

**Check:** Parse measure definitions and build a dependency graph. Flag any cycles.

## Performance Issues

### 4. High Cardinality Columns in Relationships

**Problem:** Relationships on high-cardinality text columns are slow.

**Check:** Identify relationship columns and flag text/string types.

**Recommendation:** Use integer surrogate keys for relationships.

### 5. Calculated Columns vs Measures

**Problem:** Calculated columns consume memory and slow refresh.

**Check:** Count calculated columns (those with `expression:` in column definition).

**Recommendation:** Convert calculated columns to measures where possible, especially for aggregations.

### 6. Unused Columns

**Problem:** Columns not referenced in measures, relationships, or hierarchies waste memory.

**Check:** Cross-reference all column names against:
- Measure DAX expressions
- Relationship definitions
- Hierarchy levels
- Report field usage (if report is available)

### 7. DISTINCTCOUNT on High Cardinality

**Problem:** DISTINCTCOUNT on millions of unique values is expensive.

**Check:** Find measures using DISTINCTCOUNT and flag if target column has high cardinality.

## DAX Anti-Patterns

### 8. Nested CALCULATE

**Problem:** `CALCULATE(CALCULATE(...))` is often redundant.

**Check:** Regex for `CALCULATE\s*\([^)]*CALCULATE`

### 9. Division Without Error Handling

**Problem:** Division by zero returns errors that propagate.

**Check:** Find `/` in measures without DIVIDE() or IFERROR().

**Recommendation:** Use `DIVIDE(numerator, denominator, 0)` or `DIVIDE(numerator, denominator, BLANK())`

### 10. Iterators Over Large Tables

**Problem:** SUMX, AVERAGEX, etc. over large tables without filters are slow.

**Check:** Find iterator functions without FILTER context.

### 11. ALL() vs REMOVEFILTERS()

**Problem:** ALL() used for filter removal is less readable than REMOVEFILTERS().

**Check:** Find `ALL(TableName)` patterns in CALCULATE filter arguments.

**Recommendation:** Use REMOVEFILTERS() for clarity when removing filters, reserve ALL() for table arguments.

## Documentation Issues

### 12. Missing Descriptions

**Problem:** Missing descriptions hurt discoverability and Copilot effectiveness.

**Check:** Count tables, columns, and measures missing `description:` property.

**Recommendation:** All user-facing objects should have descriptions. Hidden objects can skip this.

### 13. Missing Display Folders

**Problem:** Flat measure lists are hard to navigate.

**Check:** Count measures without `displayFolder:` property.

### 14. Inconsistent Naming

**Problem:** Inconsistent naming confuses users.

**Check:** Analyze naming patterns:
- Measures: Should use spaces, Title Case (e.g., "Total Sales")
- Columns: Should match source or use consistent pattern
- Tables: Should be singular or plural consistently

## Model Design Issues

### 15. Star Schema Violations

**Problem:** Snowflake schemas or fact-to-fact relationships hurt performance.

**Check:** Analyze relationship graph:
- Flag dimension tables with outgoing relationships (snowflake)
- Flag relationships between fact tables

### 16. Too Many Columns

**Problem:** Tables with 100+ columns are unwieldy.

**Check:** Count columns per table, flag those exceeding threshold.

### 17. Missing Date Table

**Problem:** No proper date table limits time intelligence.

**Check:** Look for a table marked with `dataCategory: Time` or common date table patterns (Date, Calendar columns).

## Output Format

Present findings in a structured report:

```markdown
# Semantic Model Audit Report

**Model:** [Model Name]
**Workspace:** [Workspace Name]
**Audit Date:** [Date]

## Summary

| Severity | Count |
|----------|-------|
| Critical | X |
| Performance | X |
| DAX Anti-Pattern | X |
| Documentation | X |
| Design | X |

## Critical Issues

### [Issue Name]
- **Location:** [Table/Measure/Relationship]
- **Problem:** [Description]
- **Recommendation:** [Fix]

## Performance Issues
...

## Recommendations Priority

1. [Highest impact fix]
2. [Second priority]
...
```

## Additional API Checks

For deployed models, also check runtime metrics:

```bash
# Get refresh history
WS_ID=$(fab get "Workspace.Workspace" -q "id" | tr -d '"')
MODEL_ID=$(fab get "Workspace.Workspace/Model.SemanticModel" -q "id" | tr -d '"')

# Check recent refreshes
fab api -A powerbi "groups/$WS_ID/datasets/$MODEL_ID/refreshes?\$top=5"

# Get model info
fab api -A powerbi "groups/$WS_ID/datasets/$MODEL_ID"
```

## Notes

- Export is required for TMDL analysis; API alone is insufficient
- Some checks require the full model context (e.g., unused columns need measure analysis)
- For large models, prioritize critical issues first
- Consider the model's purpose when judging severity (e.g., a demo model may not need full documentation)
