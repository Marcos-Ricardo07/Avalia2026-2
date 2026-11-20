# Documentação de Engenharia de Agentes e Harness

## 1. Ambiente de Orquestração

- **Harness Principal:** GitHub Copilot no VS Code, com agentes personalizados em `.github/agents/`.
- **Modelo(s) Base:** Seleção automática do Copilot (Copilot Fusion); nas sessões registradas, o modelo escolhido foi o GPT-6 Luna.
- **Estratégia de Coordenação:** Os agentes trabalham modularmente na pasta do projeto, alinhados pelos contratos da API, regras das skills e resultados de testes.

## 2. Agentes e Responsabilidades

### Agente 1: Backend Sênior
- **Papel:** Implementa a API serverless do Quadro Kanban com Cloudflare Pages Functions e D1. É responsável pela autenticação Discord, validações no servidor e isolamento dos cartões por usuário.
- **Arquivo de definição:** `.github/agents/backend-senior.agent.md`
- **Pastas/arquivos que pode alterar:** `functions/`, `migrations/` e `wrangler.jsonc`.
- **Skills que lê:** `skills/serverless-d1-discord/SKILL.md`.
- **Entregas realizadas:** (a preencher ao longo do projeto)

### Agente 2: Frontend Sênior
- **Papel:** Implementa a interface responsiva e acessível em HTML, CSS e JavaScript puro. Integra a interface com a API e oferece arrastar e soltar nativo e controles alternativos para mover cartões.
- **Arquivo de definição:** `.github/agents/frontend-senior.agent.md`
- **Pastas/arquivos que pode alterar:** `public/`.
- **Skills que lê:** `skills/vanilla-dom-ui/SKILL.md` e `skills/kanban-drag-drop/SKILL.md`.
- **Entregas realizadas:** (a preencher ao longo do projeto)

### Agente 3: Testador e Validador (Teste e Validação)
- **Papel:** Testa independentemente a API, a segurança, a interface e os requisitos da rubrica, registrando evidências reproduzíveis. Não corrige código de produção; relata defeitos aos agentes responsáveis.
- **Arquivo de definição:** `.github/agents/testador-validador.agent.md`
- **Pastas/arquivos que pode alterar:** `tests/` e `docs/`.
- **Skills que lê:** `skills/teste-validacao/SKILL.md`, `skills/serverless-d1-discord/SKILL.md`, `skills/vanilla-dom-ui/SKILL.md` e `skills/kanban-drag-drop/SKILL.md`.
- **Entregas realizadas:** (a preencher ao longo do projeto)

## 3. Fluxo de trabalho

O Backend implementa a API; o Testador testa e relata os resultados, e o Backend corrige os defeitos encontrados. Em seguida, o Frontend implementa a interface e o Testador a valida. Os resultados ficam registrados em `docs/relatorio-testes.md`.

## 4. Casos de teste e resultados

Os casos executados, resultados, defeitos e limitações de teste estão em [`docs/relatorio-testes.md`](./docs/relatorio-testes.md).
