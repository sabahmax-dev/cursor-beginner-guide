# `/poteto-mode` — uma orquestracao pronta

> **pstack** e um plugin do Cursor. **`/poteto-mode`** e a porta de entrada: voce diz o objetivo, ele escolhe um playbook e chama outras skills (`/how`, `/why`, `/architect` e o resto) quando o passo precisa.
> O nome e **poteto** (handle da autora), nao potato.

**Pre-requisitos:** [01 — Visao geral](./01-o-que-e-orquestracao.md) e [04 — Skills](../04-skills/01-o-que-sao-skills.md). Instale o plugin antes de colar os prompts abaixo.

---

## Por que esta pagina existe

A Parte 2 ensinou voce a desenhar orquestracao: skill para o processo, agente principal para editar, sub-agentes para trabalho isolado.

**pstack** e a versao empacotada disso. Voce nao precisa inventar o playbook de "corrija o bug com evidencia" ou "explique como este subsistema funciona." Instala, digita `/poteto-mode` e um criterio de pronto.

Use quando quiser um **metodo padrao** entre conversas. Continue escrevendo skills suas quando o processo e **do projeto** (convencoes deste repo, checklist de deploy, fluxo que o pstack nao conhece).

---

## O que e o pstack

Quatro camadas em cima do Cursor:

| Camada | Papel |
| ------ | ----- |
| **`/poteto-mode`** | Despachante. Casa o pedido com um playbook e fica ativo nos turnos seguintes ate voce sair ou comecar uma tarefa nova. |
| **Playbooks** | Fluxos nomeados (investigacao, bug fix, feature, acompanhar PR, rodadas overnight, e outros). Em geral voce nao nomeia o playbook. |
| **Skills roteadas** | `/how`, `/why`, `/architect`, `/arena`, `/swarm`, `/interrogate`, `/tdd`, `/blast-radius` e outras. O mode chama. Voce tambem pode chamar uma direto. |
| **Principios** | Vieses curtos: menor mudanca, provar no artefato real, gravar licao repetida como check em vez de mais texto. |

A linha da autora: ir fundo primeiro, escrever menos, verificar na coisa real. Throughput sem essa prova nao e o objetivo.

Instale no [marketplace do Cursor](https://cursor.com/marketplace/cursor/pstack). Fonte: [cursor/plugins `pstack/`](https://github.com/cursor/plugins/tree/main/pstack).

---

## O que o `/poteto-mode` faz com o seu prompt

```
Seu prompt
    → poteto-mode le os principios
    → escolhe um playbook
    → copia os passos para a lista de todos
    → chama /how, /why, /architect, … so quando o passo precisa
    → verifica e reporta
```

Voce entrega **intencao** e como saber que acabou. A skill carrega a **ordem**.

Um passo pulado deve ficar na lista com `skip: <motivo>`. Se o agente omitir passos em silencio, peca para seguir o playbook.

---

## O loop (a parte que vale decorar)

Para trabalho que pode mudar o sistema:

**Entender → desenhar → construir em unidades pequenas → provar no artefato real → gravar o que aprendeu.**

| Momento | Skill tipica | Papel |
| ------- | ------------ | ----- |
| Voce nao tem modelo mental | `/how` | Como o subsistema funciona. Exploradores em paralelo, depois uma explicacao. |
| Voce pode mudar um formato com historia | `/why` | Por que foi feito assim, a partir de git, tickets, docs, chat, se esses MCPs existem. |
| A mudanca cruza fronteira de modulo | `/architect` | Tipos e mapa de modulos antes de preencher corpos. |
| Varios desenhos validos | `/arena` | Mesmo prompt, varios modelos, escolher e enxertar. |
| Um diff em que voce nao confia | `/interrogate` | Varios modelos revisam. O lead julga. Nada aplica sozinho. |
| Uma mudanca que parece pequena | `/blast-radius` | Um fato de seguranca, provado rodando codigo. |
| Pronto | Testes e checks do *seu* projeto | "Compilou" nao e pronto. |

Perguntas so de leitura pulam desenho e build. Esse e o playbook de **investigacao**: `/how`, e `/why` quando a pergunta e motivacao.

---

## Primeiro setup

1. Adicione o plugin no Cursor (`/add-plugin pstack`, ou instale **pstack** no marketplace).
2. Rode **`/setup-pstack`** uma vez. As skills assumem nomes de modelo que voce pode nao ter. Mapeie cada papel para um modelo que voce consegue spawnar, ou use `inherit-parent` / `auto`.
3. Instale tambem o **cursor-team-kit** se quiser `/deslop` e as skills de controle de CLI/UI que o poteto-mode cita e nao empacota.

Sem o passo 2, a revisao multi-modelo cai num modelo so, e isso e a maior parte do produto.

---

## Como promptar

Diga o objetivo e como voce vai saber que funcionou. Nao liste cada skill.

**Bug com criterio:**

```text
/poteto-mode users get two notifications after a retry. repro first, then fix and verify.
```

**Entender, sem editar:**

```text
/poteto-mode new task. figure out why the cache entry survives logout. don't change any code yet.
```

`new task` manda o mode sticky recasar o pedido em vez de continuar o playbook anterior.

**Voce vai sair:**

```text
/poteto-mode i'm stepping away. keep going until the migration check reports zero old callers. log your decisions.
```

**Armadilha:** nao escreva "use /how, depois /architect, depois /arena." O playbook ja sequencia isso. Nomeie uma skill so quando quiser **sobrescrever** um passo.

Quando o chat ja tem a tarefa, `/poteto-mode do it` ou `keep going until done` basta. O playbook segura a estrutura.

---

## `/how` e `/why` (quando chamar voce mesmo)

Use **`/how`** quando quiser um walkthrough para guardar: visao geral, conceitos, como funciona, onde moram os arquivos, pegadinhas.

Use **`/why`** quando for mudar uma decisao com historia (numero magico, excecao de import, feature flag). Ele busca git e os MCPs que voce tem (Linear, docs, Slack, Sentry). Fonte vazia faz parte da resposta. E caro. Nao rode em todo arquivo.

Use **`/teach`** quando quiser how + why numa explicacao so, diagrama a diagrama.

No dia a dia de implementacao, fique no `/poteto-mode` e deixe ele rotear.

---

## Como isso encaixa no resto do guia

| Ideia do guia | Equivalente no pstack |
| ------------- | --------------------- |
| Regra (sempre ligada) | Principios + regra de modelos do `/setup-pstack` |
| Skill (sob demanda) | Playbooks e skills roteadas |
| Agente principal | O chat em que voce digitou `/poteto-mode` |
| Sub-agente | Exploradores, investigadores, runners de arena, revisores |

O pstack **nao** substitui skills do projeto. Se o repo tem skills proprias ou um comando de check (`make test`, lint, suite de features), diga isso no prompt. O poteto-mode nao conhece isso se voce nao apontar.

---

## Quando nao usar

- Pergunta casual ("o que essa funcao retorna?"). Fique no Ask mode.
- Rename de uma linha. Menor mudanca; pule o playbook de feature inteiro.
- `/why` por habito. Use antes de reescrever uma fronteira, nao antes de cada edit.
- Autopilot overnight ate o CI e os secrets funcionarem de verdade naquele ambiente.

Saia do mode sticky dizendo isso. Em turnos casuais ele deve ficar fora do caminho.

---

## Experimente uma vez

1. Abra um repo real no Cursor.
2. Confirme que o pstack esta ligado e que `/setup-pstack` ja rodou.
3. Cole:

```text
/poteto-mode new task. explain how the main HTTP entrypoint starts, without changing any files.
```

4. Olhe a lista de todos. Voce deve ver passos de investigacao, nao um refactor imediato.
5. Se depois quiser uma mudanca, comece com `/poteto-mode new task.` e um estado de pronto **verificavel** (teste, comando, screenshot).

---

## Resumo

| Ideia | Lembrete |
| ----- | -------- |
| **pstack** | Plugin: playbooks + skills + principios |
| **`/poteto-mode`** | Entrada padrao. Objetivo + check. Ele roteia. |
| **`/how`** | Como o codigo funciona |
| **`/why`** | Por que foi construido assim (evidencia citada) |
| **Suas skills** | Continuam obrigatorias para o processo *deste* projeto |

---

> **Ver tambem:** [01 — Visao geral](./01-o-que-e-orquestracao.md) · [02 — Orquestrando na pratica](./02-orquestrando-na-pratica.md) · [pstack em cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack)
