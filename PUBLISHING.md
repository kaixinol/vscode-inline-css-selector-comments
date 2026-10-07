# Publishing

Azure DevOps global Personal Access Tokens are retired on **2026-12-01**. This extension
publishes through the GitHub Actions workflow in `.github/workflows/publish.yml`, which
authenticates with a **federated managed identity** via Microsoft Entra ID. No long-lived
secret is stored in the repository.

One-time setup, then publishing is a button click.

## One-time setup

### 1. Create a user-assigned managed identity

In the [Azure portal](https://portal.azure.com), search for **Managed Identities** and create
one. You need an active Azure subscription (a free-tier one is enough).

> Use a **managed identity**, not an app registration. An app registration authenticates
> fine but then fails at the publish step with
> `InvalidAccessException: The requested operation is not allowed`.

Record the **Client ID** and **Tenant ID** from the identity's Properties page.

### 2. Add a federated credential trusting this repository

On the same managed identity: **Settings → Federated credentials → + Add credential**.

| Field | Value |
| --- | --- |
| Federated credential scenario | GitHub Actions deploying Azure resources |
| Organization | `kaixinol` |
| Repository | `vscode-inline-css-selector-comments` |
| Entity type | **Environment** |
| GitHub environment name | `marketplace-publish` |

Use **Environment**, not Branch or Tag. A tag-scoped credential only ever matches that one
exact tag name and breaks on the second release.

The environment named here must also be created in the repository:
**Settings → Environments → New environment → `marketplace-publish`**. The workflow declares
it via `environment:`, and it must exist or the job will not start.

### 3. Add the repository secrets

In the repository: **Settings → Secrets and variables → Actions → New repository secret**.

| Name | Value |
| --- | --- |
| `AZURE_CLIENT_ID` | Client ID from step 1 |
| `AZURE_TENANT_ID` | Tenant ID from step 1 |

`azure/login` reads exactly these two names.

### 4. Authorize the identity on the Marketplace

The Marketplace keeps its own identity record, separate from the Azure resource ID and the
Entra object ID. A managed identity that has never authenticated has no profile there yet, so
adding it by either of the other IDs returns "not found". Retrieve the ID the Marketplace
expects by making the call once, authenticated as the identity:

```bash
az rest -u https://app.vssps.visualstudio.com/_apis/profile/profiles/me \
  --resource 499b84ac-1321-427f-aa17-267ca6975798
```

Take the `id` field from the JSON output. Then, on
[marketplace.visualstudio.com/manage](https://marketplace.visualstudio.com/manage), add that
managed identity as a member of the publisher and assign it the **Contributor** role.

## Publishing

Go to **Actions → Publish → Run workflow**.

- **dry-run** (default) packages the VSIX without uploading. Use it to verify the pipeline.
- Uncheck **dry-run** to actually publish.

The workflow fails on purpose if a `v*` tag disagrees with the `package.json` version.

To confirm it landed:

```bash
npx @vscode/vsce ls --publisher kaesinol
```

## Local packaging

Packaging needs no credentials at all:

```bash
npm install -g @vscode/vsce
vsce package --no-git-tag-version --skip-license
```

Install the resulting VSIX locally via **Extensions: Install from VSIX...** in the Command
Palette. Note the Marketplace rejects SVG images in README/CHANGELOG, so screenshots must be
PNG.
