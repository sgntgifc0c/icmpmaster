# Dicionário de Dados — v0.1 (dado bruto, sem rótulo)

**Projeto:** Preditor de degradação de rede com RTT normalizado
**Referência:** Tarefa 1 — Coleta bruta
**Escopo:** apenas variáveis brutas da medição. Nenhuma coluna de classe (OK/RISCO/FALHA), baseline ou estatística derivada entra nesta versão.

---

| Campo | Tipo | Unidade | Descrição | Pode ser vazio? |
|---|---|---|---|---|
| `timestamp` | datetime (UTC) | — | Momento em que a medição ICMP foi realizada. Usado para ordenar os registros de cada fluxo. | Não |
| `measurement_id` | inteiro | — | Identificador da medição na RIPE Atlas à qual o registro pertence. Compõe a identidade do fluxo quando existir. | Não |
| `probe_id` | inteiro | — | Identificador do probe (origem) que executou a medição. | Não |
| `dst_addr` | texto (IP) | — | Endereço IP de destino da medição. | Não |
| `fluxo_id` | texto | — | Chave estável do fluxo, construída como `probe_id \| dst_addr` (mais `measurement_id` quando existir). Usada para agrupar registros do mesmo par origem-destino ao longo do tempo. | Não |
| `rtt_avg_ms` | float | ms | RTT médio da rajada de pacotes ICMP enviados nessa medição. | Sim — vazio se não houve nenhuma resposta (nunca usar 0 para indicar ausência) |
| `rtt_min_ms` | float | ms | RTT mínimo da rajada, quando a API fornecer esse detalhe. | Sim |
| `rtt_max_ms` | float | ms | RTT máximo da rajada, quando a API fornecer esse detalhe. | Sim |
| `enviados` | inteiro | contagem | Quantidade de pacotes ICMP enviados na rajada. | Não |
| `recebidos` | inteiro | contagem | Quantidade de pacotes ICMP recebidos de volta na rajada. | Não |
| `perda_pct` | float | % | Percentual de perda de pacotes, calculado como `(enviados − recebidos) / enviados × 100`. | Não (sempre calculável a partir de enviados/recebidos) |
| `jitter_ms` | float | ms | Desvio-padrão dos RTTs da rajada, calculado apenas quando há 2 ou mais respostas na mesma rajada. | Sim — vazio se houver menos de 2 respostas (nunca usar 0 para indicar estabilidade) |
| `timeout_atual` | inteiro (0 ou 1) | — | Indicador binário: 1 se não houve RTT registrado ou se a perda foi de 100% nessa medição; 0 caso contrário. | Não |
| `pais_origem` / `pais_destino` | texto | — | País associado ao probe de origem e ao destino, usado apenas para auditoria de diversidade geográfica (confirmar presença de caminho curto e longo). **Não é feature da árvore.** | Sim |
| `rota_nome` | texto | — | Identificação textual da rota/caminho, quando aplicável, apenas para auditoria. **Não é feature da árvore.** | Sim |

---

## Observações gerais

- Nenhum campo de classe/rótulo (`status`, `classe`, `OK/RISCO/FALHA`) existe nesta versão — isso é proibido nesta etapa.
- `rtt_avg_ms` e `jitter_ms` nunca recebem `0` como substituto de "sem dado": ausência é representada como valor vazio/nulo.
- `pais_origem`, `pais_destino` e `rota_nome` existem apenas para auditoria da diversidade de caminhos coletados (curto vs. longo) — não entram como variável de entrada do modelo em nenhuma etapa futura.
- Este dicionário cobre apenas o dado bruto. Colunas derivadas (baseline, z-score, rótulo) serão documentadas em uma v0.2, na Tarefa 2.

## Changelog

- **v0.1** — Versão inicial, campos brutos da coleta (Tarefa 1).
