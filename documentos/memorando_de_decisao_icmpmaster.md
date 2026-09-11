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
Período coberto: de 05/11/2024 - 12:00:15 a 05/11/2024 - 23:26:43, conforme os valores mínimo e máximo encontrados na coluna timestamp do arquivo network_dataset.csv.

- **Campos disponíveis:**
 timestamp; bandwidth; throughput; congestion; packet_loss; latency; jitter; Routers; Planned route; Network measure; Network target; Video target; Percentage video occupancy; Bitrate video; Number videos

- **Licença de uso:** 
Apache 2.0

**Resumo do que foi encontrado:**
Foi encontrado um dataset estruturado em formato tabular no Kaggle, sob licença Apache 2.0, que contém registros de monitoramento de rede e tráfego multimídia sob carga. Esse conjunto de dados oferece diretamente as colunas de desempenho necessárias para o pipeline do projeto — como latency, packet_loss e jitter — permitindo o uso imediato dos dados de forma estática. Não foi necessário configurar nenhuma infraestrutura de captura em tempo real, pois os dados já estão prontos para serem utilizados.

**Ambiente de coleta:** 
Os dados foram produzidos em um ambiente de rede simulado e controlado, estruturado com ferramentas de emulação de rede para replicar condições reais. A topologia incluiu nós interconectados (servidores, roteadores e switches) configurados para gerar tráfego diversificado e permitir o controle preciso de parâmetros, viabilizando a injeção controlada de comportamentos normais e anômalos. Portanto, o ambiente de coleta possui caráter de testbed/laboratório, utilizando roteadores e dispositivos emulados, e não representa diretamente o tráfego de uma rede de produção em larga escala.

## 3. Opção B — API do RIPE Atlas

<!-- O que foi encontrado sobre a API: autenticação, criação e consulta de medições. Cite a fonte de cada informação. -->

- **Documentação consultada (link):** 
https://atlas.ripe.net/docs/getting-started/what-is-ripe-atlas

- **Autenticação exigida:** 
A API do RIPE Atlas oferece autenticação por API key, recomendada para acesso programático, e autenticação baseada em sessão para aplicações web. A maioria das operações de leitura pode ser realizada sem autenticação, incluindo a consulta de resultados públicos. Entretanto, a criação de novas medições exige uma API key com a permissão correspondente.

- **Como se cria uma medição:**
Pela API REST do RIPE Atlas, uma nova medição é criada por meio de uma requisição POST para o endpoint /api/v2/measurements/. O corpo da requisição é enviado em JSON e deve conter pelo menos uma definição da medição, incluindo informações como tipo do teste e alvo, além da seleção das probes que executarão o teste. Também podem ser definidos parâmetros de início, término e execução única ou recorrente. A criação exige uma API key com permissão para criar medições. Em caso de sucesso, a API retorna os identificadores das medições criadas.

- **Como se consultam os resultados:**
Para consultar os resultados, você acessa a página de resultados da medição específica (clicando no ID dela na lista) ou faz uma requisição GET para o endpoint da API da plataforma informando o ID da medição (/api/v2/measurements/<id>/results/), sendo que dados públicos podem ser acessados livremente e dados restritos exigem chave de API.

**Resumo do que foi encontrado:**
O RIPE Atlas é uma infraestrutura global de monitoramento ativo de redes que utiliza milhares de sondas para coletar métricas de desempenho como latência, perda e rotas. Por meio da sua API, é possível realizar consultas públicas e recuperar resultados de testes anteriores via requisições GET de forma gratuita e sem autenticação, ou criar novas medições customizadas exigindo autenticação por chave de API e o consumo de créditos da plataforma.

## 4. Comparação

<!-- Preencha a tabela com base no que você levantou nas seções 2 e 3. -->

| Decisão | Escolha da equipe | Justificativa |
|---|---|---|
| **ID da medição (measurement_id)** | **7714918** | Escolhemos essa medição pública do RIPE Atlas por ser do tipo ping (ICMP), com alvo estável e ampla cobertura de sondas. O ID foi validado previamente via requisição GET no navegador, confirmando que retorna resultados para o período consultado. |
| **Intervalo consultado** | **Últimas 2 horas antes do momento da execução** | Optamos por uma janela curta para garantir volume suficiente de registros sem tornar a consulta lenta ou ultrapassar limites da API. O período é calculado dinamicamente em UTC a partir do instante da execução, garantindo dados recentes e reprodutibilidade. |
| **Duração do intervalo** | **2 horas** | O período de 2 horas mostrou-se adequado após teste exploratório: retornou quantidade significativa de medições (suficiente para análise posterior) sem risco de timeout ou estouro de memória. |
| **Tipo de medição** | **Ping (ICMP)** | O tipo ping é obrigatório para o projeto, pois é o único que fornece as métricas exigidas pelo contrato de dados (`latencia_ms`, `perda_pacotes_pct`, `jitter_ms`). Traceroute, DNS ou HTTP não fornecem esses campos diretamente. |
| **Formato de saída: CSV ou Parquet** | **CSV** | Escolhemos CSV por ser o formato alinhado ao contrato de dados do projeto (`log_rede.csv`), facilitando auditoria e leitura direta por qualquer ferramenta (Excel, pandas, etc.). Parquet exigiria `pyarrow` e seria menos legível nesta fase de inspeção bruta. |

## 5. Recomendação

<!-- Uma frase direta: qual opção você recomenda. -->

Recomendo a utilização da Opção B(RIPE Atlas) para o projeto.

## 6. Justificativa

<!-- Por que essa opção vence a outra, com base nas evidências das seções 2, 3 e 4 — não em preferência pessoal. -->

A Opção B foi escolhida por oferecer controle ativo sobre a coleta de dados e ampla diversidade geográfica por meio de sondas globais distribuídas, o que possibilita direcionar testes para alvos específicos. Enquanto a Opção A oferece mais praticidade e baseado em um dataset estático e de escopo fixo, a Opção B traz maior flexibilidade para cenários dinâmicos. Embora a Opção B apresente maior complexidade de implementação e dependência de créditos para novas medições ativas, o acesso gratuito a dados históricos e públicos já existentes via requisições compensa essa barreira, garantindo métricas reais e sob demanda. Além disso, caso os GET's gratuitos esgotassem, existe a opção de solicitar a equipe fornecedora (através do email: atlas@ripe.net)  para mais créditos. https://atlas.ripe.net/docs/getting-started/user-defined-measurements

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


