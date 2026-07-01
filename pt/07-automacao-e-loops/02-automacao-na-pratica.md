# Automacao na pratica — passo a passo

> Defina a condicao de saida primeiro. Depois escolha loop local, Cloud Agent ou Automacao.

---

## Quando montar um loop

Faca isso quando:

- Voce ja corrigiu a **mesma classe de falha** mais de uma vez (CI, lint, testes instaveis)
- O processo tem um **sinal claro de "pronto"** (checks verdes, checklist completa)
- Quer que o agente **verifique o proprio trabalho** antes de pedir seu review
- O time precisa do **mesmo loop** em todo PR ou toda semana

---

## Passo a passo: loop local de correcao de CI

### 1. Defina a condicao de saida em uma linha

**Exemplo:** "Todos os checks do GitHub Actions neste PR passam."

Essa linha vira a secao **Condicao de saida** da skill e o que voce repete no prompt.

---

### 2. Crie a skill de loop

Crie `.cursor/skills/loop-on-ci/SKILL.md` (ou o nome do seu projeto para a mesma ideia):

```markdown
# Loop on CI

> Corrige checks falhando ate ficar verde ou atingir o maximo de iteracoes.

## Trigger

- CI falhou, checks vermelhos, "corrigir CI", "deixar verde"

## Passos da iteracao

### 1. Listar falhas

Rodar ou ler saida de CI. Um bullet por check falhando com o trecho do erro.

### 2. Corrigir a falha de maior sinal

Menor mudanca que ataca a causa raiz. Uma categoria por iteracao quando possivel.

### 3. Verificar e commitar

Re-rodar o check localmente se possivel. Commit com mensagem citando o nome do check.

## Condicao de saida

- Todos os checks obrigatorios passam, OU
- 5 iteracoes atingidas → parar, listar bloqueios, pedir humano

## Checkpoint humano

- Revisar diff antes de push/merge
```

Commite a skill para o time compartilhar o mesmo loop.

---

### 3. Rode o loop no modo Agent

Abra uma sessao na branch com falha. Contexto minimo: referencia da skill, nome da branch e log de CI se precisar.

**Exemplo de prompt:**

```
Use a skill loop-on-ci em .cursor/skills/loop-on-ci/SKILL.md.

Corrija os checks falhando nesta branch. Apos cada tentativa, mostre:
1. O que falhou
2. O que voce mudou
3. Se os checks passam agora

Pare em 5 iteracoes ou quando todos os checks estiverem verdes.
```

---

### 4. Revise como orquestrador humano

Antes do merge:

- [ ] As correcoes batem com as falhas (sem refactors fora do escopo)?
- [ ] A condicao de saida foi de fato verificada (nao assumida)?
- [ ] Os commits sao legiveis para quem revisa?

---

## Passo a passo: loop review-and-ship

Use quando o codigo esta quase pronto mas voce quer que o agente **itere na qualidade** antes de abrir ou atualizar o PR.

### 1. Escreva os criterios de envio

Exemplo de checklist:

- Testes passam localmente
- Lint limpo
- Sem logs de debug
- Descricao do PR preenchida
- Escopo bate com o ticket

Coloque o checklist em `.cursor/skills/review-and-ship/SKILL.md`.

### 2. Rode com instrucao explicita de loop

```
Use a skill review-and-ship. Percorra o checklist.
Para cada item que falhar, corrija e verifique de novo.
Pare quando todos passarem ou apos 3 passadas completas — depois resuma o que faltou.
```

### 3. Opcional: sub-agente para exploracao

Se as falhas exigem busca ampla no repo (ex.: "achar todos os call sites"), use um **sub-agente readonly** primeiro e passe o resumo ao agente principal para editar. Mesmo padrao de [02 - Orquestrando na pratica](../06-orquestracao-de-agentes/02-orquestrando-na-pratica.md).

---

## Passo a passo: Cloud Agent para tarefa longa

### 1. Escreva um brief completo

Cloud Agents funcionam melhor com tarefa **autocontida**:

- O que construir ou corrigir
- Onde no repo (caminhos, padroes)
- Como verificar (testes, comandos)
- O que esta fora do escopo

Anexe **regras** e **skills** no repo para o ambiente na nuvem carregar.

### 2. Inicie um Cloud Agent

- No Cursor: selecione **Cloud** ao lado do input do agente
- Ou: [cursor.com/agents](https://cursor.com/agents)

Cole o brief. Mencione skills por caminho se a tarefa combinar com um playbook.

### 3. Revise artefatos antes do merge

Quando o agente terminar, confira:

- Diff e descricao do PR
- Logs, screenshots ou videos anexados
- Se os passos de verificacao rodaram de fato

Assuma o desktop remoto se precisar clicar na UI.

---

## Passo a passo: criar uma Automacao

### 1. Escolha o trigger

| Objetivo                     | Trigger a considerar             |
| ---------------------------- | -------------------------------- |
| Revisar todo PR novo         | GitHub/GitLab: PR aberto         |
| Reagir a falhas de CI        | GitHub/GitLab: CI concluido      |
| Tarefa semanal               | Agendado (cron)                  |
| Triagem no chat              | Slack: Nova mensagem             |
| Integracao customizada       | Webhook                          |

Crie em [cursor.com/automations/new](https://cursor.com/automations/new) ou na **Agents Window**.

### 2. Escreva o prompt da automacao

Seja especifico sobre:

- O que checar, mudar ou produzir
- Quais ferramentas estao habilitadas (comentar no PR, Slack, MCP)
- Quando **agir** vs. **nao fazer nada** (barra de qualidade)

**Exemplo (automacao de review de PR):**

```
Quando um PR nao-rascunho abrir neste repo:

1. Leia o diff e a issue vinculada se houver
2. Verifique: testes para logica nova, sem secrets, segue .cursor/rules/
3. Se houver problemas: comentarios inline no PR com arquivo/linha
4. Se problema critico de seguranca: solicitar review de @security-team
5. Se estiver ok: comentario curto de aprovacao resumindo o que foi revisado

Nao fazer merge. Nao dar push a menos que o autor do PR pediu correcoes.
```

### 3. Escolha acesso ao repositorio

- **Sem repositorio** — so Slack/MCP/webhook; sem editar codigo
- **Um repositorio** — revisar ou corrigir um codebase
- **Ambiente multi-repo** — tarefas que cruzam varios codebases

### 4. Ative e monitore

- Rode manualmente uma vez se a UI permitir teste
- Confira as primeiras execucoes por ruido (falsos positivos)
- Aperte o prompt quando o agente agir demais ou de menos

Explore o [marketplace de Automacoes](https://cursor.com/marketplace/automations) para templates (review de PR, digest Slack, scan de seguranca).

---

## Organizando skills de loop no repo

Prefira **loops pequenos e nomeados** em vez de uma skill "conserta tudo":

```
.cursor/skills/
├── loop-on-ci/           ← checks verdes
├── review-and-ship/      ← passada de qualidade pre-PR
├── fix-flaky-test/       ← um arquivo de teste, iterar
└── create-vue-component/ ← one-shot (nao e loop)
```

**Componiveis:** skills de orquestracao chamam skills one-shot dentro de um loop (ex.: review-and-ship roda skill de lint a cada iteracao).

---

## Testando um loop

1. **Comece com falha conhecida** — CI vermelho ou teste falhando em branch descartavel
2. **Limite iteracoes baixo** (3–5) na primeira execucao
3. **Exija evidencia** — agente deve colar saida de comando ou status do check
4. **Compare** — rode a mesma tarefa sem a skill; note deriva e prompts extras
5. **Atualize a skill** quando descobrir passo faltando ou "pronto" falso

---

## Exemplo completo (historia)

**Cenario:** PR aberto com dois checks falhando — lint e testes unitarios.

1. **Loop local (maquina do autor):** desenvolvedor roda skill `loop-on-ci` no modo Agent; agente corrige lint, depois testes, em iteracoes separadas; da push quando verde.
2. **Automacao (time):** automacao "PR aberto" faz review readonly; comenta se codigo novo nao tem testes.
3. **Cloud Agent (opcional):** para correcao grande o autor delega "corrigir todas as falhas de teste na branch `feature/x`" ao Cloud; revisa o PR quando notificado.

As mesmas **regras** valem em todo lugar. **Skills** definem como cada loop se comporta. **Voce** faz o merge.

---

## Resumo

1. Escreva a **condicao de saida** antes do prompt.
2. Capture loops locais repetiveis em **`.cursor/skills/`** com maximo de iteracoes e verificacao.
3. Use **Cloud Agents** para tarefas longas e bem delimitadas que da para revisar depois.
4. Use **Automacoes** para loops repetitivos do time disparados por evento.
5. Mantenha sempre um **checkpoint humano** antes de merge ou producao.

---

> **Proximo:** [07 - Exercicio: automacao e loops](../05-guias-praticos/07-exercicio-automacao-loops.md) — montar e rodar uma skill de loop no projeto · [06 - Orquestracao](../06-orquestracao-de-agentes/01-o-que-e-orquestracao.md) — skills, agentes e sub-agentes
