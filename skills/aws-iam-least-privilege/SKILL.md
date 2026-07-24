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

**IMPORTANT**: Always use `resource_changes[]` as the authoritative list — never
rely on `planned_values` which may omit resources. List every distinct `address`
before proceeding to ensure nothing is missed.

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

Always produce **both**:

1. A human-readable table grouped by resource address showing which IAM actions apply to each resource
2. A consolidated IAM policy JSON with scoped ARNs

**Resource ARN scoping (REQUIRED)**: Never use `"Resource": "*"` as a final
answer. Always scope to the narrowest ARN pattern possible:
- Use known values from the plan (region, resource names, AMI IDs, etc.)
- For IDs unknown at plan time, use a wildcard suffix: e.g.
  `arn:aws:ec2:REGION:ACCOUNT:instance/*`
- Replace `REGION` and `ACCOUNT` with actual values when known from the plan
  (e.g. region from provider config, account from existing ARNs in prior_state)
- Remind the user to replace `<account-id>` placeholders before use

## Examples

- `aws_s3_bucket` → `s3:CreateBucket`, `s3:PutBucketVersioning`
- `aws_lambda_function` + `aws_iam_role` → `lambda:CreateFunction`, `iam:PassRole`
