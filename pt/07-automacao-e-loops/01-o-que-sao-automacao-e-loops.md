# Automacao e loops — quando o agente continua sozinho

> **Skill** documenta o processo. **Loop** repete ate uma condicao ser atendida.
> Automacao e delegar a repeticao ao agente — com trilhos de seguranca.

---

## O que sao loops de automacao?

Um **loop de automacao** e um fluxo que o agente executa **mais de uma vez** ate algo mensuravel ser verdade: testes passam, CI verde, checklist de review completa, ou PR pronto para merge.

Isso e diferente do ciclo **Pesquisar → Planejar → Implementar** em [01 - Fundamentos](../01-fundamentos/03-workflow.md). Aquele ciclo e **dirigido por voce** — voce troca de modo de proposito. Loops de automacao sao **dirigidos pelo agente** — voce define a condicao de saida e deixa o agente iterar.

Pense em tres camadas:

| Camada            | Quem dirige      | Escopo tipico                              |
| ----------------- | ---------------- | ------------------------------------------ |
| **Loop local**    | Voce + agente    | Uma sessao: corrigir → testar → corrigir   |
| **Cloud Agent**   | Agente           | Tarefa longa na nuvem; voce revisa depois  |
| **Automacao**     | Evento/cron      | Roda sem voce iniciar cada vez             |

Quando essas camadas se alinham com **skills**, **regras** e **orquestracao**, voce para de redigitar o mesmo prompt de "tenta de novo ate funcionar".

---

## Como isso se relaciona com o que voce ja sabe

| Conceito          | Papel no loop                                               |
| ----------------- | ----------------------------------------------------------- |
| **Regras**        | Limites que nao mudam (estilo, seguranca, stack)            |
| **Skill**         | Define os passos de cada iteracao e o checklist de validacao|
| **Orquestracao**  | Quem explora, quem edita, quem resume entre iteracoes       |
| **Loop**          | Repete a skill ate a condicao de saida ser atendida          |

```
Regra (sempre) → Skill (cada iteracao) → Agente (age)
                      ↑
          Sub-agente (opcional: explorar falhas / rodar checks)
                      ↓
            Condicao de saida? ──nao──→ loop de novo
                      │
                     sim
                      ↓
            Checkpoint humano (merge, deploy, aprovar)
```

> Skills: [04 - Skills](../04-skills/01-o-que-sao-skills.md) · Orquestracao: [06 - Orquestracao](../06-orquestracao-de-agentes/01-o-que-e-orquestracao.md)

---

## Tres tipos de automacao

### 1. Loops iterativos locais (IDE)

Voce fica no modo Agent e pede ao agente para **continuar** ate uma condicao ser verdadeira. O agente roda comandos, le a saida, corrige codigo e tenta de novo.

**Melhor para:** corrigir testes falhando, deixar lint verde, apertar um PR antes do review.

**Trilhos que voce deve definir:**

- **Condicao de saida** — o que significa "pronto" (ex.: `npm test` retorna 0)
- **Maximo de iteracoes** — parar apos N tentativas e pedir ajuda humana
- **Escopo** — quais arquivos ou checks o agente pode alterar

**Exemplo de prompt:**

```
Use a skill loop-on-ci. Corrija os checks falhando nesta branch.
Pare quando todos os checks de CI passarem ou apos 5 iteracoes — o que vier primeiro.
Mostre o que falhou antes de cada tentativa de correcao.
```

### 2. Cloud Agents (background agents)

**Cloud Agents** rodam em uma VM isolada na nuvem. Planejam, editam, rodam comandos, testam o trabalho e podem abrir um PR — sem seu laptop ficar conectado. Antes eram chamados de Background Agents.

**Melhor para:** features que levam tempo, bugs que voce quer iniciar e revisar depois, trabalho que da para descrever com clareza de antemao.

**Como iniciar:**

- No Cursor, selecione **Cloud** ao lado do input do agente
- Na web ou mobile: [cursor.com/agents](https://cursor.com/agents)
- No Slack, GitHub ou Linear: mencione `@Cursor`

Cloud Agents produzem **artefatos** (logs, screenshots, videos) para voce validar sem fazer checkout da branch.

> Detalhes: [Cloud Agents — documentacao Cursor](https://cursor.com/docs/cloud-agent)

### 3. Automacoes (sempre ligadas)

**Automacoes** rodam Cloud Agents em **agenda** ou quando um **evento** dispara — PR aberto, CI concluido, mensagem no Slack, issue no Linear, webhook e mais.

**Melhor para:** review de PR a cada abertura, triagem de bugs no Slack, scans de seguranca apos CI, tarefas de limpeza agendadas.

**Como funcionam:**

1. Escolha um trigger (agenda, PR aberto no GitHub, CI concluido, etc.)
2. Escreva um prompt dizendo o que o agente deve fazer
3. Selecione ferramentas (comentar no PR, Slack, MCP, memorias)
4. Anexe zero, um ou varios repositorios
5. Salve e ative

Automacoes nao exigem que voce esteja no teclado. Cada execucao e cobrada conforme o preco de API do modelo escolhido.

> Detalhes: [Automacoes — documentacao Cursor](https://cursor.com/help/ai-features/automations.md) · Templates: [marketplace de Automacoes](https://cursor.com/marketplace/automations)

---

## Quando usar loops

| Situacao                              | Abordagem tipica                            |
| ------------------------------------- | ------------------------------------------- |
| Corrigir CI ate ficar verde na branch | **Loop local** + skill de loop              |
| Revisar PR e enviar quando pronto     | **Loop local** + skill review-and-ship      |
| Implementar feature enquanto ausente  | **Cloud Agent**                             |
| Revisar todo PR novo automaticamente  | **Automacao** (trigger PR aberto)           |
| Triar bugs vindos do Slack            | **Automacao** (trigger mensagem Slack)      |
| Limpeza semanal de dependencias       | **Automacao** (trigger agendado)            |

**Regra pratica:** se voce ja disse "tenta de novo ate funcionar" mais de duas vezes na mesma tarefa, capture isso como **skill de loop** ou **automacao**.

---

## Anatomia de uma skill de loop

Skills de loop vivem em `.cursor/skills/` como qualquer outra skill, mas acrescentam **iteracao** e **condicoes de parada**:

```markdown
# Loop on CI

> Corrige checks falhando ate ficar verde ou atingir o maximo de iteracoes.
> Use quando CI falhar na sua branch ou PR.

## Trigger

- "Corrigir CI", "deixar checks verdes", "loop ate CI passar"

## Precondicoes

- [ ] Na branch correta
- [ ] Consegue rodar os mesmos checks localmente (ou via logs de CI)

## Passos da iteracao

### 1. Ler falhas

- Rodar checks ou ler saida de CI
- Listar cada check falhando com a mensagem de erro

### 2. Corrigir uma categoria por vez

- Escolher a falha de maior sinal primeiro
- Fazer a menor correcao que ataca a causa raiz

### 3. Verificar

- Re-rodar o check falhando localmente se possivel
- Commit com mensagem descritiva

## Condicao de saida

- [ ] Todos os checks obrigatorios passam
- OU maximo de iteracoes (padrao: 5) atingido → parar e resumir bloqueios

## Checkpoint humano

- [ ] Revisar o diff antes do merge
- [ ] Confirmar que nao entrou mudanca fora do escopo
```

---

## Loop local vs. Cloud Agent vs. Automacao

| Aspecto              | Loop local               | Cloud Agent               | Automacao                      |
| -------------------- | ------------------------ | ------------------------- | ------------------------------ |
| **Voce inicia**      | Sim, cada sessao         | Sim, uma vez por tarefa   | Nao — o trigger inicia         |
| **Roda ausente**     | Nao (ligado a sessao)    | Sim                       | Sim                            |
| **Melhor iteracao**  | Feedback apertado        | Tarefas longas ponta a ponta | Tarefas repetitivas do time |
| **Modelo de custo**  | Sessao no IDE            | Por execucao de Cloud Agent | Por execucao da automacao  |
| **Review humano**    | Entre iteracoes          | Antes do merge            | Antes do merge (ou comentario auto) |

---

## Erros comuns

| Erro                             | Efeito                              | Ajuste                                        |
| -------------------------------- | ----------------------------------- | --------------------------------------------- |
| Sem condicao de saida            | Agente loop infinito, queima contexto | Definir "pronto" + max iteracoes na skill   |
| Sem passo de verificacao         | Agente declara sucesso cedo demais  | Exigir saida de comando ou status de CI       |
| Automatizar trabalho vago        | PRs ruidosos, correcoes erradas     | Comecar com loops locais; automatizar quando estavel |
| Pular checkpoint humano          | Codigo ruim mergeado                | Sempre revisar antes do merge                 |
| Um mega-loop para tudo           | Dificil debugar falhas              | Skills de loop pequenas por cenario (CI, testes, lint) |

---

## Resumo

| Conceito              | Lembrete em uma frase                                        |
| --------------------- | ------------------------------------------------------------ |
| **Loop local**        | Agente repete na sessao ate condicao de saida ou limite       |
| **Skill de loop**     | Documenta passos, verificacao e regras de parada             |
| **Cloud Agent**       | Agente longo na nuvem; devolve artefatos                     |
| **Automacao**         | Cloud Agent em agenda ou evento — sem start manual             |
| **Checkpoint humano** | Voce aprova antes de merge, deploy ou mudanca em producao    |

---

> **Proximo:** [02 - Automacao na pratica](./02-automacao-na-pratica.md) — passo a passo: loop de CI, review-and-ship, Cloud Agents e Automacoes
