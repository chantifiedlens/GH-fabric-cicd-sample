# GH-fabric-cicd-sample to operationalize fabric-cicd to work with Microsoft Fabric and GitHub Actions
This repository contains sample GitHub Actions workflows that demonstrate how to deploy Microsoft Fabric assets using either the Python `fabric-cicd` library or the Microsoft Fabric CLI (`ms-fabric-cli`).

The repository includes three workflows in `/.github/workflows`:
- `fabric-cicd-demo-variables.yml`
- `Fabric-CLI-fabric-cicd.yml`
- `OIDC-fab-deploy.yml`

The `fabric-cicd-demo-variables.yml` workflow is based on a blog post that covers how to [operationalize fabric-cicd to work with Microsoft Fabric and GitHub Actions](https://ChantifiedLens.com/2025/04/11/operationalize-fabric-cicd-to-work-with-Microsoft-Fabric-and-GitHub-Actions/).

## Workflow summary

### `fabric-cicd-demo-variables.yml`
This workflow uses the Python `fabric-cicd` library directly to deploy Fabric artifacts into a Test environment first, then into Prod.

Key behaviors:
- Runs on `workflow_dispatch` (manual trigger).
- Installs `fabric-cicd` with Python 3.11.
- Authenticates with Azure using a service principal and secret.
- Uses `auth_spn_secret_AzDo.py` to deploy the workspace items from the local `workspace/` folder.

Required repository secrets:
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TENANT_ID`

Required repository variables:
- `ItemsInScope` (for example: `Notebook,Environment,Report,SemanticModel`)
- `TestWorkspace` (GUID of the test workspace)
- `ProdWorkspace` (GUID of the production workspace)

Notes:
- The workflow uses the Azure PowerShell `Az.Accounts` module to obtain a Fabric access token.
- It is suitable for deployments where you want to control test and prod deployments from the same workflow.
- The `fabric-cicd` docs are available at https://microsoft.github.io/fabric-cicd/latest/.

### `Fabric-CLI-fabric-cicd.yml`
This workflow combines Microsoft Fabric CLI (`ms-fabric-cli`) with the `fabric-cicd` Python library.

Key behaviors:
- Runs on `workflow_dispatch` and accepts inputs for `WorkspaceName`, `CapacityName`, and `EntraObjectId`.
- Installs `ms-fabric-cli` and `fabric-cicd` with Python 3.12.
- Authenticates the Fabric CLI with a service principal secret using `fab auth login`.
- Creates a Fabric workspace, assigns access with `fab acl set`, then deploys workspace items with `fabric-cicd`.

Required repository secrets:
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TENANT_ID`

Required repository variables:
- `ItemsInScope`

Notes:
- This workflow is useful when you want to create or populate a Fabric workspace before deploying content.
- It demonstrates using `fab create`, `fab acl set`, `fab get`, and then `python auth_spn_secret_AzDo.py`.
- See the Fabric CLI docs for `ms-fabric-cli` at https://microsoft.github.io/fabric-cli/.
- The `fabric-cicd` docs for deployment scripts are at https://microsoft.github.io/fabric-cicd/latest/.

### `OIDC-fab-deploy.yml`
This workflow uses GitHub OIDC to authenticate securely with the Fabric CLI, avoiding long-lived Azure secrets in the workflow.

Key behaviors:
- Triggers on `push` and `pull_request` events for the `main` branch.
- Requests GitHub `id-token: write` permission for OIDC.
- Installs `ms-fabric-cli` and authenticates using a federated token.
- Runs `fab deploy --config "./workspace/config.yml" --target_env "${{vars.TargetEnvironment}}" -f`.

Required repository secrets:
- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`

Required repository variables:
- `TargetEnvironment`

Important security notes:
- Azure must be configured with a federated credential that trusts this GitHub repository and branch.
- The service principal used for OIDC should have least privilege.
- This workflow is a stronger production pattern than using client secrets, but it still requires proper Azure-side trust and repo protection.
- The `fab deploy` command docs are available in the Fabric CLI documentation at https://microsoft.github.io/fabric-cli/.

## Repository contents and deployment inputs

The sample workspace content under `workspace/` is based on the original [fabric-cicd repository](https://github.com/microsoft/fabric-cicd). The sample files include reports, semantic models, notebooks, and environment definitions.

Customize `workspace/parameter.yml` as needed for your own Fabric environment.

## Recommended links
- Microsoft Fabric CLI docs: https://microsoft.github.io/fabric-cli/
- Fabric CLI `fab deploy` command docs: https://microsoft.github.io/fabric-cli/commands/deploy/
- Fabric CICD docs: https://microsoft.github.io/fabric-cicd/latest/
- Original fabric-cicd repo: https://github.com/microsoft/fabric-cicd
- Blog post on operationalizing fabric-cicd with GitHub Actions: https://ChantifiedLens.com/2025/04/11/operationalize-fabric-cicd-to-work-with-Microsoft-Fabric-and-GitHub-Actions/

## Disclaimer
This repository is provided "as is" based on the [MIT license](https://opensource.org/licenses/MIT). It is a sample implementation and not a fully managed production deployment. Ensure you verify and secure Azure identities, permissions, and GitHub workflow triggers before using any workflow in production.
