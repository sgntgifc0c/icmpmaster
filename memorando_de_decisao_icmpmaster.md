# Memorando de Decisão — Fonte de Dados do Projeto


| Campo | Informação |
|---|---|
| Curso / Disciplina | `Ciência da Computação / Estruturas de dados II` |
| Projeto integrador | `icmpmaster` |
| Orientador(a) | `Andrea Ono Sakai` |
| Data de entrega desta etapa | `08/09/2026` |
| Integrantes do grupo | `Eduardo Quintino Filho, Felipe Veiga da Silva, Henrique de Aguiar Fernandes, Niccolas Lupetti dos Santos, Tharik Lima da Silva` |

---

> Preencha cada seção com o que você encontrou na pesquisa. Não deixe nenhum campo com o texto entre colchetes — substitua pelo seu conteúdo. Toda informação levantada nas Opções A e B precisa indicar a fonte de onde veio.

## 1. Situação

<!-- Em uma frase: qual decisão precisa ser tomada e por quê. 
O pipeline do projeto já está definido: qualquer fonte de dados precisa produzir registros que se transformem em janelas e, por fim, em X = [latência, perda, jitter]. Falta decidir de onde virão esses dados na próxima fase. A equipe do projeto precisa recomendar, com base em pesquisa e não em preferência pessoal, se a próxima etapa deve usar um dataset real já publicado ou a API do RIPE Atlas. O grupo deve produzir um memorando de decisão com a recomendação da tomada de decisão. A recomendação só tem valor se for sustentada por pesquisa real — não existe resposta pronta para copiar; ela precisa ser construída a partir do que vocês encontraram.
-->


A equipe precisa decidir entre utilizar um dataset real publicado ou a API do RIPE Atlas para alimentar o pipeline com métricas de rede (latência, perda e jitter), garantindo a viabilidade técnica e a robustez necessárias para a próxima fase do projeto.

## 2. Opção A — Dataset real

<!-- O que foi encontrado sobre um dataset real de ICMP. Cite a fonte de cada informação. -->

- **Origem / link:**
 https://www.kaggle.com/datasets/kaiser14/network-anomaly-dataset

- **Formato:**
 CSV

- **Período coberto:** 
Dados estáticos consolidados de monitoramento de tráfego

- **Campos disponíveis:**
 timestamp; bandwidth; throughput; congestion; packet_loss; latency; jitter; Routers; Planned route; Network measure; Network target; Video target; Percentage video occupancy; Bitrate video; Number videos

- **Licença de uso:** 
Apache 2.0

**Resumo do que foi encontrado:**
Foi encontrado um dataset estruturado em formato tabular no Kaggle, sob licença Apache 2.0, que contém registros de monitoramento de rede e tráfego multimídia sob carga. Esse conjunto de dados oferece diretamente as colunas de desempenho necessárias para o pipeline do projeto — como latency, packet_loss e jitter — permitindo o uso imediato dos dados de forma estática. Não foi necessário configurar nenhuma infraestrutura de captura em tempo real, pois os dados já estão prontos para serem utilizados.

## 3. Opção B — API do RIPE Atlas

<!-- O que foi encontrado sobre a API: autenticação, criação e consulta de medições. Cite a fonte de cada informação. -->

- **Documentação consultada (link):** 
https://atlas.ripe.net/docs/getting-started/what-is-ripe-atlas

- **Autenticação exigida:** 

Chaves de API (API Keys) para acesso programático ou autenticação por sessão. A maioria das leituras funcionam sem autenticação, mas é obrigatória para criar ou gerenciar medições.

- **Como se cria uma medição:**
Para criar uma medição, você usa um assistente de três passos na interface web ou manda uma requisição pela API. Primeiro, nas definições, você escolhe o tipo de teste (como ping ou traceroute) e o IP ou site alvo. Depois, seleciona as sondas de rede (pode deixar 50 aleatórias pelo mundo ou escolher por país/mapa). Por fim, no agendamento, você define quando o teste começa e termina, ou marca como One-off para rodar apenas uma vez na hora.

- **Como se consultam os resultados:**
Para consultar os resultados, você acessa a página de resultados da medição específica (clicando no ID dela na lista) ou faz uma requisição GET para o endpoint da API da plataforma informando o ID da medição (/api/v2/measurements/<id>/results/), sendo que dados públicos podem ser acessados livremente e dados restritos exigem chave de API.

**Resumo do que foi encontrado:**
O RIPE Atlas é uma infraestrutura global de monitoramento ativo de redes que utiliza milhares de sondas para coletar métricas de desempenho como latência, perda e rotas. Por meio da sua API, é possível realizar consultas públicas e recuperar resultados de testes anteriores via requisições GET de forma gratuita e sem autenticação, ou criar novas medições customizadas exigindo autenticação por chave de API e o consumo de créditos da plataforma.

## 4. Comparação

<!-- Preencha a tabela com base no que você levantou nas seções 2 e 3. -->

| Critério | Opção A — Dataset real | Opção B — API RIPE Atlas |
|---|---|---|
| Controle sobre a coleta | |B|
| Diversidade geográfica | |B|
| Custo / complexidade de implementação | |B|
| Tempo até os primeiros dados estarem disponíveis |A| |

## 5. Recomendação

<!-- Uma frase direta: qual opção você recomenda. -->

Recomendo a utilização da Opção B(RIPE Atlas) para o projeto.

## 6. Justificativa

<!-- Por que essa opção vence a outra, com base nas evidências das seções 2, 3 e 4 — não em preferência pessoal. -->

A Opção B vence a Opção A porque oferece controle ativo sobre a coleta de dados e ampla diversidade geográfica através de milhares de sondas reais globalmente distribuídas, enquanto o dataset estático (Opção A) é limitado, desatualizado e não permite direcionar testes para os alvos específicos necessários. Embora a Opção B apresente maior complexidade de implementação e dependa de créditos para novas medições, a possibilidade de consultar dados históricos e públicos existentes de forma gratuita via requisições GET compensa a barreira técnica, garantindo métricas reais e sob demanda.

## 7. Riscos e limitações

<!-- O que pode dar errado com a opção escolhida, e como isso poderia ser mitigado. -->

O principal risco é o esgotamento de créditos por excesso de novas medições e a indisponibilidade pontual de sondas, mas isso é mitigado priorizando consultas gratuitas em dados já existentes (GET), selecionando grupos amplos de sondas por região e implementando um tratamento de erros adequado.

## 8. Contribuição Individual dos Integrantes

<!-- cada integrante deve descrever, com suas próprias palavras, o que efetivamente fez nesta etapa. Contribuições genéricas como "ajudei em tudo" não serão aceitas. Use verbos de ação e seja específico (ex.: "pesquisei , analisei, testei, ... apresentei prós/contras ao grupo, ...").-->

### Integrante 1 — `Eduardo Quintino Filho`
- **O que fez nesta etapa:** `Pesquisei e preenchi parte da Dataset`
- **Tempo dedicado (aprox.):** `1h20`
- **Evidência da contribuição**:
**Imagem da evidencia na mesma pasta do memorando**

### Integrante 2 — `Felipe Veiga da Silva`
- **O que fez nesta etapa:** `Auxiliei o Eduardo com a pesquisa da Dataset e ajudei a preencher o campo 2`
- **Tempo dedicado (aprox.):** `1h20`
- **Evidência da contribuição**:
**Imagem da evidencia na mesma pasta do memorando**

### Integrante 3 — `Henrique de Aguiar Fernandes`
- **O que fez nesta etapa:** `Fiz parte da pesquisa sobre a API do RIPE Atlas`
- **Tempo dedicado (aprox.):** `1h00`
- **Evidência da contribuição**:
**Imagem da evidencia na mesma pasta do memorando**

### Integrante 4 — `Niccolas Lupetti dos Santos`
- **O que fez nesta etapa:** `Preenchi o campo 1, auxiliei o Tharik com a finalização da pesquisa sobre a API do RIPE Atlas e ajudei a preencher os demais campos`
- **Tempo dedicado (aprox.):** `1h40`
- **Evidência da contribuição**:

**Imagem da evidencia na mesma pasta do memorando**

### Integrante 5 — `Tharik Lima da Silva`
- **O que fez nesta etapa:** `Finalizei a pesquisa sobre a API do RIPE Atlas e preenchi os demais campos`
- **Tempo dedicado (aprox.):** `1h40`
- **Evidência da contribuição**:
**Imagem da evidencia na mesma pasta do memorando**

---

## Fontes consultadas

<!-- Mínimo de 3 fontes. Liste todas as páginas de documentação, artigos ou repositórios usados. -->

1. https://atlas.ripe.net/docs/apis/rest-api-manual/measurements/results-and-latest

2. https://atlas.ripe.net/docs/apis/rest-api-manual/measurements/listing-measurements

3. https://atlas.ripe.net/docs/apis/rest-api-manual/authentication/

4. https://www.kaggle.com/datasets/kaiser14/network-anomaly-dataset

5. https://atlas.ripe.net/docs/getting-started/user-defined-measurements.html


