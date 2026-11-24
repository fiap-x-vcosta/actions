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

## Workflows previstos

| Workflow | Uso |
|----------|-----|
| `ci-dotnet` | Lint (InspectCode), unitários (gate 80% line/branch), integração, SonarCloud |
| `ci-terraform` | `fmt` / `validate`; `plan` opcional em PR |
| `build-push` | Build e push da imagem no Artifact Registry |
| `deploy` | Deploy no GKE (manifests em `k8s/`, Secret, rollout, smoke HTTPS) |
| `tf-apply` / `tf-destroy` | Apply/destroy manuais via caller `workflow_dispatch` |

Pin padrão do Terraform: **1.16.1**. Working directory padrão: **`terraform`**. Namespace K8s padrão do deploy: **`fiap-x`**.

Os arquivos em `.github/workflows/` entram em PRs seguintes.

## Deploy — secrets (contrato)

- **Org secrets (GitHub):** `DD_API_KEY`, `SERVICE_AUTH_KEY`.
- **Secret Manager (GCP):** um JSON por serviço (TF do serviço). Chaves literais ASP.NET, por exemplo `ConnectionStrings__DefaultConnection` e, só em `autenticacao`, `Jwt__SigningKeyPem`.

## Fora de escopo

- Apply/destroy do `infra-bootstrap`
- CI próprio do pacote `building-blocks` (não usa estes reusables)
- Código de aplicação, Terraform de nuvem e manifests dos serviços
