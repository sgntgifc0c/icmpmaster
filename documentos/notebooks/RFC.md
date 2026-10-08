# RFC: Preditor de degradação de rede com RTT normalizado

| | |
|---|---|
| **Curso / Disciplina** | Ciência da Computação / Estruturas de Dados II |
| **Projeto integrador** | icmpmaster |
| **Orientador(a)** | Andrea Ono Sakai |
| **Equipe** | icmpmaster |
| **Integrantes** | Eduardo Quintino Filho, Felipe Veiga da Silva, Henrique de Aguiar Fernandes, Niccolas Lupetti dos Santos, Tharik Lima da Silva |
| **Scrum Master da tarefa** | Henrique de Aguiar Fernandes |
| **Data** | 05/10/2026 |
| **Tarefa de referência** | Tarefa 1: Problema e coleta bruta (sem rótulo) |
| **Documento relacionado** | Memorando de Decisão: Fonte de Dados do Projeto (08/09/2026) |
| **Repositório** | https://github.com/sgntgifc0c/icmpmaster |

---

## 1. Problema

A árvore de decisão (que será treinada em etapa futura) vai classificar a **degradação do fluxo em relação ao próprio normal**, em três estados possíveis: **OK**, **RISCO** ou **FALHA**.

Isso **não** é o mesmo que "RTT acima de um valor fixo" nem "é uma rota longa". A referência de comparação é o comportamento histórico do **próprio fluxo**, não um limiar global.

**Unidade de análise (fluxo):** `fluxo_id = probe_id|dst_addr`, ou seja, origem e destino. O `measurement_id` é guardado como coluna separada e **não** compõe o identificador.

**O que é uma linha do dado bruto:** uma medição ICMP (ping) de um fluxo, com timestamp. Nesta etapa, ainda **sem classe/rótulo**.

**Quem usa o resultado final:** quem opera o enlace de rede.

| Classe | Ação do operador |
|---|---|
| OK | Não agir |
| RISCO | Observar |
| FALHA | Investigar |

---

## 2. Horizonte

| Etapa | O que faz | Quando entra |
|---|---|---|
| Detector | Classifica o estado da medição **atual** | Tarefa 3, com a árvore de decisão |
| Preditor | Classifica o estado esperado **12 minutos à frente** | Tarefa 3, com a árvore de decisão |

Nesta Tarefa 1, nenhuma das duas etapas é implementada; ambas ficam apenas registradas como plano futuro.

---

## 3. Custo de errar FALHA

**Decisão da equipe:** priorizar a redução de falsos negativos (FALHA real classificada como OK ou RISCO), aceitando mais falsos positivos (falsos alarmes).

**Justificativa:** o custo de não perceber uma falha de rede é maior do que o custo de investigar um alerta que depois se mostra normal. Uma falha não detectada pode se agravar e impactar o serviço antes que alguém tome ação, enquanto um falso alarme custa apenas o tempo de verificação de quem opera o enlace.

**Limite dessa escolha:** falsos positivos em excesso geram fadiga de alertas e levam o operador a ignorar o aviso. Por isso a classe RISCO existe como estágio intermediário (observar, sem investigar).

| Erro | Consequência | Gravidade |
|---|---|---|
| FALHA classificada como OK | Falha passa sem nenhuma ação | Alta |
| FALHA classificada como RISCO | Detecção atrasada | Média |
| OK classificado como FALHA | Investigação desnecessária | Baixa/Média |

---

## 4. Fonte de dados (decisão do memorando)

O pipeline do projeto exige que qualquer fonte produza registros que se transformem em janelas e, por fim, em `X = [latência, perda, jitter]`. O memorando de 08/09/2026 comparou duas opções e recomendou a **Opção B**.

| Critério | Opção A: dataset Kaggle (descartada) | Opção B: API do RIPE Atlas (escolhida) |
|---|---|---|
| Realismo | Dados emulados em testbed de laboratório | Medições reais de sondas distribuídas pelo mundo |
| Cobertura temporal | ~11 h 26 min em um único dia (05/11/2024) | Intervalo configurável |
| Diversidade de caminhos | Limitada ao testbed | Permite caminhos curtos e longos |
| Ampliação da coleta | Conteúdo fixo | Possível por período e por medição |
| Custo | Gratuito | Leitura pública gratuita; créditos só para criar medições |

**Decisão:** usar a API do RIPE Atlas, somente com requisições `GET` a medições públicas já existentes.

**Por que o Kaggle foi descartado:** cobre um único dia, o que impede separar Período A (baseline) e Período B (rotulado); é emulado, sem a variabilidade real da internet; e não pode ser ampliado.

**Parâmetros definidos no memorando:**

| Parâmetro | Valor | Observação |
|---|---|---|
| Tipo de medição | Ping (ICMP) | Fornece RTT, enviados e recebidos, base para latência, perda e jitter |
| Medição validada | `measurement_id = 7714918` | Validada com `GET` exploratório; os `measurement_id` finais ficam em `config/` |
| Janela exploratória | 2 horas | Usada apenas no teste inicial do memorando |
| Formato de saída | CSV | Facilita auditoria e leitura em qualquer ferramenta |

**Janela final da coleta:** a janela de 2 horas foi apenas exploratória. A coleta desta tarefa usa os dois blocos descritos na Seção 5.

---

## 5. Dados e coleta

- **Fonte:** medições públicas já existentes de ping IPv4 (mesh de Anchors do RIPE Atlas), somente `GET`.
- **Diversidade:** vários fluxos, com pelo menos um caminho curto e um caminho longo no mesmo período.
- **Dois blocos de tempo contíguos, sem sobreposição:**
  - **Período A:** usado somente para o baseline da Tarefa 2.
  - **Período B:** medições que serão rotuladas.
  - Referência do projeto: 7 dias + 7 dias a partir de 06/09/2026 04:32 UTC.
- **Campos brutos:** `timestamp`, `measurement_id`, `probe_id`, `dst_addr`, `fluxo_id`, RTT da rajada (médio e, se existirem, mínimo e máximo), enviados, recebidos, `perda_pct`, `jitter_ms`, `timeout_atual`.
- **Cálculo dos campos derivados:**
  - `perda_pct = (enviados − recebidos) / enviados × 100`
  - `jitter_ms` = desvio-padrão dos RTT da rajada, somente com 2 ou mais respostas; caso contrário, vazio.
- **Ausência de dados:** RTT vazio nunca vira 0; jitter sem base nunca vira 0; o timeout permanece no arquivo.
- **Bruto preservado:** o JSON original é mantido e a tabela tratada não o apaga.
- **Robustez da coleta:** requisições HTTP com timeout, releitura em erro transitório e coleta idempotente (rodar de novo não duplica).
- **Configuração externa:** período, probes, destinos e `measurement_id` ficam em `config/`, não espalhados no código.

---

## 6. Papel da rota, país e destino nesta etapa

País, IP e nome da rota **podem** ser registrados no dado bruto, apenas para **auditoria de diversidade** (confirmar que há caminhos curtos e longos no mesmo arquivo).

O `dst_addr` faz parte do `fluxo_id` (identificador do fluxo), mas **não** será usado como variável de entrada (feature) da árvore de decisão. País, IP e nome da rota também ficam fora da árvore.

---

## 7. Riscos e premissas

| Risco / premissa | Mitigação |
|---|---|
| Esgotamento de créditos, caso sejam criadas medições novas | Priorizar `GET` em medições públicas e não criar medições próprias |
| Indisponibilidade pontual de sondas ou dados faltantes | Selecionar grupos amplos de sondas por região; manter timeouts e RTT vazio no arquivo |
| Erro transitório ou limite de requisições da API | Timeout nas requisições, releitura em erro transitório, coleta idempotente |
| Medição escolhida deixar de retornar dados | Guardar `measurement_id` em `config/` e validá-lo antes de cada coleta |
| Poucos dados no Período A para o mínimo da Tarefa 2 | Conferir o volume por fluxo no relatório de qualidade antes de encerrar a coleta |
| Jitter não vir pronto na API | Calcular a partir dos RTT da rajada, somente com 2 ou mais respostas |

---

## 8. Fora de escopo

**Nesta Tarefa 1:**

- Rótulo/classe (OK, RISCO, FALHA) em qualquer linha do dado
- Baseline ou valor "normal" de referência
- Mediana, `z_robusto` ou qualquer estatística de normalização
- Split de treino/teste
- Treinamento da árvore de decisão ou qualquer métrica de modelo

**Em todo o projeto:**

- Criar medição própria no RIPE Atlas ou gastar crédito (apenas `GET` em medições públicas)
- Usar o dataset do Kaggle como fonte de dados
- Transformar timeout ou RTT vazio em 0
- Usar RTT absoluto, país, continente ou nome da rota como definição de falha
- Usar país, IP ou nome da rota como variável da árvore

**Justificativa:** separar a coleta (neutra) da decisão de rotulação (Tarefa 2), evitando que a interpretação do time contamine o dado bruto.

---

## 9. Referências

- Diário da Tarefa 1: `Tarefa1_Coleta_Bruta.md`
- Dicionário de dados: `dicionario_v0.1.md`
- Memorando de Decisão: Fonte de Dados do Projeto (08/09/2026)
- Repositório: https://github.com/sgntgifc0c/icmpmaster
- RIPE Atlas, resultados de medições: https://atlas.ripe.net/docs/apis/rest-api-manual/measurements/results-and-latest
- RIPE Atlas, autenticação: https://atlas.ripe.net/docs/apis/rest-api-manual/authentication/
- RIPE Atlas, medições definidas pelo usuário: https://atlas.ripe.net/docs/getting-started/user-defined-measurements.html
- Dataset descartado: https://www.kaggle.com/datasets/kaiser14/network-anomaly-dataset