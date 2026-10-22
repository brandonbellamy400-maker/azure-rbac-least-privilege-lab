# azure-rbac-least-privilege-lab
Designed and deployed a custom Azure RBAC role scoped to read-only storage access, assigned it to a test user, and validated enforcement from that user's live session.
# Azure RBAC Least-Privilege Lab

Designed and deployed a custom Azure RBAC role scoped to read-only storage access, assigned it to a test user, and validated the assignment from that user's live session — demonstrating the full loop from role definition to real-world enforcement.

## Overview

Azure RBAC is often configured using built-in roles, which are frequently broader than a given identity actually needs. This lab demonstrates writing a **custom role from scratch**, scoped to the minimum permissions required, deployed as code rather than clicked together in the portal — then verifying the assignment behaves as intended from the assigned user's own session.

## What I Built

A custom role, **"Storage Read and List Operator,"** granting:
- `Microsoft.Storage/storageAccounts/read`
- `Microsoft.Storage/storageAccounts/listKeys/action`
- `Microsoft.Resources/subscriptions/resourceGroups/read`
- Data action: `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read`

No write or delete permissions were included.

> **Correction (found on review):** The role *does* include `listKeys/action`. That was a mistake for a least-privilege role: the account key gives full read/write access to all data in the storage account, which bypasses the narrow `blobs/read` data action entirely. The fix is to remove `listKeys/action` and rely only on the data action with Entra ID (Azure AD) authentication. See Next Steps.

## Steps

### 1. Define the role as code

The role was authored as a JSON definition and deployed via Azure CLI rather than the portal UI, so the permission set is version-controllable and auditable.

```bash
az role definition create --role-definition storage-reader-role.json
```

![Custom role definition JSON and CLI output](Screenshot%202026-08-27%20160303.png)

### 2. Assign the role to a test identity

The custom role was assigned to a test user ("Adam") scoped to the storage account.

![Add role assignment in the Azure portal](Screenshot%202026-08-27%20160722.png)

### 3. Confirm the assignment

Access Control (IAM) on the storage account confirms the assignment took effect, scoped to "This resource" — shown alongside other existing role assignments for context.

![Access Control (IAM) showing the assigned role](Screenshot%202026-08-27%20160946.png)

### 4. Verify from the assigned user's session

Signed in as the test user directly (not the admin account) and opened the storage account in Storage Browser. The portal showed an authorization error: the user was not allowed to perform `Microsoft.Storage/storageAccounts/blobServices/read`. That action was not in the custom role, so the denial confirms Azure only allowed what was explicitly granted.

![Access denied error from the test user's session](Screenshot%202026-08-27%20163548.png)

> **Note on scope:** This test shows an ungranted action being denied. It does not prove the granted `blobs/read` data action succeeds. That needs a follow-up test with Entra ID auth against a known blob path (for example, `az storage blob download --auth-mode login`), which is a planned next step.

## What This Demonstrates

- Authoring least-privilege RBAC roles as code (JSON + CLI) rather than manual portal configuration
- Scoping a custom role to specific actions, and reviewing it afterward to catch an over-broad permission (`listKeys`)
- Validating an assignment from the assigned identity's own session, not just from an admin's view
- Reading and interpreting Azure authorization error messages to confirm enforcement

## Next Steps

- Remove `listKeys/action` from the role definition so access depends only on the `blobs/read` data action
- Verify the granted `blobs/read` data action succeeds via Azure AD auth mode CLI calls
- Reproduce the role definition and assignment in Terraform for full IaC parity with other projects in this portfolio
