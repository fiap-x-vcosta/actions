# actions

Reusable workflows de CI/CD da org [`fiap-x-vcosta`](https://github.com/fiap-x-vcosta).

Os repositórios consumidores ficam só com callers finos apontando para `@main`. Um push nesta `main` atualiza todos. Este repositório **não** entra em `ci_repositories` (sem WIF próprio) e **não** expõe `tf-apply` / `tf-destroy` para o `infra-bootstrap` (apply do bootstrap continua só local, por owner).

Canon de produto: [`docs`](https://github.com/fiap-x-vcosta/docs).

## CI automático × CD manual (custo)

| Workflow | Gatilho típico | Automático? |
|----------|----------------|-------------|
| `ci-dotnet` / `ci-terraform` | PR / push | **Sim** |
| `build-push` | Merge em `main` do serviço | **Sim** |
| `deploy` / `tf-apply` / `tf-destroy` | `workflow_dispatch` no caller | **Manual** |

GKE, Cloud SQL e LBs só sobem na janela de demo. Não há apply/deploy em todo merge de propósito — evita custo GCP no trabalho contínuo.

## Workflows

| Arquivo | Uso |
|---------|-----|
| [`ci-dotnet.yml`](.github/workflows/ci-dotnet.yml) | Lint (InspectCode), unitários (gate 80% line/branch), integração, SonarCloud |
| [`ci-terraform.yml`](.github/workflows/ci-terraform.yml) | `fmt` / `validate`; `plan` opcional em PR |
| `build-push` | Build e push da imagem no Artifact Registry *(PR seguinte)* |
| `deploy` | Deploy no GKE (manifests em `k8s/`, Secret, rollout, smoke HTTPS) *(PR seguinte)* |
| `tf-apply` / `tf-destroy` | Apply/destroy manuais via caller `workflow_dispatch` *(PR seguinte)* |

Pin padrão do Terraform: **1.16.1**. Working directory padrão: **`terraform`**. Namespace K8s padrão do deploy: **`fiap-x`**.

## Exemplos de caller

### CI .NET (serviço)

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  ci:
    uses: fiap-x-vcosta/actions/.github/workflows/ci-dotnet.yml@main
    permissions:
      contents: read
      packages: read
    with:
      sonar_project_key: fiap-x-vcosta_autenticacao-service
      enable_github_packages: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

Com `enable_github_packages: true`, o caller precisa de `packages: read` no job. O reusable exporta `NUGET_AUTH_TOKEN=${{ github.token }}` nos jobs .NET para o `%NUGET_AUTH_TOKEN%` do `nuget.config` do serviço.

### CI Terraform (com plan em PR)

```yaml
name: CI

on:
  push:
  pull_request:

permissions:
  contents: read
  id-token: write

jobs:
  terraform:
    uses: fiap-x-vcosta/actions/.github/workflows/ci-terraform.yml@main
    with:
      run_plan: true
```

### CI Terraform (só fmt/validate — bootstrap)

```yaml
jobs:
  terraform:
    uses: fiap-x-vcosta/actions/.github/workflows/ci-terraform.yml@main
    with:
      run_plan: false
```

## Inputs principais

### `ci-dotnet`

| Input | Default | Obrigatório |
|-------|---------|-------------|
| `sonar_project_key` | — | sim |
| `solution` | `FiapX.sln` | não |
| `unit_project` | `tests/UnitTests/UnitTests.csproj` | não |
| `integration_project` | `tests/IntegrationTests/IntegrationTests.csproj` | não |
| `coverage_threshold` | `80` | não |
| `global_json_file` | `global.json` | não |
| `enable_github_packages` | `false` | não |

Secret: `SONAR_TOKEN`.

### `ci-terraform`

| Input | Default | Notas |
|-------|---------|-------|
| `terraform_version` | `1.16.1` | |
| `working_directory` | `terraform` | |
| `run_plan` | `true` | só em `pull_request` |
| `require_k8s_vars` | `false` | checa projeto/região/cluster e exporta `TF_VAR_*` |

## Deploy — secrets (contrato)

- **Org secrets (GitHub):** `DD_API_KEY`, `SERVICE_AUTH_KEY`.
- **Secret Manager (GCP):** um JSON por serviço (TF do serviço). Chaves literais ASP.NET, por exemplo `ConnectionStrings__DefaultConnection` e, só em `autenticacao`, `Jwt__SigningKeyPem`.

## Fora de escopo

- Apply/destroy do `infra-bootstrap`
- CI próprio do pacote `building-blocks` (não usa estes reusables)
- Código de aplicação, Terraform de nuvem e manifests dos serviços
