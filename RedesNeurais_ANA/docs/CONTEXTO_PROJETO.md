# Contexto do projeto — Redes Neurais com dados hidrológicos da ANA

## 1. Identificação
- **Título provisório:** Previsão da cota do Rio Taquari utilizando redes neurais artificiais e dados hidrológicos da ANA.
- **Disciplina:** Redes Neurais; professor Sandro.
- **Tarefa acadêmica:** regressão supervisionada, não classificação.
- **Ambiente:** Python, notebook Jupyter (`.ipynb`) executado no VS Code.
- **Origem dos dados:** ANA / HidroWeb, estações na região de Encantado (RS), bacia Taquari–Antas.
- **Situação:** proposta e planejamento; nenhuma métrica ou previsão foi produzida a partir deste pacote inicial.
- **Origem UCI:** professor esclareceu que UCI era uma sugestão, portanto as bases da ANA são permitidas. A exigência original menciona Google Colab; execução no VS Code utiliza o mesmo formato `.ipynb`, mas confirmar plataforma de entrega se necessário.

## 2. Questão de pesquisa
É possível prever a **cota do rio no dia seguinte** com base em cotas e precipitações dos dias anteriores? A contribuição das informações de chuva melhora uma previsão ingênua que repete a cota atual?

A resposta deverá ser **medida em dados não usados no treinamento**. Não assumir antecipadamente ganho de desempenho nem capacidade operacional de alertar cheias.

## 3. Séries previstas
| Nome esperado | Estação | Informação | Papel inicial |
| --- | --- | --- | --- |
| `86720000_Cotas.csv` | 86720000 | Cota/nível do rio | Variável-alvo `cota(t+1)` e entradas com cotas anteriores |
| `2951010_Chuvas.csv` | 2951010 | Precipitação diária | Entradas históricas e acumulados anteriores |
| `86720000_Vazoes.csv` | 86720000 | Vazão do rio | Comparação ou entrada opcional, após verificar seu caráter derivado e a cobertura |

O arquivo `86720000_QualAgua.csv` apareceu no estudo inicial, mas **não faz parte do modelo mínimo**: suas observações são mais esparsas e têm outro objetivo. Os arquivos originais devem ser colocados manualmente em `dados/` (não acompanham este pacote).

### Diagnóstico histórico preliminar — **não confundir com dados processados**
O relatório técnico original (`docs/RELATORIO_TECNICO_ORIGINAL.md`) menciona, após critérios exploratórios de consistência e status, aproximadamente **18.531 datas** compartilhadas pelas três séries, entre 19/04/1943 e 30/04/2026. Essas datas **podem conter lacunas**, portanto o número de exemplos utilizáveis para treino ainda **não é conhecido**. Reproduzir e auditar a contagem nos CSVs reais antes de usá-la em slides ou conclusões.

## 4. Como interpretar os CSVs ANA
Os dados foram exportados em formato **mensal com colunas diárias** (por exemplo `Cota01` até `Cota31`), frequentemente com texto antes do cabeçalho `EstacaoCodigo;`. Há versões brutas e consistidas das medições e indicadores de qualidade/status por dia. O processo deve:
1. Detectar o cabeçalho verdadeiro e ler respeitando `;`, a codificação dos arquivos e a vírgula decimal.
2. Converter colunas diárias em linhas **uma data real por observação**; invalidar datas impossíveis (31 de abril, por exemplo).
3. Identificar observações duplicadas, qualidade e consistência, escolher uma observação por variável/data segundo regra documentada.
4. Manter chuva `0` como valor válido quando realmente observada; manter vazios como ausentes.
5. Conciliar calendários e avaliar lacunas, diferenças de horários/referências e períodos de interseção.
6. Checar **unidade da cota** e referência da régua; não excluir cotas negativas automaticamente.

## 5. Formulação supervisionada inicial
- **Exemplo:** uma data de referência `t` com histórico dos **últimos 7 dias** de cota e precipitação (proposta inicial, sujeita à análise de cobertura).
- **Entradas possíveis:** cotas `t-6 ... t` e chuvas `t-6 ... t`, talvez acumulados anteriores de 3 e 7 dias.
- **Saída:** cota medida em `t+1`.
- **Tipo:** regressão, com saída numérica contínua e neurônio final `Dense(1)` linear.
- **Regra central:** todas as entradas precisam existir **no momento declarado da previsão**. Se a chuva diária de `t` ainda não estiver disponível nesse instante, retroceder janela/horário lógico adequadamente.
- **Dados consecutivos:** não atravessar lacunas como se um dia faltante fosse zero, nem confundir uma linha existente com um dia consecutivo.

## 6. Experimentos propostos
1. **Persistência:** prever `cota(t+1) = cota(t)`; referência simples essencial.
2. **Um neurônio:** Keras `Sequential` com camada densa de saída, inspirado no professor.
3. **MLP pequena:** uma camada oculta; comparar em validação.
4. **MLP maior:** duas camadas ocultas, somente se os dados e o tempo justificarem.

Selecionar a arquitetura na validação, avaliar **uma vez** no teste cronologicamente posterior. Reportar MAE, RMSE, R² e gráficos de nível real versus estimado; correlação apenas como informação adicional. Normalizar entradas entre 0 e 1 com escala aprendida **apenas no treino**.

## 7. Relação com o PI5
O estudo acadêmico pode orientar futuramente escolhas de variáveis de chuva/nível do PI5. Contudo, o PI5 envolve outro local/sistema e inclui hardware, APIs meteorológicas e backend. **Não há transferência automática** de um modelo treinado em Encantado para barragens, nem validação de enchentes, racionamento ou sistema de alerta. ANA traz observações passadas; previsões meteorológicas futuras (por exemplo Open-Meteo) são uma fonte diferente e não devem ser confundidas com valores observados futuros.

## 8. Referências internas
- `docs/RELATORIO_TECNICO_ORIGINAL.md`: relatório detalhado com formatos, hipóteses, diagnóstico preliminar e limitações.
- `docs/REQUISITOS_PROFESSOR.md`: enunciado da disciplina e organização dos 5 minutos.
- `referencias/dl_class03_LinearRegressionSingleNeuron.ipynb`: sequência didática de uma regressão neural simples.
- [ANA / HidroWeb](https://www.snirh.gov.br/hidroweb/serieshistoricas).
