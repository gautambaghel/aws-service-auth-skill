---
name: aws-iam-least-privilege
description: >
  Derives minimum AWS IAM permissions to deploy Terraform resources.
  USE FOR: least-privilege IAM from terraform plan, permissions for
  terraform apply, IAM policy from plan, minimum IAM for terraform,
  HCP Terraform or TFE IAM permissions.
  DO NOT USE FOR: general IAM questions, auditing running resources,
  non-AWS providers.
---

# AWS IAM Least-Privilege from Terraform Plan

**UTILITY SKILL** — Minimum IAM write permissions to deploy a Terraform plan.

Reference data (service map, tfctl commands, URLs, verb list, always-include
rules): [references/aws-service-map.md](references/aws-service-map.md). Read it first.

## Step 1 — Get the plan JSON

If not provided, ask the user to share it. See "Plan Acquisition" in the
reference file (local terraform, HCP Terraform/TFE via tfctl, Terraform MCP).
Do not proceed without the plan JSON.

## Step 2 — Extract resource types

From `resource_changes[]` (authoritative — never use `planned_values`), collect
unique `type` where `change.actions` contains `"create"` or `"update"`.
Skip `"delete"`, `"no-op"`. List every distinct `address` before proceeding.

## Step 3 — Map to AWS services

Use the mapping table in the reference file. Unlisted: derive from `aws_{service}_*`.

## Step 4 — Get actions

Check "Pre-computed Common Actions" in the reference file first; use those
directly if covered, else fetch service reference JSON for unlisted types.

## Step 5 — Select minimum permissions

Include an action when **both** hold:
1. `Annotations.Properties.IsWrite == true`
2. Name starts with a create/deploy verb — see reference file

Apply always-include rules from reference file.

## Step 6 — Output

Produce **both**:

1. Human-readable list grouped by resource address with IAM actions
2. IAM policy JSON with scoped ARNs — see "ARN Scoping" in the reference file.
   Never use `"Resource": "*"` as a final answer.
