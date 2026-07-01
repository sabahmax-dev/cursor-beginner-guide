# Exercicio 7: Automacao e loops na pratica

> **Objetivo:** Criar e rodar uma skill de loop que itera ate uma condicao de saida clara.
> **Modo:** Agent Mode
> **Tempo estimado:** 25 minutos
> **Pre-requisito:** [04 - Skills](../04-skills/01-o-que-sao-skills.md) e [07 - Automacao e loops](../07-automacao-e-loops/01-o-que-sao-automacao-e-loops.md)

---

## Cenario

Voce vai criar uma **skill de loop** (nao one-shot), rodar em um check real do projeto e observar como condicao de saida e maximo de iteracoes mantem o agente no trilho.

---

## Exercicio 7.1 — Criar uma skill de loop

Vamos criar uma skill para corrigir checks falhando ate ficar verde ou ate o limite de iteracoes.

### Prompt para copiar e colar:

```
Crie uma skill de loop em .cursor/skills/loop-on-ci/SKILL.md

A skill deve documentar um processo iterativo para corrigir CI ou checks locais
falhando na branch atual. Baseie os comandos de verificacao no que este projeto
realmente usa (test runner, linter, typecheck).

A skill deve incluir:
1. Trigger — quando usar esta skill
2. Precondicoes — branch, capacidade de rodar checks
3. Passos da iteracao — ler falhas, corrigir uma categoria, verificar, commitar
4. Condicao de saida — todos os checks passam OU maximo 5 iteracoes
5. Checkpoint humano — revisar diff antes do merge

Inclua comandos concretos deste repositorio (leia package.json, Makefile ou config de CI).
```

### O que observar:

- O agente deve **inspecionar como os checks rodam** no projeto antes de escrever a skill
- A skill deve exigir **evidencia** (saida de comando), nao so "corrigi"
- Maximo de iteracoes deve estar explicito

---

## Exercicio 7.2 — Rodar o loop (simulado ou real)

Se tiver check falhando, use. Se tudo estiver verde, introduza uma falha pequena e segura (ex.: import nao usado que quebra lint) em branch descartavel.

### Prompt para copiar e colar:

```
Use a skill loop-on-ci em .cursor/skills/loop-on-ci/SKILL.md.

Corrija os checks falhando nesta branch. Apos cada iteracao mostre:
1. O que falhou (com trecho da saida do comando)
2. O que voce mudou
3. Se os checks passam agora

Pare em 5 iteracoes ou quando todos os checks obrigatorios estiverem verdes.
```

### O que observar:

- O agente carrega a skill e segue os **passos de iteracao em ordem**
- Cada iteracao termina com tentativa de **verificacao**
- O agente **para** quando verde ou no limite — nao faz loop infinito

### Verifique:

- [ ] A skill foi carregada?
- [ ] Falhas listadas com evidencia?
- [ ] Correcoes no escopo (sem refactors aleatorios)?
- [ ] Loop parou na condicao de saida ou no maximo de iteracoes?

---

## Exercicio 7.3 — Comparar: correcao one-shot vs. loop

Rode a mesma falha uma vez **sem** referenciar a skill de loop — so "corrija o check falhando".

| Aspecto             | Sem skill de loop         | Com skill de loop            |
| ------------------- | ------------------------- | ---------------------------- |
| Tamanho do prompt   | Maior a cada vez          | Referencia curta a skill     |
| Verificacao         | Muitas vezes pulada       | Obrigatoria a cada iteracao  |
| Condicao de parada  | Vaga                      | Limite explicito + checks verdes |
| Repetibilidade      | Deriva entre sessoes      | Mesmos passos sempre         |

---

## Exercicio 7.4 — Opcional: esbocar uma Automacao

Nao precisa ativar billing ou automacoes em producao neste exercicio. Escreva o prompt que **usaria** se uma automacao "PR aberto" revisasse todo PR deste repo.

### Prompt para copiar e colar:

```
Rascunhe um prompt de automacao (nao crie em cursor.com) para:
Trigger: PR aberto no GitHub neste repo.

A automacao deve:
- Ler o diff
- Verificar contra nossas .cursor/rules/ se existirem
- Comentar no PR se faltam testes para logica nova
- Nao fazer nada se o PR estiver ok

Saida: apenas o texto do prompt, adequado para cursor.com/automations/new.
```

### O que observar:

- Claro **quando agir vs. nao fazer nada**
- Sem merge ou push a menos que explicitamente permitido
- Referencia padroes reais do projeto

---

## O que aprendemos

1. **Skills de loop acrescentam iteracao e regras de parada** — nao so passos ordenados
2. **Condicao de saida deve ser verificavel** — saida de comando ou status de CI
3. **Maximo de iteracoes protege contexto e custo** — sempre defina limite no primeiro uso
4. **Loops locais compoem com Cloud Agents e Automacoes** — mesmas skills, triggers diferentes
5. **Checkpoint humano continua obrigatorio** — loops automatizam repeticao, nao aprovacao

---

## Parabens!

Voce completou todos os exercicios. Agora voce sabe:

- **Pesquisar** com Ask Mode antes de agir
- **Planejar** com Plan Mode antes de implementar
- **Implementar** com Agent Mode de forma controlada
- **Usar regras** para garantir consistencia automatica
- **Criar skills** para automatizar tarefas repetitivas
- **Orquestrar** skills, agente principal e sub-agentes
- **Rodar loops de automacao** com trilhos ate o trabalho estar de fato pronto
- **Gerenciar contexto** para manter a qualidade das respostas

### Proximo nivel

- Adicione uma skill `review-and-ship` para qualidade pre-PR
- Experimente um **Cloud Agent** em tarefa bem delimitada em [cursor.com/agents](https://cursor.com/agents)
- Explore o [marketplace de Automacoes](https://cursor.com/marketplace/automations) para templates do time
- Experimente e itere — e assim que se aprende

> _"Escolha uma ferramenta, pratique muito. E parte arte e parte ciencia — voce precisa de muita repeticao para desenvolver a intuicao do que confiar ao modelo e do que manter para voce."_

---

> **Veja tambem:** [07 - Automacao e loops](../07-automacao-e-loops/01-o-que-sao-automacao-e-loops.md) · [06 - Workflow completo](./06-exercicio-workflow-completo.md)
