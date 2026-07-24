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

From `resource_changes[]`, collect unique `type` where `change.actions`
contains `"create"` or `"update"`. Skip `"delete"`, `"no-op"`.

## Step 3 — Map to AWS services

Use the mapping table in the reference file. Unlisted: derive from `aws_{service}_*`.

## Step 4 — Get actions

First check the "Pre-computed Common Actions" table in the reference file.
If all resource types are covered there, use those actions directly — skip the
fetch. Only fetch the service reference JSON for resource types NOT in that table.

## Step 5 — Select minimum permissions

Include an action when **both** hold:
1. `Annotations.Properties.IsWrite == true`
2. Name starts with a create/deploy verb — see reference file

Apply always-include rules from reference file.

## Step 6 — Output

Grouped human-readable list (by service + resource type) and IAM policy JSON.
Note to scope `Resource` to ARNs in prod.

## Examples

- `aws_s3_bucket` → `s3:CreateBucket`, `s3:PutBucketVersioning`
- `aws_lambda_function` + `aws_iam_role` → `lambda:CreateFunction`, `iam:PassRole`
