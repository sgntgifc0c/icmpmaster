# RFC — Preditor de degradação de rede com RTT normalizado


**Data:**
05/10/2026

**Tarefa de referência:** Tarefa 1 — Problema e coleta bruta (sem rótulo)


## 1. Problema

A árvore de decisão (que será treinada em etapa futura) vai classificar a **degradação do fluxo em relação ao próprio normal**, em três estados possíveis: **OK**, **RISCO** ou **FALHA**.

Importante: isso **não** é a mesma coisa que "RTT acima de um valor fixo" nem "é uma rota longa". A referência de comparação é o comportamento histórico do **próprio fluxo**, não um limiar global.

**Unidade de análise (fluxo):**
```
fluxo_id = probe_id | dst_addr
```
(origem, destino, e `measurement_id` quando existir)

**O que é uma linha do dado bruto:** uma medição ICMP (ping) de um fluxo, com timestamp — nesta etapa, ainda **sem classe/rótulo**.

**Quem usa o resultado final:** quem opera o enlace de rede — decide se investiga (FALHA), observa (RISCO) ou não age (OK).

---

## 2. Horizonte

| Etapa | O que faz | Quando entra |
|---|---|---|
| Detector | Classifica o estado da medição **atual** | — |
| Preditor | Classifica o estado esperado **12 minutos à frente** | — |
| Árvore de decisão (treino) | Usa os dados rotulados | Só na **Tarefa 3** |

Nesta Tarefa 1, nenhum desses pontos é implementado — apenas registrado como plano futuro.

---

## 3. Custo de errar FALHA

> *Esta é a decisão que a equipe precisa tomar e justificar.*

Perguntas-guia para a equipe discutir:

- O que é mais caro para quem opera a rede: **investigar à toa** (falso alarme) ou **deixar passar uma falha real** (falso negativo)?
- Existe algum cenário em que um falso alarme tem custo baixo (ex: só gera uma notificação) enquanto uma falha não detectada pode causar impacto maior (ex: queda de serviço sem ninguém perceber)?
- Essa resposta pode mudar dependendo do tipo de fluxo (ex: caminho crítico vs. caminho secundário)?

**Decisão da equipe:**
A equipe prioriza evitar falsos negativos (deixar passar uma FALHA real), mesmo que isso gere mais falsos positivos (falso alarme). Justificativa: o custo de não perceber uma falha de rede é maior do que o custo de investigar um alerta que depois se mostra normal — uma falha não detectada pode se agravar e impactar o serviço antes que alguém tome ação, enquanto um falso alarme custa apenas o tempo de verificação de quem opera o enlace.

---

## 4. Fora de escopo (nesta tarefa)

Explicitamente **não** fazem parte da Tarefa 1:

- Rótulo/classe (OK, RISCO, FALHA) em qualquer linha do dado
- Baseline ou valor "normal" de referência
- Mediana, `z_robusto`, qualquer estatística de normalização
- Split de treino/teste
- Treinamento da árvore de decisão ou qualquer métrica de modelo
- Usar RTT absoluto, país, continente ou nome da rota como definição de falha

**Justificativa:** separar a coleta (neutra) da decisão de rotulação (Tarefa 2), evitando que a interpretação do time "contamine" o dado bruto.

---

## 5. Papel da rota/país nesta etapa

País, IP de origem/destino e nome da rota **podem** ser registrados no dado bruto, mas apenas para fins de **auditoria de diversidade** (confirmar que há caminhos curtos e longos no mesmo arquivo). Eles **não** serão usados como variável de entrada (feature) da árvore de decisão.

---

## 6. Referências

- Diário da Tarefa 1: `Tarefa1_Coleta_Bruta.md`
- Dicionário de dados: `dicionario_v0.1.md` 
- Repositório: `https://github.com/sgntgifc0c/icmpmaster`
