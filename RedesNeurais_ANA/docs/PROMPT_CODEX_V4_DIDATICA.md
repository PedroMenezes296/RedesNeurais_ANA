# Prompt para o Codex — Notebook V4 didático (Rio Taquari / ANA)

> **Instrução principal:** implemente a V4 neste repositório. Leia `AGENTS.md`, `README.md`, `resultados.md`, `docs/` e os notebooks anteriores antes de editar. Não altere nem apague V1, V2 ou V3. Se caminhos diferirem, localize os arquivos equivalentes e registre a diferença. Não invente saídas ou métricas.

## 1. Entrega e objetivo

Criar **`notebooks/analise_redes_neurais_v4.ipynb`** novo, executável no VS Code com o ambiente do projeto. Estudo acadêmico, reprodutível e didático para prever **cota média diária do Rio Taquari em Encantado (estação ANA 86720000)** e classificar a **cota prevista** por faixas predefinidas. Explorar históricos de **7, 15 e 30 dias**, chuva diária e **acumulados anteriores**. Não confundir estação fluviométrica em rio com nível/volume armazenado em barragem; não inferir racionamento.

Conservar as fontes originais sem alteração; criar os resultados em **`resultados/v4/`** e **ACRESCENTAR** seção detalhada da V4 ao **`resultados.md` existente**, sem apagar resultados anteriores. Salvar gráficos PNG legíveis, tabelas CSV e um manifesto dos arquivos produzidos. Todo resultado deve vir de uma execução real; registrar erros e lacunas em vez de preencher artificialmente.

## 2. Regra de divisão: 80% para treino/desenvolvimento, 20% para teste

- **Divisão cronológica, nunca aleatória**: ordenar pelo dia-alvo e reservar os **20% finais** como teste, sem usar essas observações para selecionar modelo, hiperparâmetros, limiares estatísticos ou transformações.
- Os **80% iniciais** são o conjunto de *desenvolvimento/treinamento*. Para escolher arquitetura e efetuar *early stopping*, separar **uma validação temporal dentro dos 80%** (sugestão: 85% desse bloco para ajuste e 15% para validação). Explicar visualmente: 68% treino efetivo + 12% validação + 20% teste **não equivale** a 80% de ajuste efetivo; é 80% desenvolvimento, 20% teste. Se usar ajuste final nos 80% após selecionar hiperparâmetros, explicar procedimento, ausência de vazamento e como definir número de épocas, sem consultar o teste.
- **Comparação justa entre J7/J15/J30**: fixar mesmo calendário de cortes e usar **interseção comum de datas-alvo elegíveis para todos os experimentos comparados**. Reportar totais antes/depois da interseção e quantos exemplos existem em cada conjunto. Se variantes de disponibilidade de chuva (`t` e `t-1`) forem comparadas, avaliar também em datas-alvo comuns.
- Inputs devem existir na data em que a previsão seria emitida; alvo `cota(t+1)` (média diária). Não usar dado de amanhã nas features, escala, rótulos ou estatísticas do treinamento. Ajustar imputação, seleção de features e escaladores **somente no treino efetivo**. Não preencher falhas de chuva como zero. Não criar janelas que atravessem lacunas.
- Estabelecer claramente a hipótese operacional da chuva até `t`: só disponível após fechar o dia `t`. Se houver cenário com chuva até `t-1`, explicar a diferença. A estação de chuva isolada não representa toda a bacia.
- Como o período de teste V1/V2/V3 já foi inspecionado, qualificar a V4 como **avaliação exploratória**, não prova confirmatória. Opcionalmente fazer validação temporal em múltiplas origens (*walk-forward*) apenas no bloco de desenvolvimento; **não usar teste para escolher o melhor cenário**.

## 3. Dados e auditoria

- Usar CSVs ANA existentes para cotas `86720000_Cotas.csv`, chuvas `2951010_Chuvas.csv` e, se útil como análise descritiva, vazões `86720000_Vazoes.csv`.
- Reproduzir a expansão mensal->diária, `Status=1`, média diária para cotas, preferência `NivelConsistencia=2`, deduplicação documentada, ordenação e ausência de interpolação indevida. Auditar as quantidades e a cobertura por ano.
- Verificar cabeçalhos/metadados. A convenção hidrográfica de cotas da ANA é geralmente **cm**, mas se os CSVs não declararem expressamente a unidade, sinalizar **convenção provável, não confirmação específica**. Para usar limiares em metros, demonstrar explicitamente a hipótese de conversão `100 unidades CSV = 1 metro`, com auditoria cruzada (p.ex. leitura instantânea de 02/05/2024 e levantamento SGB), e documentar que não há prova de continuidade histórica do zero da régua. Não misturar leitura instantânea de 2314 com a média diária usada na regressão.
- A série mede **rio**, não percentual de reservatório; a cota é relativa ao zero da régua, e não profundidade absoluta.

## 4. Engenharia de atributos e alvo

- Construir janelas **móveis de dias corridos consecutivos**, e não semanas de segunda a domingo: para cada data `t`, usar cotas de `t-(n-1)` a `t` com `n in {7,15,30}` para prever `t+1`.
- Comparar pelo menos: **(A)** cotas anteriores; **(B)** cotas + precipitações diárias; **(C)** cotas + indicadores de precipitação antecedente (somas de 3/7/15/30 dias, dias sem chuva, máximo diário; somente os calculáveis a partir de informações até `t`). Não exigir 30 dias para um indicador de 30 dias sem documentar sua perda de cobertura. Não adicionar dezenas de recursos sem testes básicos/controle de dimensionalidade.
- Mostrar com exemplos de datas como calcular `rolling().sum()` e o deslocamento temporal correto. Para a previsão `t+1`, acumulado até `t` deve ser computado sem incluir `t+1`; para evento do dia-alvo, esclarecer o deslocamento correspondente.
- Incluir baselines **persistência** (`cota prevista(t+1)=cota(t)`) e **Ridge** nas mesmas datas de teste. Treinar modelos MLP simples, com arquitetura(s) comparáveis às versões anteriores, registrando seeds, camadas, neurônios, otimizador, learning rate, batch size, loss, número máximo e real de épocas, early stopping e demais escolhas.

## 5. Regressão: métricas e gráficos de aprendizado

- Relatar **MAE, RMSE, R²**, unidade do erro, número de amostras e comparação honesta com baselines. **Não chamar MAE ou R² de acurácia.** Não converter regressão em percentual de acerto sem antes definir tolerância mensurável (p.ex. acerto sob erro absoluto <= X cm), e nesse caso identificá-la como uma *métrica adicional por tolerância*, não como acurácia padrão.
- **Curvas de aprendizado reais**: para a(s) arquitetura(s) representativa(s), plotar por época **loss do treino e da validação**, com indicação da época escolhida pelo *early stopping* e descrição simples de possível *overfitting/underfitting*. Não inventar gráfico ilustrativo. Explicar que não é o gráfico do cotovelo do K-means.
- Gráficos **observado × previsto** em período de teste coincidente J7/J15/J30, dispersão `real × previsto` com diagonal ideal, resíduos ao longo do tempo e por nível, barras comparativas MAE/RMSE/R² (R² em escala separada ou painéis, para não misturar magnitudes). Acrescentar um gráfico de chuva diária e acumulada antes de episódios elevados/baixos, com datas.
- Documentar se a série diária suaviza máximos instantâneos; não afirmar desempenho em picos de enchente a partir de médias diárias.

## 6. Regressão seguida de classificação por faixas

Aplicar **as mesmas regras aos valores reais e previstos no teste**, depois da regressão. Isso é **classificação determinística pós-regressão**, não uma segunda rede neural de classificação. Apresentar faixas com fronteiras sem ambiguidade; quando a unidade CSV->cm não estiver confirmada, emitir alerta visível antes de reportar resultados em metros e não atribuir status operacional.

| Categoria V4 | Regra proposta (em m, se conversão validada) | Natureza |
|---|---|---|
| Nível baixo experimental | `cota < P10` do treino e abaixo de 5 m | Percentil estatístico; **não é critério oficial de seca** |
| Usual | de P10 inclusive até **menos de 5 m** | Faixa analítica |
| Atenção | `5 <= cota < 9` | Referência histórica de boletins SGB; conferir mesma estação/referencial |
| Alerta | `9 <= cota < 12` | Referência histórica de boletins SGB |
| Faixa acima da cota de inundação | `12 <= cota <= 15` | 12 m documentado, mas leitura diária ≠ máxima instantânea |
| Extremo experimental | `cota > 15` | **Limite criado para estudo, não classificação oficial de desastre** |

Resolver exatamente `15 m` na faixa anterior conforme tabela; se preferir `>=15` extremo, documentar alteração antes de implementar. Evitar chamar rótulos `seca`, `desastre confirmado` e `racionar agora` sem dados externos oficiais. **Nunca tratar 15 m como teto físico da previsão** nem limitar `y_pred` artificialmente.

- Mostrar **matriz de confusão** das categorias V4 no conjunto de teste, **acurácia**, precisão, recall e F1 por categoria, *macro-F1*, *balanced accuracy* e **suporte (quantidade real de casos)**. Classes sem exemplos no teste precisam ser explicitamente marcadas **não avaliáveis** (não divulgar 100% de acerto). Incluir baseline simples de sempre prever a classe majoritária, e, se pertinente, classificar também a persistência.
- Identificar **falsos negativos** das faixas elevadas (observado >= 9/12 m e previsto abaixo) e reportar contagens, datas e razões, além dos falsos positivos. Não confiar apenas na acurácia global de dados desbalanceados. Se não houver ocorrências suficientes de >=12 ou >15 m no teste diário, dizer explicitamente; não fabricar eventos, não declarar classificação de inundação validada.
- Como complemento, preservar ou reproduzir a classificação **estatística P10/P90** da V3 para uma comparação histórica, **rotulada separadamente** das classes hidrológicas V4. Não reutilizar os ~94% da V3 como acurácia das novas faixas.
- Calcular métricas a partir das previsões fora do treino, não do conjunto de treinamento. Classificar previsões de regressão não prova utilidade de alerta em tempo real.

## 7. Episódios de cheias e de nível baixo

- Escolher, **para análise retrospectiva descritiva**, episódios reais extremos (altos e baixos) a partir da série disponível; apresentar tabela com data, cota observada, dados disponíveis, maior precipitação diária, acumulados antecedentes e variação de cota; comparar janelas 7/15/30 e eventuais datas com ausência de dados.
- Distinguir o uso de eventos conhecidos para **explicação histórica** do uso de eventos futuros para **seleção/treino**. Evitar vazamento temporal e seleção de teste baseada em desempenho.
- Relacionar eventos de 2023/2024 às fontes SGB **apenas quando os dados diários suportarem**, inclusive a ausência da média em 02/05/2024. Não comparar diretamente valor instantâneo com média diária nem inferir racionamento a partir de baixa cota.

## 8. Regra DIDÁTICA obrigatória: Markdown ANTES de CADA célula de código

Estruturar o notebook com **células Markdown imediatamente antes de cada célula executável**, sem exceção. Cada explicação deve ser acessível a um iniciante e conter, de forma compacta:

1. **O que esta célula faz** e por que existe;
2. **Conceitos em linguagem simples** (ex.: `seed`, neurônio, época, *batch size*, MinMaxScaler, MLP, Huber, regressão, classificação, matriz de confusão, precision/recall, early stopping);
3. **Parâmetros reais desta célula**, seus valores e o significado, sem prometer o que ainda não foi computado;
4. **O que esperamos ver na saída** e **como interpretá-la**; evitar conclusões pré-fabricadas.

Exemplo de explicação de uma célula de modelo: "Nesta etapa testaremos uma MLP com duas camadas ocultas de 64 e 32 neurônios. Neurônios são unidades matemáticas que combinam as entradas e aprendem padrões. A seed 42 fixa parte dos sorteios para melhorar a reprodutibilidade, mas resultados podem variar dependendo do hardware e bibliotecas. Usaremos até 120 épocas (passagens pelo treino), lotes de 128 exemplos e early stopping para interromper quando a validação parar de melhorar." **Adaptar esse texto às configurações reais**, não copiar parâmetros incompatíveis.

Após seções de gráficos, inserir **célula Markdown interpretativa posterior** usando os resultados efetivamente obtidos, explicando tendências, limitações e o que significa errar casos de cheia. Usar UTF-8; títulos e legendas em português, unidades consistentes, gráficos legíveis.

## 9. Estrutura sugerida de seções do notebook

1. Perguntas da pesquisa, limites de interpretação e diferença entre rio e barragem.
2. Bibliotecas, seed e ambiente de execução.
3. Importação e auditoria de dados ANA.
4. Transformação das séries diárias e limpeza.
5. Exploração de níveis altos/baixos, eventos conhecidos e chuva antecedente.
6. Construção das features e alvos, com exemplos que demonstram ausência de vazamento.
7. Divisão cronológica 80/20 e validação interna.
8. Baselines e pré-processamento com scalers ajustados somente no treino.
9. Treinamento de MLPs e curvas de perda treino/validação.
10. Comparação J7/J15/J30, com e sem indicadores acumulados.
11. Avaliação final da regressão: métricas, observado × previsto, resíduos.
12. Classificação por faixas propostas V4 e comparação com faixas estatísticas V3.
13. Acurácia, matriz de confusão, precision/recall/F1 e análise de falsos negativos nos eventos altos.
14. Conclusões, limitações, reprodutibilidade e trabalhos futuros.

## 10. Registro obrigatório no resultados.md

Acrescentar **`## Versão 4 — ...`** no final de `resultados.md`; registrar:
- Data/ambiente/versões de bibliotecas, arquivos de entrada, filtros, janelas elegíveis e intervalos exatos de treino, validação e teste;
- Unidade assumida/confirmada, limiares e ressalvas, parâmetros dos modelos e seleção por validação;
- Tabelas com MAE/RMSE/R² e comparação contra persistência e Ridge nas mesmas datas;
- Tabela de classificação V4: acurácia, macro-F1, balanced accuracy, precision/recall/F1/suporte por classe; baseline, matriz de confusão; classificação V3 P10/P90 opcional separada;
- Casos reais de erro significativo e falsos negativos em níveis altos; gráficos, respectivos links relativos, conclusões, limitações e recomendações para pesquisas futuras;
- Descrever que os resultados são **exploratórios** e que classificação não corresponde a alerta oficial de emergência nem determina racionamento.

## 11. Critérios de aceitação / checagem final

- [ ] V1, V2 e V3 intactas; V4 criada e executa do início ao fim (ou falha documentada com causa precisa).
- [ ] Toda célula de código é precedida por Markdown didático com parâmetros corretos.
- [ ] Separação 80% desenvolvimento/20% teste cronológica, validação interna explícita, sem vazamento.
- [ ] Janelas 7/15/30 comparadas nas mesmas datas do teste, com contagem de amostras reportada.
- [ ] Acumulados de chuva antecedente calculados sem dados do futuro.
- [ ] Curvas reais loss/val_loss por época, gráficos de regressão e explicação do formato (não K-means).
- [ ] MAE, RMSE, R², persistência, Ridge, documentação da unidade e limites.
- [ ] Classificação V4: matriz de confusão, acurácia, precision, recall, F1 e suporte, inclusive métricas nos extremos; aviso para classes raras/ausentes.
- [ ] Nenhuma conclusão de seca, racionamento, risco operacional real ou desastre confirmado a partir de critério experimental.
- [ ] `resultados.md` atualizado em UTF-8 sem apagar histórico, `resultados/v4/` contendo outputs reais e manifesto.

**Ao terminar:** informe os arquivos criados, parâmetros efetivamente usados, resumo quantitativo genuíno, etapas que não puderam ser realizadas e pontos que precisam de validação externa. **Não invente resultados, screenshots ou links.**
