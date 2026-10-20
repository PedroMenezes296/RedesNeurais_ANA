# Relatório técnico — Estudo de séries hidrológicas da ANA com redes neurais

**Projeto de origem:** Projeto Integrador 5 (PI5) — monitoramento de nível de água e apoio à identificação de condições de cheia e de nível baixo  
**Uso adicional pretendido:** projeto acadêmico independente de uma disciplina de Redes Neurais  
**Data da elaboração:** 19/10/2026  
**Natureza:** proposta de pesquisa e plano de trabalho, **não** um estudo com modelos já treinados  
**Fonte primária dos dados analisados:** arquivos CSV fornecidos pelo estudante, exportados do HidroWeb / Agência Nacional de Águas e Saneamento Básico (ANA).

> **Orientação para outro chat:** este documento reúne o contexto e as decisões já discutidas. O passo seguinte é construir um notebook Python reprodutível, começando por leitura, limpeza, validação e análise exploratória dos CSVs. **Não assumir resultados preditivos, correlações ou métricas antes de calculá-los.** Pedir os três CSVs originais caso ainda não estejam anexados no novo chat.

---

## 1. Resumo executivo

O objetivo é investigar se **históricos de precipitação, nível (cota) e vazão de um rio** permitem caracterizar padrões associados a períodos de cheia e de estiagem e, posteriormente, treinar e avaliar uma **rede neural para prever níveis ou vazões futuros**. O estudo começa por estações localizadas em **Encantado, no Rio Grande do Sul**, na região da bacia Taquari-Antas.

A proposta tem duas aplicações relacionadas, mas que devem ser documentadas separadamente:

1. **Trabalho da disciplina de Redes Neurais:** pipeline científico de dados históricos da ANA, análise exploratória, construção de variáveis temporais, comparação de modelos e avaliação objetiva de capacidade de generalização.
2. **PI5:** aproveitar o aprendizado sobre variáveis relevantes (por exemplo, precipitação acumulada, sequência de dias chuvosos, níveis anteriores) para subsidiar, no futuro, um módulo de previsão de tendência ou risco. **Este estudo não equivale à implementação nem à validação de um modelo embarcado no FPGA.**

A abordagem inicial será de **séries temporais diárias**. O estudo deve contemplar tanto **níveis elevados e cheias** quanto **níveis baixos e estiagens**, sem presumir que um nível de rio equivale diretamente ao volume útil de um reservatório ou ao risco de racionamento de água.

### Entregas previstas

- Notebook Python (preferencialmente Google Colab) com explicações em Markdown e células executáveis.
- Base diária limpa e auditável, em CSV ou Parquet, mantendo informações de origem e qualidade.
- Relatório de cobertura temporal e qualidade das três séries.
- Gráficos de precipitação, cota, vazão e episódios representativos.
- Indicadores de chuva acumulada, persistência de chuva/estiagem e variação do nível.
- Baselines e ao menos uma rede neural treinada, **somente após validar as séries**.
- Métricas e gráficos em conjunto de teste cronologicamente posterior ao treino.
- Discussão explícita das limitações dos dados e do alcance real das conclusões.

---

## 2. Motivação e problema de pesquisa

O PI5 procura monitorar o nível de água por sensores locais e combinar informações do ambiente para apoiar uma avaliação de tendência. Uma das hipóteses é que **a quantidade de precipitação e sua distribuição nos dias antecedentes contribuem para explicar variações subsequentes do nível do rio**. É necessário testar essa hipótese com observações históricas: o efeito depende da bacia, das condições antecedentes, das contribuições a montante e de outros fatores não representados por uma única estação de chuva.

Os dois eixos científicos são:

- **Cheias:** investigar chuvas intensas ou persistentes que antecedem subidas rápidas de cota, valores de cota elevados e picos de vazão.
- **Estiagens:** investigar sequências sem chuva e chuva acumulada insuficiente, correlacionando-as com quedas de cota e períodos de baixa vazão.

**Pergunta central sugerida:**

> Em que medida dados históricos diários de precipitação e do próprio estado do rio permitem estimar o nível ou a vazão futura em Encantado e identificar padrões antecedentes a extremos hidrológicos?

**Perguntas secundárias:**

1. Quanto choveu nos **1, 3, 5, 7, 15 e 30 dias** anteriores aos maiores picos de cota observados?
2. Quantos dias com precipitação ocorreram antes desses picos? Foram dias consecutivos ou eventos separados?
3. Qual o intervalo típico, e qual sua variabilidade, entre eventos chuvosos e elevação expressiva da cota?
4. A precipitação acumulada explica o comportamento melhor do que a chuva de um único dia?
5. A cota dos dias anteriores melhora a previsão quando adicionada às variáveis de chuva?
6. Existem associações entre baixa precipitação acumulada em **15, 30 e 60 dias** e períodos de vazão baixa?
7. Qual horizonte tem utilidade maior: prever o nível do **próximo dia**, de **3 dias à frente** ou a **tendência de subida/descida**?
8. Qual o ganho de uma rede neural em relação a previsões simples, como repetir o nível observado no dia anterior?
9. Como a confiabilidade muda em episódios raros e extremos, em comparação com condições normais?

Essas janelas e horizontes são **sugestões de hipóteses**, não valores cientificamente validados a priori.

---

## 3. Bases de dados recebidas e papel de cada uma

### 3.1 Fonte e localização dos dados

Os arquivos recebidos são exportações do **Sistema HidroWeb / ANA**. A organização é **por estação de monitoramento**, não um arquivo único contendo todo o Rio Grande do Sul. As observações refletem um ponto de monitoramento e devem ser analisadas com atenção à posição relativa das estações na bacia.

| Arquivo original | Código de estação | Tipo de série | Uso proposto |
|---|---|---|---|
| `86720000_Cotas.csv` | `86720000` | Níveis/cotas do rio | Alvo principal da primeira versão; estudo de altas e baixas cotas |
| `86720000_Vazoes.csv` | `86720000` | Vazões | Alvo alternativo e indicador complementar de cheias/estiagens |
| `2951010_Chuvas.csv` | `2951010` | Precipitação | Principal variável meteorológica histórica de entrada |
| `86720000_QualAgua.csv` | `86720000` | Qualidade da água | Base secundária; não integrar ao primeiro modelo |

**Observação de interpretação:** o código e o nome da estação não garantem que a estação de chuva seja representativa da precipitação em toda a área contribuinte da seção do rio. Recomenda-se, em fase posterior, investigar pluviômetros a montante e características da bacia.

### 3.2 Diagnóstico técnico efetivamente verificado nos arquivos enviados

Os números abaixo foram obtidos por inspeção programática dos CSVs fornecidos, com identificação do cabeçalho real e das colunas diárias. **Linhas do CSV são em geral registros mensais e não dias individuais.**

| Arquivo | Linhas de dados no CSV | Colunas | Primeiro mês registrado | Último mês registrado | Níveis de consistência registrados |
|---|---:|---:|---|---|---|
| Cotas | 2.087 | 78 | out/1941 | abr/2026 | Bruto (`1`) e consistido (`2`) |
| Vazões | 1.012 | 78 | out/1941 | abr/2026 | Bruto (`1`) e consistido (`2`) |
| Chuvas | 1.701 | 75 | abr/1943 | abr/2026 | Bruto (`1`) e consistido (`2`) |
| Qualidade da água | 51 | 157 | ago/2002 | set/2022 | Bruto (`1`) |

**Diagnóstico preliminar da expansão diária**, com a regra exploratória “priorizar o registro consistido do mesmo dia, quando presente, e selecionar apenas valores com `Status = 1` (Real)”:

| Série | Datas com valor após preferência por consistido | Datas classificadas como “Real” após essa seleção |
|---|---:|---:|
| Cotas | 24.731 | 20.177 |
| Vazões | 24.619 | 20.103 |
| Chuvas | 28.884 | 28.805 |

- **Interseção preliminar:** 18.531 datas aparecem simultaneamente nas três séries após esses critérios.
- **Intervalo extremo dessa interseção:** 19/04/1943 a 30/04/2026.
- **Atenção:** os 18.531 dias **não formam necessariamente uma série diária sem lacunas**. A contagem **não** equivale ao volume final de exemplos treináveis; janelas temporais completas reduzirão esse número.
- A regra acima é **uma primeira aproximação de qualidade**, a validar com a semântica oficial de `NivelConsistencia`, `Status`, métodos de medição, dias com medições múltiplas e eventuais alterações históricas na estação.
- Detectaram-se valores negativos de cota, que **não devem ser excluídos automaticamente**: podem corresponder a leituras relativas ao zero de referência da estação; verificar antes de tratar como erro.
- Essas contagens não demonstram, por si só, que as medições diárias possuem exatamente a mesma convenção de horário ou período hidrológico.

**Atenção ao alcance:** números de linhas, datas e datas válidas foram verificados diretamente nos arquivos; padrões de chuva/cheia, métricas de previsão e frequência de eventos ainda **não** foram analisados nem calculados.

### 3.3 Cotas — `86720000_Cotas.csv`

**Finalidade:** representar a altura relativa da superfície da água na seção de monitoramento. É a melhor candidata para o **alvo da primeira tarefa de regressão**.

**Campos identificados:**

- `EstacaoCodigo`: identificador da estação.
- `NivelConsistencia`: `1` bruto; `2` consistido, conforme o cabeçalho do próprio CSV.
- `Data`: data de referência do registro mensal.
- `hora` / `MediaDiaria` / `TipoMedicaoCotas`: metadados sobre horário, agregação e instrumento/método.
- `Cota01` ... `Cota31`: valores para os dias 1 a 31 do respectivo mês.
- `Cota01Status` ... `Cota31Status`: situação individual dos valores.
- `Maxima`, `Minima`, `Media` e outras estatísticas do período.

**Questão da unidade:** em bases fluviométricas da ANA, cotas frequentemente são expressas em centímetros relativos à referência da régua, mas **deve-se confirmar a convenção/metadados específicos da série** antes de rotular gráficos ou converter para metros. Uma cota alta não significa automaticamente transbordamento; é necessário um limiar local ou registro de evento para interpretar criticidade.

### 3.4 Vazões — `86720000_Vazoes.csv`

**Finalidade:** representar o volume de água que passa pela seção do rio por unidade de tempo, geralmente em m³/s, condicionada à confirmação da unidade de exportação. É útil para complementar análise de cheias, caracterizar estiagens e comparar modelagem alternativa.

**Campos identificados:**

- `EstacaoCodigo`, `NivelConsistencia`, `Data`, `Hora`, `MediaDiaria`, `MetodoObtencaoVazoes`.
- `Vazao01` ... `Vazao31` e respectivas colunas `Status`.
- `Maxima`, `Minima`, `Media` e estatísticas auxiliares.

**Cuidado metodológico:** vazão pode ser **derivada da cota por curva de descarga** ou por outro procedimento. Portanto, uma forte associação estatística entre cota e vazão pode ser parcialmente estrutural, não evidência de duas medições independentes. Investigar o método disponível e evitar conclusões causais precipitadas.

### 3.5 Chuvas — `2951010_Chuvas.csv`

**Finalidade:** medir a precipitação histórica observada e elaborar variáveis como acumulados, duração de períodos chuvosos e ausência prolongada de chuva.

**Campos identificados:**

- `EstacaoCodigo`, `NivelConsistencia`, `Data`, `TipoMedicaoChuvas`.
- `Chuva01` ... `Chuva31`: precipitação para cada dia do mês, usualmente em mm.
- `Chuva01Status` ... `Chuva31Status`: marcação de qualidade do valor diário.
- `Total`, `Maxima`, `NumDiasDeChuva`, `TotalAnual` e campos de situação da estatística.

**Importante:** zero (`0,0`) é uma observação potencialmente válida de ausência de chuva; **não** converter zero para valor ausente. Já campos vazios ou registros marcados como duvidosos precisam de tratamento e rastreabilidade. A definição de “dia de chuva” deve ser explicitada; por exemplo, limiar de 0,1 mm ou 1,0 mm, justificado e explorado em análise de sensibilidade.

### 3.6 Qualidade da água — `86720000_QualAgua.csv` (escopo secundário)

Essa base contém observações esparsas sobre parâmetros como `TempAmostra`, `pH`, `Turbidez`, `OD`, `CondutividadeEletrica` e `Choveu`. São 51 linhas de amostragem no arquivo examinado, cobrindo datas extremas entre 2002 e 2022. O campo `Choveu` indica uma ocorrência binária de chuva na amostragem, **não** a precipitação diária em mm.

**Decisão:** manter a base documentada, mas **excluí-la da primeira fase de treino**, porque a frequência de coleta e o objetivo de qualidade da água diferem do problema de previsão diária de cota/vazão. Poderá sustentar outra pesquisa no futuro.

---

## 4. Como interpretar os CSVs sem introduzir erros

A formatação das exportações da ANA requer tratamento específico:

1. **Cabeçalho textual inicial:** várias linhas de identificação institucional, legenda e filtros precedem a linha que começa com `EstacaoCodigo;`. Localizar o início do cabeçalho dinamicamente, em vez de presumir sempre o mesmo número de linhas a descartar.
2. **Codificação:** os arquivos fornecidos puderam ser lidos em `cp1252`; definir a codificação ao chamar `pandas.read_csv`.
3. **Separador:** `;`.
4. **Decimal:** vírgula, por exemplo `95,5382`.
5. **Formato mensal em colunas diárias:** a linha contém `Data` referente ao mês, e os campos `Chuva01` etc. representam dias específicos; é preciso converter o formato *wide* em *long*.
6. **Dias inexistentes:** ignorar `31` em abril, `30` em fevereiro etc. Gerar a data com validação do calendário.
7. **Duplicidade entre bruto e consistido:** pode haver dois registros referentes ao mesmo mês/dia. Não somá-los nem tratá-los como observações independentes.
8. **Qualidade diária:** valores da série podem ter `Status` que diferem da classificação da linha. Manter coluna de status após a expansão.
9. **Valores faltantes:** preservar como `NaN` e documentar sua causa quando identificável. Nunca transformar falta de chuva observada em zero artificial.
10. **Horário de observação:** confirmar, conforme metadados, se os valores de precipitação representam chuva acumulada em um dia civil ou em janela de observação deslocada. Isso afeta o alinhamento chuva → nível.
11. **Mudanças de instrumento ou procedimentos:** investigar se alteram a comparabilidade de partes da série histórica.
12. **Controle de proveniência:** guardar arquivo original, código de estação, `NivelConsistencia`, status, coluna de origem e regra de desempate.

**Regra candidata para duplicações:** priorizar o valor consistido (`2`) **quando válido**; se não for válido, avaliar de forma explícita se é aceitável utilizar o bruto (`1`) com qualidade `Status = 1`. A regra deve ser escrita, testada e auditada; o diagnóstico preliminar da seção 3 priorizou o registro consistido antes de filtrar o status, portanto **não** realizou automaticamente esse fallback.

---

## 5. Objetivos e recorte inicial da disciplina de Redes Neurais

### 5.1 Objetivo geral

Investigar a capacidade de modelos supervisionados de redes neurais de **estimar a cota do Rio Taquari em Encantado para dias futuros**, utilizando cota histórica e precipitação observada em dias anteriores, com vazão como variável adicional ou tarefa alternativa.

### 5.2 Objetivos específicos

1. Caracterizar a cobertura e a integridade das séries históricas ANA.
2. Construir uma base diária integrada, sem duplicidade nem datas inválidas.
3. Quantificar episódios de alta e baixa cota/vazão, com definições operacionais explícitas.
4. Explorar chuva acumulada, frequência e persistência em diferentes janelas antecedentes.
5. Construir experimentos supervisionados de previsão de cota do dia seguinte.
6. Comparar uma rede neural simples a modelos ingênuos e regressões tradicionais.
7. Avaliar desempenho em datas futuras não usadas no treinamento.
8. Examinar erros em situações extremas, e não somente o erro médio geral.
9. Documentar limites científicos e possibilidades de extensão ao PI5.

### 5.3 Escopo mínimo viável recomendado

Começar por **regressão para prever a cota de amanhã** (`cota_t+1`) com dados diários disponíveis até o dia `t`. É um problema claro, avaliável e adequado para um trabalho acadêmico de rede neural. **Não iniciar simultaneamente com LSTM, previsão horária, classificação de enchentes e implantação no FPGA.** Essas são extensões possíveis depois de termos uma base e um benchmark confiáveis.

**Questão fundamental de disponibilidade temporal:** se a predição for feita **ao final do dia t**, é razoável usar chuva e cota do próprio dia `t` apenas se de fato já estiverem disponíveis naquele horário. Caso contrário, usar somente informações até `t-1`. Descrever isso claramente para evitar vazamento temporal.

---

## 6. Perguntas e hipóteses investigativas para cheias e estiagens

### 6.1 Cheias / níveis elevados

- Maior acumulado em 3 dias precede, em média, maior elevação de cota nos dias seguintes?
- Dias consecutivos de chuva se associam a picos diferentes dos causados por uma única chuva intensa?
- Qual atraso (`lag`) entre chuva e mudança de cota apresenta maior associação nos dados?
- Quais máximos e taxas de subida aparecem em eventos históricos específicos?
- O efeito é mais forte quando o nível já estava elevado antes do episódio chuvoso?

### 6.2 Níveis baixos / estiagens

- Quantos dias sem precipitação relevante precedem mínimos de cota e vazão?
- A chuva acumulada em 15, 30 ou 60 dias é informativa para antever queda sustentada da vazão?
- A persistência dos níveis baixos é distinta de quedas breves e isoladas?
- A sazonalidade altera o significado de uma mesma cota ou vazão baixa?

### 6.3 Definições de evento

**Não escolher um limite em centímetros ou em m³/s arbitrariamente como “enchente” ou “racionamento”.** Para um primeiro estudo exploratório, usar quantis da série de treino (por exemplo, acima do percentil 95 ou abaixo do percentil 5) como **indicadores estatísticos de extremos**, deixando claro que **não equivalem a limiares oficiais de inundação/abastecimento**. Se houver dados de cotas de alerta, inundação ou registros históricos confiáveis de ocorrência, estes podem definir eventos com maior validade prática.

Agrupar dias consecutivos acima/abaixo do limite em **episódios**, para não contar cada dia de um mesmo evento como uma enchente independente. Separar episódios por um intervalo mínimo a ser definido e documentado.

---

## 7. Construção da base analítica diária

### 7.1 Esquema de saída desejado

Exemplo de colunas da tabela diária integrada:

| Campo | Significado |
|---|---|
| `data` | Data da observação em calendário explícito |
| `cota` | Cota validada da estação 86720000 |
| `vazao_m3s` | Vazão validada da estação 86720000 |
| `chuva_mm` | Precipitação validada da estação 2951010 |
| `status_cota`, `status_vazao`, `status_chuva` | Qualidade/proveniência de cada valor |
| `consistencia_*` | Bruto ou consistido da observação selecionada |
| `fonte_*` | Arquivo e estação de origem |

Um exemplo **fictício apenas de estrutura**:

```csv
data,cota,vazao_m3s,chuva_mm
2020-01-01,125,160.2,0.0
2020-01-02,128,165.5,17.4
2020-01-03,140,193.1,31.2
```

**Nunca apresentar esses números de exemplo como dados reais da ANA.**

### 7.2 Sequência de preparação

1. Ler e preservar os CSVs originais sem sobrescrever.
2. Localizar e interpretar cabeçalhos e legendas.
3. Expandir observações de dias 01–31 em linhas com datas reais.
4. Converter decimais e tipos; registrar impossibilidade de conversão.
5. Definir e aplicar estratégia de consistência/status.
6. Resolver duplicatas por **data, estação, variável e eventual horário**, com auditoria de conflito.
7. Organizar calendário diário e marcar lacunas explicitamente.
8. Unir precipitação, cota e vazão pela data; inspecionar possíveis deslocamentos de horário de observação.
9. Avaliar intervalos contínuos úteis para janelas temporais.
10. Criar tabela *limpa* e relatório quantitativo da exclusão de observações.

### 7.3 Relatório de qualidade recomendado

Gerar estatísticas como: datas extremas; dias esperados no calendário; dias com medição por variável; percentual de cobertura anual; sobreposição entre séries; duplicatas e conflitos; proporção por status; maior lacuna; distribuição de valores; mudanças abruptas; cobertura nos períodos de interesse; evolução de métodos e consistência. Produzir **mapa de calor de disponibilidade por ano/mês**.

Não interpolar indiscriminadamente níveis e vazões em grandes lacunas. Se algum preenchimento for imprescindível, documentar método, amplitude máxima permitida, impacto na avaliação e se o preenchimento usa informações futuras (o que não deve ocorrer nas entradas de previsão).

---

## 8. Análise exploratória (EDA) antes da modelagem

### 8.1 Visualizações prioritárias

- Série temporal completa de **cota** e série temporal completa de **vazão**, com indicação de lacunas.
- Série de **precipitação diária** e de **acumulados móveis**.
- Histograma, boxplot e quantis de cada variável; comparações por estação do ano.
- Chuva em barras e cota em linha **em eixos próprios, com escalas e unidades explícitas**.
- Ampliação temporal de **eventos selecionados**, com dados anteriores e posteriores ao pico.
- Gráfico de dispersão da chuva acumulada antecedente vs. variação de cota futura.
- Correlações defasadas (*cross-correlation*), sempre interpretadas com cautela pela autocorrelação e sazonalidade.
- Comparação de nível/vazão em períodos de chuva persistente e períodos secos.
- Mapa de ausência de dados por ano/mês.

### 8.2 Indicadores a calcular

**Para chuva:** total diário, acumulados prévios de 3/5/7/15/30/60 dias, número de dias com chuva na janela, maior chuva diária na janela, sequência corrente de dias úmidos/secos e dias desde a última chuva relevante.

**Para o rio:** cota corrente, cotas defasadas, variação em 1/3/7 dias, máximo e mínimo móvel histórico, vazão corrente e defasada, duração de episódios extremos, tempo até pico após evento de precipitação.

**Para episódios:** data de início, data de pico/mínimo, duração, chuva acumulada anterior, maior chuva antecedente, variação total de cota, vazão correspondente, completude dos dados e incerteza do evento.

**Não calcular automaticamente acumulados atravessando lacunas** como se dados ausentes representassem zero. Exigir a presença de todas as observações necessárias, ou explicitar um critério mínimo de completude.

---

## 9. Formulação supervisionada para a rede neural

### 9.1 Unidade de observação e horizonte

Cada exemplo corresponde a uma data `t` com histórico disponível até `t`, e alvo em `t+1` (primeira versão). A dependência temporal exige **separação cronológica** entre treino, validação e teste.

### 9.2 Entradas candidatas (*features*)

**Grupo A — histórico do rio:** `cota_t`, `cota_t-1`, `cota_t-2`, `cota_t-3`, `cota_t-7`, mudanças recentes de cota; opcionalmente vazão recente.

**Grupo B — precipitação histórica:** `chuva_t`, `chuva_t-1`, `chuva_t-2`, `chuva_t-3`, acumulados antecedentes de 3/5/7/15/30 dias, número de dias chuvosos, maior chuva antecedente e persistência de dias secos.

**Grupo C — contexto opcional:** mês do ano codificado de modo cíclico (`sen(mês)`, `cos(mês)`), características sazonais e, numa extensão futura, observações de outras estações da bacia.

**Proposta inicial de baixo custo:** cota dos últimos 7 dias, chuva dos últimos 7 dias e dois acumulados antecedentes (3 e 7 dias). Testar ablação: cota sozinha; cota + chuva; cota + chuva + vazão. O resultado revelará se a adição de precipitação de fato melhora a previsão nessa base.

### 9.3 Alvo (*target*)

**Alvo principal (regressão):** `cota_(t+1)`, na unidade original da série após validação. Também avaliar `delta_cota_(t+1) = cota_(t+1) - cota_t`, pois a previsão da mudança pode tornar mais clara a capacidade de identificar subida/descida. Comparar ambas, se houver tempo.

**Alvo secundário eventual:** `vazao_(t+1)`.

**Classificação futura (opcional):** indicadores de condição estatisticamente alta/baixa ou direção do nível. Exige cuidado com desbalanceamento e critérios de evento. Não confundir classe estatística com alerta operacional real.

### 9.4 Exemplo conceitual de uma amostra

```text
Data de referência t: D7
Entradas:
  cotas de D1..D7
  chuvas de D1..D7
  chuva acumulada de D5..D7
  chuva acumulada de D1..D7
Alvo:
  cota observada em D8
```

Todos os valores precisam existir e ser válidos nas datas relevantes. A disponibilidade da chuva observada de `D7` depende do instante operacional da previsão (ver seção 5.3).

### 9.5 Quando considerar previsão de chuva futura

No PI5, existe uma segunda fonte: **Open-Meteo**, que retorna previsões horárias de temperatura e precipitação (incluindo probabilidade de precipitação e códigos do tempo) consultadas por `GET` com latitude/longitude. **Esse conteúdo é previsão futura, não o histórico observado da ANA.**

Não misturar retrospectivamente as chuvas reais dos próximos dias com entradas de um modelo que pretende prever o futuro: isso seria **vazamento de informação**. Um modelo que vá consumir previsão Open-Meteo precisa ser testado com **previsões meteorológicas históricas emitidas antes da data prevista** (arquivos de previsões passadas / *hindcasts*, quando disponíveis), não com observações realizadas depois da previsão. A versão inicial da disciplina pode usar apenas **histórico observado até `t`**, evitando essa dificuldade.

---

## 10. Plano experimental de redes neurais

### 10.1 Separação temporal dos dados

Não usar `train_test_split` aleatório sobre amostras diárias da mesma série. A sobreposição entre janelas próximas gera similaridade enorme entre exemplos e pode inflar métricas.

**Proposta:** separar os dados cronologicamente em blocos de treino, validação e teste, respeitando períodos com cobertura suficiente. Uma opção ilustrativa é 70% / 15% / 15% em ordem temporal; ajustar datas concretas após inspeção do histórico. Conferir eventos de cheia e estiagem em cada bloco e não escolher splits somente para favorecer o modelo.

- Ajustar `StandardScaler` ou `MinMaxScaler` **somente no treino**; transformar validação e teste com os mesmos parâmetros.
- Se o alvo for escalado, ajustar o escalador do alvo apenas com dados do treino e inverter escala para métricas interpretáveis.
- Evitar janelas de entrada que atravessam lacunas ou que recebam rótulos de períodos não autorizados no treino.
- Para avaliação robusta, considerar *walk-forward validation* ou múltiplos recortes cronológicos com calibração exclusivamente no passado.
- Se necessário, usar uma janela de embargo perto das fronteiras de split para controlar sobreposição indevida de exemplos, de acordo com o horizonte alvo.

### 10.2 Baselines obrigatórios

1. **Persistência:** prever `cota_(t+1) = cota_t`.
2. **Média móvel ou extrapolação simples:** alternativa temporal simples, sem rede neural.
3. **Regressão linear ou Ridge:** benchmark interpretável com as mesmas features.

Uma rede neural deve superar esses referenciais em teste, ou a conclusão acadêmica precisa reconhecer que não demonstrou ganho para o problema/configuração escolhidos.

### 10.3 Rede neural inicial sugerida

- Tipo: **MLP (perceptron multicamadas)** para regressão com janelas temporais transformadas em vetor.
- Entrada: número de neurônios igual ao número de features escolhidas.
- Arquitetura inicial ilustrativa: `Dense(64, relu) → Dense(32, relu) → Dense(1)`.
- Perda: MAE, MSE ou Huber, conforme diagnóstico de extremos/outliers.
- Otimizador: Adam; usar early stopping monitorando a validação.
- Regularização opcional: L2 ou dropout, somente se houver evidência de sobreajuste.
- Semente aleatória fixa e documentação de versão das bibliotecas e hiperparâmetros.

**Alternativas futuras:** LSTM ou GRU para sequências, e CNN temporal, se houver tempo e tamanho de amostra adequados. **Não presumir que arquiteturas mais complexas serão melhores.** Para a disciplina, uma MLP bem avaliada pode ser mais sólida metodologicamente do que uma LSTM sem comparação adequada.

### 10.4 Experimentos e ablações recomendados

| Experimento | Entradas | Pergunta |
|---|---|---|
| E0 — Persistência | Cota atual | Qual o mínimo a superar? |
| E1 — Linear | Cotas passadas | Modelo simples já explica muito? |
| E2 — MLP só nível | Cotas passadas | A rede aprende padrões temporais do nível? |
| E3 — MLP nível + chuva | Cotas e chuva histórica | A precipitação adiciona informação? |
| E4 — MLP nível + chuva + vazão | Três séries | A vazão melhora a previsão? |
| E5 — Horizonte estendido | Mesmo conjunto, alvo em t+3 | Qual a perda de precisão com horizonte maior? |

**Recomendação:** começar com E0–E3. Incluir E4/E5 apenas se o pipeline estiver correto e o tempo permitir.

---

## 11. Avaliação de desempenho

### 11.1 Métricas para regressão

- **MAE**: erro absoluto médio, na unidade da cota.
- **RMSE**: destaca erros maiores; importante para avaliar erros em picos.
- **R²**: explicação relativa de variância, interpretada com cautela em série temporal.
- **Erro por faixa de nível**: avaliar separadamente regiões comuns, altas e baixas.
- **Erro por horizonte**: caso se avaliem `t+1`, `t+3` ou outros prazos.

Evitar MAPE como métrica principal para cotas que possam estar próximas de zero ou negativas devido à referência local.

### 11.2 Avaliar os extremos separadamente

A maioria dos dias pode estar em condições usuais. Um modelo que prevê muito bem nesses dias pode falhar justamente nos raros eventos de interesse. Recomenda-se:

- Calcular MAE/RMSE em dias de níveis estatisticamente altos e baixos, definidos com referência **apenas ao treino**.
- Mostrar observações e previsões em janelas de eventos de pico e de baixa vazão/cota.
- Examinar tendência de subestimação dos máximos e superestimação dos mínimos.
- Se a tarefa evoluir para classificação, reportar precisão, recall, F1, matriz de confusão e PR-AUC quando aplicável, especialmente para a classe rara.
- Não calibrar limiar de evento a partir das datas de teste.

### 11.3 Gráficos dos resultados

- Curvas de treino e validação por época.
- Cota observada vs. cota prevista ao longo de trecho do teste.
- Dispersão observado × previsto com linha de igualdade.
- Histograma dos resíduos e resíduos por faixa de nível.
- Comparativo de baselines e redes em métricas de teste.
- Exemplos de falhas importantes em períodos críticos.

---

## 12. Limitações científicas e cuidados de interpretação

1. **Rio vs. reservatório:** a cota do Rio Taquari na estação Encantado não corresponde à porcentagem de armazenamento de uma barragem. Não converter a cota em “% da capacidade” sem curva cota–volume e referência específica do reservatório.
2. **Cota de inundação:** valores altos não significam automaticamente ocorrência de inundação; obter referência oficial local para uma classificação operacional.
3. **Racionamento:** baixos níveis de rio/vazão são sinais hidrológicos relevantes, mas não suficientes para afirmar racionamento; decisões dependem de captação, armazenamento, consumo, operação e políticas locais.
4. **Chuva em um ponto:** o pluviômetro 2951010 não mede a distribuição espacial das chuvas por toda a bacia. Eventos concentrados a montante podem gerar subidas de nível com pouca chuva na estação local.
5. **Relação temporal:** medir correlação chuva–cota não comprova causalidade isoladamente; considerar sazonalidade, vazões de tributários, evapotranspiração, solo, intervenções humanas e possíveis mudanças de regime.
6. **Dados consistidos:** consistência é atributo de qualidade, não garantia absoluta de ausência de erro.
7. **Amostras dependentes:** dados diários sucessivos são autocorrelacionados; não inferir número de observações independentes a partir do total de dias.
8. **Mudanças históricas:** grandes séries podem incluir diferenças de equipamento, métodos, zero de régua e condições geomorfológicas; documentar informações conhecidas.
9. **Medições derivadas:** vazão pode ser calculada a partir da cota, reduzindo a independência das fontes.
10. **Não extrapolar:** resultados obtidos em Encantado não são automaticamente transferíveis para qualquer barragem, estação ou região.
11. **Diferença de resolução:** ANA aqui fornece observações históricas diárias; Open-Meteo fornece previsões horárias; hardware do PI5 pode amostrar com resolução de minutos. O alinhamento entre fontes deve ser definido separadamente, sem inventar precisão temporal.
12. **Operação em tempo real:** bom desempenho retrospectivo não valida, por si só, um sistema real de alerta ou um modelo embarcado.

---

## 13. Estrutura sugerida do notebook Google Colab

**Nome proposto:** `Analise_ANA_Encantado_Redes_Neurais_PI5.ipynb`

| Parte | Seções/células principais | Saída esperada |
|---|---|---|
| 0 | Capa, objetivos, fontes, escopo e limitações | Notebook autoexplicativo |
| 1 | Importações, versões, sementes e caminhos | Ambiente reprodutível |
| 2 | Upload/leitura dos CSVs com cabeçalho variável | DataFrames brutos |
| 3 | Validação de esquema, unidades e tipos | Diagnóstico de entrada |
| 4 | Conversão wide→long, controle de status e consistência | Três séries diárias |
| 5 | Auditoria de duplicatas e cobertura | Tabelas e gráficos de qualidade |
| 6 | Alinhamento temporal e junção por data | Base diária integrada |
| 7 | EDA: chuva, cota, vazão e eventos | Gráficos e conclusões preliminares |
| 8 | Engenharia de variáveis e alvo t+1 | Dataset supervisionado |
| 9 | Separação cronológica e escalonamento | Treino/validação/teste |
| 10 | Persistência e regressão linear | Baselines |
| 11 | MLP e acompanhamento do treinamento | Modelo treinado |
| 12 | Métricas gerais e análise de extremos | Avaliação objetiva |
| 13 | Ablação de features e discussão | Resultados comparativos |
| 14 | Limitações e trabalhos futuros | Conclusão acadêmica |
| 15 | Exportação do dataset e resultados | Artefatos reprodutíveis |

**Bibliotecas indicadas:** `pandas`, `numpy`, `matplotlib`, `scikit-learn` e `tensorflow`/`keras` ou `pytorch` (escolher **uma** abordagem principal para evitar complexidade desnecessária). `seaborn` é opcional, não obrigatório. `scipy` pode auxiliar análises estatísticas.

**Produtos de exportação sugeridos:**

```text
projeto_ana_redes_neurais/
├── dados_originais/
│   ├── 86720000_Cotas.csv
│   ├── 86720000_Vazoes.csv
│   └── 2951010_Chuvas.csv
├── notebooks/
│   └── Analise_ANA_Encantado_Redes_Neurais_PI5.ipynb
├── resultados/
│   ├── ana_encantado_diario_limpo.csv
│   ├── dataset_supervisionado.csv
│   ├── metricas_modelos.csv
│   └── figuras/
├── modelos/
│   └── modelo_mlp.*
└── README.md
```

Os nomes são ilustrativos. Não criar nem alegar que já existem arquivos de resultados ou pesos de modelos antes de executar os experimentos.

---

## 14. Plano de execução por fases

### Fase A — Curadoria e EDA (prioridade máxima)

- Ler três CSVs e descrever as estações, fontes e esquema.
- Transformar dados mensais em séries diárias.
- Resolver bruto/consistido e inspecionar status.
- Quantificar disponibilidade e lacunas.
- Produzir gráficos históricos e identificar episódios candidatos.
- **Critério de conclusão:** três séries diárias confiáveis, com regra de limpeza documentada e base integrada reprodutível.

### Fase B — Formulação de predição

- Escolher alvo principal `cota_(t+1)`.
- Fixar horário lógico da previsão, conjunto inicial de features e critério para janelas completas.
- Separar os intervalos cronológicos de treino, validação e teste.
- **Critério de conclusão:** exemplos supervisionados sem vazamento e com quantidade/cobertura de dados suficiente.

### Fase C — Baselines e rede neural

- Medir desempenho de persistência e regressão linear.
- Treinar MLP simples.
- Analisar curvas de aprendizado e regularização.
- Comparar modelos e grupos de entradas com o mesmo teste.
- **Critério de conclusão:** métricas calculadas e resultados reprodutíveis, independentemente de a rede superar ou não os baselines.

### Fase D — Análise científica e relatório final

- Descrever padrões de chuva anteriores a picos e mínimos observados.
- Quantificar erros em extremos e discutir limitações.
- Consolidar figuras, tabelas, parâmetros e conclusões.
- **Critério de conclusão:** relatório que diferencia claramente *observações*, *associações*, *previsões* e *hipóteses ainda não confirmadas*.

### Fase E — Extensões opcionais

- Comparação com LSTM/GRU.
- Predição multi-horizonte.
- Inclusão de outras estações pluviométricas a montante.
- Variáveis hidrometeorológicas adicionais ou dados de reservatórios, quando relevantes.
- Estudo futuro de integração com o backend do PI5; FPGA apenas após definição de requisitos de entrada, capacidade e critérios de validação.

---

## 15. Relação com o restante do PI5 (sem ampliar o escopo da disciplina)

### 15.1 O que já está definido no projeto integrador

- O hardware local com **Arduino Mega 2560**, HC-SR04, DHT, RTC, buzzer e relé foi validado em bancada segundo o histórico do projeto.
- A comunicação via **ESP12F ainda está pendente de funcionamento/validação**, portanto não tratar envio em rede como pronto.
- O backend teve estrutura inicial criada e sua arquitetura de persistência foi redirecionada para **MongoDB**; a implementação das coleções e integrações permanece trabalho futuro.
- Uma rotina Python já conseguiu executar `GET` à **Open-Meteo**, selecionando a localização por latitude/longitude, usando `forecast_days=3` e obtendo **72 previsões horárias** de cinco variáveis: `temperature_2m`, `precipitation`, `precipitation_probability`, `rain`, `weather_code`.
- A rotina da Open-Meteo processa os arrays e mostra registros no terminal; **isso não significa persistência no MongoDB já concluída**.

### 15.2 O que o presente estudo poderá contribuir

- Identificar quais características do histórico de precipitação e nível oferecem poder preditivo.
- Estimar horizonte e grau de incerteza para previsões curtas.
- Produzir uma referência metodológica para conectar fontes meteorológicas e hidrológicas.
- Sustentar a escolha de features e estratégias de normalização futuras.

### 15.3 O que o estudo **não** entrega automaticamente

- Um modelo validado para qualquer reservatório.
- Limiares oficiais de transbordamento ou racionamento.
- Um sistema de alerta operacional.
- Um modelo FPGA pronto para implantação.
- Integração pronta de leituras Arduino, dados meteorológicos e dados históricos.

---

## 16. Decisões pendentes para iniciar no outro chat

As decisões abaixo podem ser tomadas **após** a inspeção e EDA dos CSVs; não precisam bloquear o início da curadoria:

1. **Confirmação da unidade e referência da cota** na estação 86720000.
2. **Regra final de qualidade e desempate** entre observações brutas e consistidas, incluindo status estimado/duvidoso.
3. **Convenção temporal** da observação da chuva e alinhamento com as cotas.
4. **Recorte de anos** com cobertura suficientemente contínua em todas as variáveis.
5. **Variável-alvo inicial**: recomendada cota do dia seguinte; vazão como comparação posterior.
6. **Limiar de “dia chuvoso”** e janelas temporais a comparar.
7. **Critério de evento extremo**, estatístico para EDA ou oficial quando possível.
8. **Arquitetura neural**: recomendada MLP na primeira entrega; extensões somente quando justificadas.
9. **Critérios da disciplina**: tempo de apresentação, métricas exigidas, tecnologias aceitas, número de experimentos e formato do relatório.

---

## 17. Instrução pronta para reutilizar em outro chat

Copie a mensagem abaixo junto com este relatório e anexe os três CSVs originais:

> Estou desenvolvendo um projeto para a disciplina de Redes Neurais, com possível aproveitamento futuro no PI5 (monitoramento hidrológico). Tenho três arquivos exportados do HidroWeb/ANA: `86720000_Cotas.csv` (cotas em Encantado), `86720000_Vazoes.csv` (vazões na mesma estação) e `2951010_Chuvas.csv` (precipitação da estação 2951010). Quero trabalhar em **Google Colab, passo a passo**, começando por **análise e limpeza das séries**, e depois treinar uma rede neural para **prever a cota do próximo dia** a partir das cotas e chuvas anteriores. Leia primeiro os arquivos reais, confirme formato, cobertura, status e duplicidades, não invente resultados nem períodos completos. Quero avaliar também padrões antecedentes a níveis elevados e baixos, sem afirmar que cota de rio representa capacidade de reservatório ou racionamento. Use divisão temporal treino/validação/teste, normalização ajustada somente no treino e comparação com baselines, principalmente persistência. Mostre e explique cada célula do notebook antes de avançar para a seguinte, começando pela importação e inspeção. O relatório Markdown anexo traz o contexto e as decisões propostas. Antes de modelar, destaque quaisquer problemas importantes de qualidade, unidade e alinhamento das séries.

---

## 18. Checklist de sucesso do trabalho acadêmico

- [ ] CSVs lidos e documentados corretamente.
- [ ] Regras de qualidade e duplicação implementadas e justificadas.
- [ ] Série diária com datas verificadas e lacunas explícitas.
- [ ] Período de sobreposição e janelas completas quantificados.
- [ ] Gráficos de chuva, cota e vazão produzidos com unidades corretas.
- [ ] Estudo exploratório dos antecedentes de extremos.
- [ ] Problema-alvo definido sem vazamento de informações futuras.
- [ ] Treino, validação e teste separados cronologicamente.
- [ ] Normalização ajustada só no treino.
- [ ] Baseline de persistência implementado.
- [ ] Ao menos uma MLP treinada e avaliada.
- [ ] Erros em situações altas e baixas discutidos.
- [ ] Experimentos, hiperparâmetros e resultados registrados.
- [ ] Conclusões distinguem achados empíricos de hipóteses.
- [ ] Limitações e possibilidades de integração futura ao PI5 documentadas.

---

## 19. Referências e proveniência

**Dados primários disponibilizados nesta conversa:**

- ANA / HidroWeb — exportação da estação 86720000: `86720000_Cotas.csv`.
- ANA / HidroWeb — exportação da estação 86720000: `86720000_Vazoes.csv`.
- ANA / HidroWeb — exportação da estação 2951010: `2951010_Chuvas.csv`.
- ANA / HidroWeb — exportação complementar da estação 86720000: `86720000_QualAgua.csv`.

**Consulta e documentação institucional:**

- ANA — Séries Históricas de Estações (HidroWeb): https://www.snirh.gov.br/hidroweb/serieshistoricas
- ANA — Portal institucional: https://www.gov.br/ana/

**Nota final:** o relatório apresenta propostas de engenharia de dados e aprendizado de máquina e um diagnóstico preliminar dos CSVs recebidos. Não foram treinadas redes neurais, não foram estimadas correlações chuva–nível nem calculadas métricas de teste nesta etapa.
