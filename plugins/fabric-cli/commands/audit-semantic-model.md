---
description: Audit a semantic model for quality, performance, and best practices
argument-hint: [workspace/model]
---

# Audit Semantic Model

Perform a comprehensive audit of the specified semantic model: $ARGUMENTS

## Instructions

Use the `semantic-model-auditor` agent to analyze this model. The audit should:

1. **Export the model** to a temporary directory using `fab export`
2. **Read and analyze TMDL files** for structural issues
3. **Check for DAX anti-patterns** in measure definitions
4. **Validate model design** against star schema best practices
5. **Report findings** organized by severity

## Audit Categories

- **Critical:** Bidirectional relationships, circular dependencies, missing data types
- **Performance:** High cardinality columns, calculated columns, unused objects
- **DAX:** Nested CALCULATE, division without DIVIDE(), inefficient iterators
- **Documentation:** Missing descriptions, display folders, inconsistent naming
- **Design:** Star schema violations, missing date table, excessive columns

## Expected Output

A structured markdown report with:

- Summary counts by severity
- Detailed findings with locations and recommendations
- Prioritized remediation list

If workspace/model not provided, prompt for the target model path.
