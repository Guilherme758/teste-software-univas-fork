# Relatório — Testes unitários com IA generativa

**Aluno(s): Guilherme Henrique**
**Ferramenta de IA utilizada:** Claude Code
**Unidades testadas:** `backend/src/utils/taskRules.ts` — RN01 e `backend/src/utils/normalize.ts` — RN03

# RN01 — Transição de status de tarefa (`backend/src/utils/taskRules.ts`)

## 1. Tabela de casos (elaborada antes de usar a IA)

| # | Entrada | Resultado esperado | Técnica (partição, valor-limite, inválido) | Regra |
|---|---------|--------------------|--------------------------------------------|-------|
| 1 | `PENDING` → `IN_PROGRESS` | `true` | Partição válida | RN01 (tabela) |
| 2 | `PENDING` → `CANCELLED` | `true` | Partição válida | RN01 (tabela; "tarefa pendente pode ser cancelada antes de ser iniciada") |
| 3 | `IN_PROGRESS` → `COMPLETED` | `true` | Partição válida | RN01 (tabela) |
| 4 | `IN_PROGRESS` → `CANCELLED` | `true` | Partição válida | RN01 (tabela) |
| 5 | `PENDING` → `COMPLETED` | `false` | Inválido (pula etapa) | RN01 ("transição não listada é proibida") |
| 6 | `IN_PROGRESS` → `PENDING` | `false` | Inválido (retrocesso) | RN01 ("transição não listada é proibida") |
| 7 | `COMPLETED` → `PENDING` | `false` | Inválido (status final) | RN01 (`COMPLETED` é final) |
| 8 | `COMPLETED` → `IN_PROGRESS` | `false` | Inválido (status final) | RN01 (`COMPLETED` é final) |
| 9 | `COMPLETED` → `CANCELLED` | `false` | Inválido (status final) | RN01 (`COMPLETED` é final) |
| 10 | `CANCELLED` → `PENDING` | `false` | Inválido (status final) | RN01 (`CANCELLED` é final) |
| 11 | `CANCELLED` → `IN_PROGRESS` | `false` | Inválido (status final) | RN01 (`CANCELLED` é final) |
| 12 | `CANCELLED` → `COMPLETED` | `false` | Inválido (status final) | RN01 (`CANCELLED` é final) |
| 13 | `PENDING` → `PENDING` | `false` | Valor-limite (mesmo status) | RN01 ("manter o mesmo status não é transição") |
| 14 | `IN_PROGRESS` → `IN_PROGRESS` | `false` | Valor-limite (mesmo status) | RN01 ("manter o mesmo status não é transição") |
| 15 | `COMPLETED` → `COMPLETED` | `false` | Valor-limite (mesmo status) | RN01 ("manter o mesmo status não é transição") |
| 16 | `CANCELLED` → `CANCELLED` | `false` | Valor-limite (mesmo status) | RN01 ("manter o mesmo status não é transição") |

## 2. Prompt utilizado

```
Escreva o teste unitário da RN01 em docs/especificacao.md. Faça apenas para o backend e não mude o código-fonte em src.
```

## 3. Revisão do teste gerado

| Problema encontrado no teste gerado | Ajuste solicitado ou realizado |
|-------------------------------------|--------------------------------|
| No `it.each` de "mesmo status", o título usava `%s → %s`, mas cada caso recebe só um parâmetro; o segundo `%s` apareceria literal no nome do teste. | Corrigido pelo próprio agente antes da execução: título alterado para `retorna false ao manter o status %s`. |
| O caso `PENDING` → `CANCELLED` é verificado duas vezes: no `it.each` das transições permitidas e no `it` isolado "permite cancelar uma tarefa pendente antes de ser iniciada" (redundância). | Mantido: o teste isolado documenta a regra explícita da RN01 sobre cancelamento antes do início. Pode ser removido se o professor preferir um caso por linha da tabela. |

## 4. Resultados

- Testes criados: 17 | aprovados: 15 | reprovados: 2
- Cobertura de linhas da unidade: 100 % (4/4 linhas; ramos 13/13) em `src/utils/taskRules.ts`
- Score de mutação (backend) ou resultado da sabotagem manual (frontend): não calculado. O Stryker exige que todos os testes passem na execução inicial (*dry run*) e abortou por causa dos 2 testes reprovados (defeito do tópico 5). Para obter o score seria preciso corrigir o código de produção ou pular os testes que falham, o que não foi feito.

## 5. Defeitos encontrados (divergências entre especificação e código)

| Regra | Comportamento especificado | Comportamento observado | Teste que evidencia |
|-------|----------------------------|-------------------------|---------------------|
| RN01 | `canTransition('PENDING', 'CANCELLED')` retorna `true`: uma tarefa pendente pode ser cancelada antes de ser iniciada. | Retorna `false`: o código não permite cancelar uma tarefa pendente. | `canTransition > transições permitidas pela tabela da RN01 > permite transição de PENDING para CANCELLED` e `canTransition > permite cancelar uma tarefa pendente antes de ser iniciada` (`backend/tests/unit/taskRules.test.ts`) |

---

# RN03 — Normalização de nomes (`backend/src/utils/normalize.ts`)

## 1. Tabela de casos

| # | Entrada | Resultado esperado | Técnica (partição, valor-limite, inválido) | Regra |
|---|---------|--------------------|--------------------------------------------|-------|
| 1 | `"  Trabalho   da  Faculdade "` | `"trabalho da faculdade"` | Partição válida (exemplo da especificação) | RN03 (exemplo) |
| 2 | `"   trabalho"` | `"trabalho"` | Partição válida | RN03 passo 1 (remoção no início) |
| 3 | `"trabalho   "` | `"trabalho"` | Partição válida | RN03 passo 1 (remoção no fim) |
| 4 | `"  trabalho  "` | `"trabalho"` | Partição válida | RN03 passo 1 (remoção no início e no fim) |
| 5 | `"\t\ntrabalho\n\t"` | `"trabalho"` | Partição válida (tabulação e quebra de linha nas pontas) | RN03 passo 1 |
| 6 | `"trabalho    da   faculdade"` | `"trabalho da faculdade"` | Partição válida | RN03 passo 2 (vários espaços) |
| 7 | `"trabalho\tda\tfaculdade"` | `"trabalho da faculdade"` | Partição válida | RN03 passo 2 (tabulação) |
| 8 | `"trabalho\nda\nfaculdade"` | `"trabalho da faculdade"` | Partição válida | RN03 passo 2 (quebra de linha) |
| 9 | `"trabalho \t\n da"` | `"trabalho da"` | Partição válida (sequência mista) | RN03 passo 2 |
| 10 | `"trabalho da"` | `"trabalho da"` | Valor-limite (um único espaço interno) | RN03 passo 2 |
| 11 | `"TRABALHO"` | `"trabalho"` | Partição válida | RN03 passo 3 |
| 12 | `"TrAbAlHo"` | `"trabalho"` | Partição válida | RN03 passo 3 |
| 13 | `"AÇÃO"` | `"ação"` | Partição válida (letras acentuadas) | RN03 passo 3 |
| 14 | `""` | `""` | Valor-limite (string vazia) | RN03 passos 1 a 3 |
| 15 | `" \t\n "` | `""` | Valor-limite (apenas espaços em branco) | RN03 passos 1 e 2 |
| 16 | `"trabalho da faculdade"` | `"trabalho da faculdade"` | Valor-limite (já normalizado) | RN03 passos 1 a 3 |
| 17 | `"a"` | `"a"` | Valor-limite (um único caractere) | RN03 passos 1 a 3 |
| 18 | `"Trabalho da Faculdade"` e `"  TRABALHO\t da   faculdade\n"` | normalizados iguais (duplicados) | Partição válida | RN03 (duplicados) |
| 19 | `"Trabalho da Faculdade"` e `"Trabalho de Faculdade"` | normalizados diferentes (não duplicados) | Partição válida | RN03 (duplicados) |

## 2. Prompt utilizado

```
vamos fazer agora para o rn03
```

(Prompt enviado na mesma sessão do teste da RN01, cujas instruções — apenas backend, sem alterar `src/` — continuaram valendo.)

## 3. Revisão do teste gerado

| Problema encontrado no teste gerado | Ajuste solicitado ou realizado |
|-------------------------------------|--------------------------------|
| O caso 5 (tabulação e quebra de linha nas pontas) depende de uma interpretação: o passo 1 fala em "espaços em branco" sem listar os caracteres; o agente usou a definição do passo 2 (espaço, tabulação, quebra de linha). | Mantido, com a dúvida resolvida pela própria especificação. Na revisão foi considerada uma leitura estrita do passo 1 (remover apenas o caractere espaço): seguindo a ordem 1 → 2 → 3, `"\t\ntrabalho\n\t"` resultaria em `" trabalho "`, com espaço sobrando nas pontas. Essa leitura foi descartada porque a especificação usa o mesmo termo, "espaços em branco", nos passos 1 e 2, e no passo 2 o define como espaço, tabulação e quebra de linha. Portanto o passo 1 também remove tabulação e quebra de linha das pontas, e o resultado esperado é `"trabalho"`, seguindo a ordem especificada. |
| Os casos 18 e 19 (duplicados) verificam apenas o critério de duplicidade: dois nomes são duplicados quando `normalizeName` produz o mesmo resultado. A RN03 diz que esses nomes "são considerados duplicados", mas não informa o que deve acontecer com uma categoria duplicada (bloquear a criação ou a edição, status de resposta da API, mensagem) nem onde a regra se aplica. Sem esse uso definido, a finalidade da informação não fica clara. | Mantido como teste do critério de comparação, que é o que a unidade `normalizeName` oferece. Nenhum teste da ação sobre duplicados foi criado: não há comportamento especificado para servir de oráculo, e a ação não pertence a `normalize.ts`. Lacuna registrada no tópico 5. |

## 4. Resultados

- Testes criados: 19 | aprovados: 19 | reprovados: 0
- Cobertura de linhas da unidade: 100 % (1/1 linha; 1/1 função) em `src/utils/normalize.ts`
- Score de mutação (backend) ou resultado da sabotagem manual (frontend): 100 % (6 mutantes gerados, 6 mortos, 0 sobreviventes)

O score de mutação foi obtido **por fora** do script do projeto. O `npm run test:mutation` não roda enquanto houver testes reprovados (os 2 da RN01), então o agente executou o Stryker diretamente, com duas configurações temporárias criadas fora do repositório: elas mutam só `src/utils/normalize.ts` e executam só `tests/unit/normalize.test.ts`. Nenhum teste nem arquivo do projeto foi alterado ou pulado.

Comando executado dentro de `backend/`:

```
npx stryker run <pasta-temporária>/stryker.rn03.json
```

`stryker.rn03.json`, que muta apenas `normalize.ts` e aponta para a configuração do Vitest abaixo:

```json
{
  "testRunner": "vitest",
  "mutate": ["src/utils/normalize.ts"],
  "reporters": ["clear-text"],
  "coverageAnalysis": "perTest",
  "tempDirName": ".stryker-tmp",
  "vitest": { "configFile": "<pasta-temporária>/vitest.rn03.config.mts" }
}
```

`vitest.rn03.config.mts`, que executa apenas o teste da RN03:

```ts
export default { test: { environment: 'node', include: ['tests/unit/normalize.test.ts'] } }
```

## 5. Defeitos encontrados (divergências entre especificação e código)

| Regra | Comportamento especificado | Comportamento observado | Teste que evidencia |
|-------|----------------------------|-------------------------|---------------------|
| RN03 | — | Nenhuma divergência entre especificação e código encontrada. | — |
| RN03 (lacuna da especificação, não defeito do código) | "Dois nomes de categoria que resultem no mesmo nome normalizado são considerados duplicados." Não há ação definida para duplicados (bloqueio, status da API, mensagem) nem indicação de onde a regra se aplica. | Não verificável: sem comportamento especificado, não há oráculo para testar o tratamento de duplicados. Apenas o critério de comparação foi testado. | `normalizeName > detecção de duplicados` (casos 18 e 19, `backend/tests/unit/normalize.test.ts`) |
