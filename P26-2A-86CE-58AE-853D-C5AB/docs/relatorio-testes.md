# Relatório de Testes

## Rodada 1 — API e auditoria estrutural

- **Data/hora:** 2026-11-20 18:37:46 -03:00
- **Escopo:** primeira rodada contra a API implementada pelo Backend Sênior; testes executados localmente no Wrangler e no D1 local.
- **Segredos/dados:** foram usados apenas usuários e tokens fictícios. O servidor foi iniciado com `DISCORD_CLIENT_ID=test-client-id` para testar o redirecionamento sem ler credenciais locais nem contatar o Discord durante o callback.

### Comandos executados

```powershell
npx.cmd wrangler d1 migrations apply DB --local
npx.cmd wrangler d1 execute DB --local --file tests/seed.sql
npx.cmd wrangler pages dev --binding DISCORD_CLIENT_ID=test-client-id
curl.exe -sS -o NUL -w "%{http_code}" http://localhost:8788/api/cartoes
node --test tests/api.test.mjs
cd ..; node validar-local.mjs P26-2A-86CE-58AE-853D-C5AB
git status --short --untracked-files=all
git ls-files --error-unmatch -- P26-2A-86CE-58AE-853D-C5AB/.dev.vars
```

As migrações locais já estavam aplicadas (“No migrations to apply”). A carga de teste foi aplicada ao D1 local. O comando de `git ls-files` não encontrou `.dev.vars` rastreado. O validador fez somente GET ao endereço público indicado no README; não foram enviados métodos de escrita ao deploy nem usados comandos `--remote`.

### Casos da API

| Caso | Resultado esperado | Resultado obtido | Status |
|---|---|---|---|
| Sem cookie em `GET /api/cartoes` e `GET /api/sessao` | Ambos `401` | Ambos `401` | Aprovado |
| Token de sessão inexistente | `401` | `401` | Aprovado |
| Cookie legado `discord_session` com JSON base64 | `401`, sem acesso | `401` nas duas rotas protegidas | Aprovado |
| Isolamento entre usuários | B não lista o cartão de A; PATCH e DELETE de B dão `404`; cartão de A permanece intacto | Lista de B não continha o cartão; PATCH/DELETE `404`; cartão de A intacto | Aprovado |
| Validações: coluna/prioridade inválidas; título vazio ou >120; descrição >1000; posição não numérica; ID não numérico; JSON inválido; PATCH sem campo permitido | `400` | Todos os nove cenários retornaram `400` | Aprovado |
| `user_id` enviado pelo cliente | Proprietário continua sendo o usuário da sessão | O cartão permaneceu com `user_id=111` após POST e PATCH forjados | Aprovado |
| Persistência e ordenação | PATCH persiste coluna/posição; GET ordena por coluna e posição | GET confirmou os valores e a ordem esperada | Aprovado |
| Exclusão própria | DELETE `204`; cartão ausente no GET seguinte | `204`; cartão ausente | Aprovado |
| SQL Injection e texto HTML | Strings retornam como texto e tabela segue disponível | Ambos os títulos foram salvos/retornados; GET seguiu funcionando | Aprovado |
| Erros seguros | Sem stack trace, SQL, caminho interno ou segredo | Corpos dos erros não continham esses dados | Aprovado |
| Logout isolado | Sessão própria retorna `204`; token passa a receber `401`; outras sessões continuam válidas | `204`, depois `401`; `teste-a` permaneceu autenticada | Aprovado |
| OAuth2: início e callback com state divergente | Redirecionamento com `state`, `scope=identify` e cookie protegido; callback divergente recusado | `302`, URL/cookie com atributos esperados; callback `400` | Aprovado |

**Resumo da suíte:** 12 casos aprovados, 0 reprovados, 0 ignorados (`node --test tests/api.test.mjs`).

### Auditoria da entrega

O comando `node validar-local.mjs P26-2A-86CE-58AE-853D-C5AB`, executado na pasta acima do projeto, marcou **9/9 itens como OK**:

1. LICENSE MIT.
2. Chave no README.
3. Item de chave no HTML marcado como OK pelo validador.
4. AGENTS documenta pelo menos três agentes.
5. Agente de teste e validação documentado.
6. Skills presentes.
7. Estrutura HTML e ausência das dependências/frameworks proibidos pelo validador.
8. Cloudflare Functions, D1 e OAuth Discord presentes.
9. Deploy online na Cloudflare.

Verificações adicionais: o primeiro endereço `.pages.dev` do README é `https://projeto-p26-2a-86ce-58ae-853d-c5ab.pages.dev` e a chave consta no README; o LICENSE contém as cláusulas da licença MIT; AGENTS informa o harness, contém os três títulos requeridos e tem 2.632 caracteres; há quatro arquivos `SKILL.md`; as alterações reportadas pelo `git status` estão dentro da pasta do projeto; `.dev.vars` não está rastreado pelo Git.

**Limite do validador:** o item 3 confirma a presença da chave no HTML recebido, mas não valida semanticamente se ela está dentro de um elemento `<footer>`. Essa inspeção detalhada foi adiada com as demais verificações do frontend.

### Defeitos encontrados

Nenhum defeito confirmado nos casos de API executados; não há arquivo ou agente responsável por correção nesta rodada.

### Limitações e roteiro manual para a próxima rodada

As verificações comportamentais do frontend foram adiadas, conforme o escopo desta rodada e porque a interface ainda será refeita. O fluxo real de autorização/troca de token com o Discord também não foi executado; apenas o redirecionamento de início e a rejeição de state divergente foram testados. Não foram feitas alterações de dados no site publicado.

Após a implementação da interface, executar localmente, com o ambiente Discord de teste configurado sem expor credenciais:

1. Abrir `http://localhost:8788` e iniciar login pelo Discord; confirmar que o perfil autenticado aparece.
2. Criar um cartão e confirmar título, descrição e prioridade.
3. Arrastar o cartão para outra coluna e reordená-lo dentro da coluna.
4. Editar o texto e mudar a prioridade; confirmar a persistência após recarregar.
5. Excluir o cartão e confirmar que não reaparece após recarregar.
6. Repetir a inspeção numa largura de celular, verificando uso e legibilidade dos controles.

Não executar operações de escrita no site publicado; para estes passos, usar somente o ambiente local.
