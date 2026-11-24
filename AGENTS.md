# Guia para agentes — `actions` (FIAP X)

Reusable workflows de CI/CD da org [`fiap-x-vcosta`](https://github.com/fiap-x-vcosta). Callers nos demais repositórios usam `fiap-x-vcosta/actions/.github/workflows/….yml@main`.

## Antes de mudar código

1. Manter callers finos: o miolo mora aqui; não copiar jobs de volta para os serviços.
2. Não adicionar `tf-apply` / `tf-destroy` consumíveis pelo `infra-bootstrap`.
3. Não inventar secrets no Git; org vars/secrets ficam no GitHub; app config no Secret Manager (JSON por serviço).
4. **Git:** nunca commit/push direto em `main`; a IA abre PRs e **nunca** faz merge (ver [`.cursor/rules/git-workflow.mdc`](.cursor/rules/git-workflow.mdc)).
5. Naming: proibida a string que combina o produto com “videos” em camel/kebab/snake. Preferir `fiap-x`, `processamento`, `midia`, `frames`.
6. README autocontido na org — sem links para material fora do Git da org.

## Responsabilidade

| Peça | Papel |
|------|--------|
| `ci-dotnet` | Lint, unit (gate 80%), integração, SonarCloud |
| `ci-terraform` | fmt / validate / plan opcional em PR |
| `build-push` | Imagem no Artifact Registry |
| `deploy` | Org secrets + JSON SM → Secret k8s; manifests `k8s/`; rollout; smoke |
| `tf-apply` / `tf-destroy` | Apply/destroy manuais via caller `workflow_dispatch` |

## Regras canônicas (resumo)

- Push nesta `main` atualiza todos os `@main`
- Este repo está fora de `ci_repositories` (sem WIF próprio)
- `building-blocks` não consome estes reusables
- Extras do `infra-k8s` (Gateway / cleanup) ficam no caller
- Deploy: org secrets só `DD_API_KEY` e `SERVICE_AUTH_KEY`; demais app secrets via JSON no Secret Manager

## Comandos

Não há build de aplicação neste repositório. Validação = revisão dos YAML + CI verde no primeiro caller após o merge.
