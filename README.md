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

No write, delete, or account-key-listing permissions were included — by design.

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

Signed in as the test user directly (not the admin account) and attempted to browse the storage account via Storage Browser, which relies on account-key-based listing — an action outside the custom role's scope. The attempt was correctly denied, confirming the user has **no implicit access beyond what was explicitly granted** in the custom role.

![Access denied error from the test user's session](Screenshot%202026-08-27%20163548.png)

> **Note on scope:** This denial confirms the absence of broader, ungranted access (account-key-based listing). It does not by itself demonstrate the specific `blobs/read` data action succeeding — that would require a follow-up test using Azure AD auth mode against a known blob path (e.g., `az storage blob download --auth-mode login`), which is a planned next step for this lab.

## What This Demonstrates

- Authoring least-privilege RBAC roles as code (JSON + CLI) rather than manual portal configuration
- Correctly scoping a custom role to only the actions required
- Validating an assignment from the assigned identity's own session, not just from an admin's view
- Reading and interpreting Azure authorization error messages to confirm enforcement

## Next Steps

- Verify the granted `blobs/read` data action succeeds via Azure AD auth mode CLI calls
- Reproduce the role definition and assignment in Terraform for full IaC parity with other projects in this portfolio
