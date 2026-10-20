# Plano de implementação — Notebook V3 didático: previsão de cotas e análise de extremos do Rio Taquari

**Projeto:** Redes Neurais — estação fluviométrica ANA 86720000 (Encantado, RS)  
**Destinatário:** Codex, para implementação no VS Code / Jupyter  
**Estado:** especificação; **não** contém resultados da V3  
**Documento de resultados obrigatório:** `resultados.md` existente (preservar as versões anteriores)

## 1. Objetivo e recorte

Criar **um novo notebook didático, do início**, sem sobrescrever os notebooks V1/V2, para:

1. Investigar episódios históricos de cotas altas e baixas e a precipitação acumulada nos **7, 15 e 30 dias anteriores**; incluir vazão na análise exploratória de níveis baixos quando os dados permitirem.
2. Treinar modelos de **regressão** que estimem a cota média diária futura, inicialmente `t+1`, com dados efetivamente disponíveis até `t`; comparar históricos 7/15/30 dias e indicadores acumulados de chuva.
3. Aplicar **classificação por regras transparentes** à cota prevista, baseada em faixas operacionais/experimentais documentadas, e avaliar acertos, erros e transições entre faixas.
4. Analisar separadamente a capacidade de antecipar **níveis altos** e **níveis baixos persistentes**, sem converter tais classes em previsão de inundação instantânea, desastre real ou recomendação de racionamento.
5. Explicar didaticamente cada etapa, permitindo ao estudante defender as escolhas para o professor Sandro.

**Problema central para a disciplina:** regressão supervisionada da cota do dia seguinte. A classificação é uma camada interpretativa complementar, não necessariamente outra rede neural.

## 2. Convenções e limites — OBRIGATÓRIO diferenciar fatos e hipóteses

A medição da estação 86720000 corresponde a um **rio, não à porcentagem de armazenamento de uma barragem**. Evitar chamar a cota de “nível da barragem” e evitar inferir volume disponível ou racionamento diretamente dela.

Referências documentadas em boletins do SGB para Encantado, **dependentes de contexto e vigência**, a conferir nas fontes:
- Atenção: **500 cm (5 m)**.
- Alerta: **900 cm (9 m)**.
- Inundação: **1.200 cm (12 m)**.

**Proposta experimental do estudante, não oficial:** marcar valores **acima de 1.500 cm (15 m)** como `faixa extrema experimental` ou `possível evento muito severo`, **não** rotular automaticamente como desastre real. Verificar se há registros que justifiquem a denominação, não apresentar esta classe como limite institucional.

**Para baixa cota não há limite oficial de seca/racionamento validado para esse ponto.** Propor inicialmente um **limiar estatístico derivado exclusivamente do treino** (ex.: P10) e rotular `nível estatisticamente baixo` ou `atenção experimental por nível baixo`, nunca `seca comprovada`, `risco de racionamento` ou `iniciar racionamento`. Um episódio prolongado de cota e vazão reduzidas pode ser denominado `condição hidrológica compatível com estiagem`, com ressalvas. Não inventar um valor inferior fixo como se fosse oficial.

### 2.1 Faixas para apresentação, após validação de unidade e referencial

Usar intervalos mutuamente exclusivos, com fronteiras programadas e explicadas:

| Categoria | Critério provisório | Natureza |
|---|---|---|
| Nível estatisticamente baixo | cota < P10 do treino | Hipótese estatística, não seca oficial |
| Faixa usual / sem alerta de cheia | P10 <= cota < 500 cm | Faixa de referência neste estudo; pode incluir outras condições |
| Atenção para cheia | 500 <= cota < 900 cm | Referência de boletim SGB, dependente de validade |
| Alerta para cheia | 900 <= cota < 1200 cm | Referência de boletim SGB, dependente de validade |
| Acima da cota de inundação | 1200 <= cota <= 1500 cm | Limite de inundação documentado; média diária não comprova inundação local |
| Extremo experimental (>15 m) | cota > 1500 cm | Corte criado para estudo; **não** “desastre” confirmado |

Se o P10 resultar >= 500 cm, interromper essa categorização e exigir revisão; não criar intervalos sobrepostos. Se unidade ou referencial não forem compatíveis, **não aplicar limites oficiais à série**; usar apenas faixas estatísticas claramente identificadas enquanto se documenta a pendência.

A classe de 12–15 m deve deixar claro que já está **acima** do limiar de inundação, não que “a inundação só ocorre em 15 m”. A marca histórica de maio/2024 (23,14 m) é observação de evento, **não teto físico**.

Fontes para consulta/registro, conferindo seus documentos e datas:
- SGB SACE Taquari: https://www.sgb.gov.br/sace/taquari_apresentacao.php
- Boletim SGB 12/05/2024: https://www.sgb.gov.br/sace/boletins/Taquari/20240512_02-20240512%20-%20032619.pdf
- Levantamento do zero ortométrico (28,160 m, não prova estabilidade histórica): https://rigeo.sgb.gov.br/bitstream/doc/25518/1/marcuzzo_silva_Mendes_zero%20ortometrico_artigo.pdf
- Levantamento da cheia de 2024: https://rigeo.sgb.gov.br/bitstream/doc/24939.10/6/2024-11-04_nota_tecnica_levantamento_cheia_2024_v.8.pdf
- ANA HidroWeb: https://www.snirh.gov.br/hidroweb/serieshistoricas

Documentar se os metadados originais de cada arquivo confirmam centímetros, o mesmo zero da régua e agregação. Registrar que dados **médios diários** não equivalem a máximas/leituras instantâneas. Exemplo de auditoria conhecido: em 02/05/2024 existe leitura instantânea `2314`, mas ela não deve ser introduzida artificialmente na série de média diária onde a observação elegível está ausente.

## 3. Arquivos e entregáveis

Entradas (não recriar ou substituir os CSVs originais):
- `dados/86720000_Cotas.csv`
- `dados/86720000_Vazoes.csv`
- `dados/2951010_Chuvas.csv`

Se a estrutura local for diferente, localizar arquivos sem assumir caminhos e documentar a decisão.

Gerar:
- `notebooks/analise_redes_neurais_v3.ipynb` — notebook novo, autocontido e educativo.
- `resultados/v3/` — CSVs de limpeza, métricas, episódios e figuras; sem sobrescrever V2.
- **Atualizar exclusivamente o arquivo `resultados.md` já existente** com uma seção claramente intitulada `Versão 3`, mantendo V1 e V2. Escrever em UTF-8, corrigindo caracteres corrompidos somente quando a correção for segura e sem apagar conteúdo.

Não criar resultados falsos, nem prometer funcionamento operacional. Salvar figuras e tabelas com caminhos relativos ao projeto.

## 4. Padrão pedagógico obrigatório antes de CADA célula de código

Antes de toda célula de código, inserir **uma célula Markdown** com, no mínimo:

1. **O que esta célula faz** — uma ou duas frases simples.
2. **Por que precisamos dela** — relação com o problema hidrológico.
3. **Conceitos usados** — explicar termos iniciantes na primeira aparição (biblioteca, DataFrame, variável, normalização, neurônio, camada, época, batch, loss, MAE, RMSE, R², baseline, limiar, percentil, seed).
4. **Entradas e saídas esperadas** — arquivos/variáveis, tipo de saída, gráficos.
5. **Como interpretar e verificar** — o que observar para detectar dados incorretos, vazamento temporal ou falhas.

Quando houver parâmetros, **registrar seus valores e a justificativa**, por exemplo:
- `seed = 42`: **uma semente aleatória**, não “42 seeds”; controla sorteios pseudoaleatórios para facilitar reprodução, mas não garante resultados absolutamente idênticos entre equipamentos e versões.
- `Dense(32, activation='relu')`: camada com **32 neurônios**, explicar o que um neurônio faz e por que testar esse tamanho.
- `Dense(64) -> Dense(32) -> Dense(1)`: duas camadas ocultas e uma saída numérica; distinguir parâmetros treináveis de quantidade de neurônios.
- `epochs=...`: número máximo de passagens pelo treino; `EarlyStopping` pode encerrar antes.
- `batch_size=...`: exemplos processados antes de atualização dos pesos.
- `validation_data`: conjunto utilizado para selecionar modelos; teste não pode influenciar escolhas.
- `MinMaxScaler`: ajustado somente ao treino, sem usar mínimo/máximo futuro.

Usar linguagem simples, mas tecnicamente correta. Evitar blocos enormes com dezenas de operações sem explicação. Depois de gráficos, acrescentar Markdown de interpretação fundamentado nos resultados realmente gerados. Se o notebook não foi executado, usar textos de orientação, nunca conclusões inventadas.

### Exemplo de célula didática Markdown

> **Etapa 9 — Definir a semente aleatória**  
> **O que faremos:** fixaremos `seed = 42` para tornar a inicialização da rede mais reproduzível.  
> **Conceito:** a *seed* é um número usado para iniciar geradores pseudoaleatórios. Ela não representa quantas redes serão treinadas.  
> **Por que importa:** permite comparar arquiteturas reduzindo parte das diferenças aleatórias entre execuções.  
> **Saída:** configuração reprodutível; não gera previsão.  
> **Verificação:** registrar versões do Python e TensorFlow e reconhecer que o hardware ainda pode influenciar pequenas diferenças.

## 5. Roteiro das células do notebook (cada item com Markdown anterior)

1. **Capa e pergunta científica:** título, objetivo, hipóteses, limites do que é previsão.
2. **Bibliotecas e versões:** Python, NumPy, pandas, matplotlib, sklearn, TensorFlow/Keras; explicar utilidade.
3. **Sementes e reprodutibilidade:** `random`, NumPy e TensorFlow, justificar seed.
4. **Localizar arquivos de dados e inspecionar cabeçalhos brutos:** `cp1252`, `;`, decimal com vírgula, `EstacaoCodigo;`.
5. **Carregar e verificar colunas:** mostrar `.head()`, `.info()`, dimensões e dicionário dos campos.
6. **Transformar formato mensal para diário:** `Cota01..31`, `Chuva01..31`, `Vazao01..31`, datas válidas, `Status`, `NivelConsistencia`, `MediaDiaria`.
7. **Limpeza/auditoria:** consistido quando elegível, mediana em empates, status real, zero de chuva válido, lacunas explícitas, não misturar instantâneo com média diária.
8. **Investigar unidades e episódio 02/05/2024:** conferir 2314 instantâneo versus ausência de média diária, registrar limitações documentais.
9. **Montar série diária alinhada e cobertura:** cota, chuva, vazão, contagem de dias e buracos.
10. **Análise exploratória das distribuições:** linha temporal, histograma, médias, quantis, extremos; não equiparar percentis a limites oficiais.
11. **Identificar episódios altos/baixos:** blocos consecutivos, data de início/pico, duração, separar episódios e evitar tratar todo dia como episódio independente.
12. **Analisar passado dos episódios:** chuva acumulada de 7, 15, 30 dias, dias chuvosos, períodos secos, cota e vazão anteriores; comparar a extremos e datas usuais; não usar resultados futuros como entradas preditivas.
13. **Definir janela de previsão e horizonte:** primeiro cota(t+1), sem restringir semanas seg–dom; janela deslizante de 7/15/30 dias para qualquer dia consecutivo válido.
14. **Engenharia de características:** níveis diários passados e chuva acumulada (3/7/15/30), chuva máxima, contagem de dias chuvosos, dias consecutivos secos, tendência de cota. Computar apenas com passado disponível.
15. **Protocolos de disponibilidade:** chuva até `t` versus até `t-1`, justificar cronologia e não comparar conjuntos de teste diferentes sem ressalva.
16. **Definir amostra comum:** mesma interseção de datas para J7/J15/J30, e se comparar protocolos usar também amostra comum entre eles. Reportar perdas por lacunas.
17. **Separar treino, validação e teste no tempo:** sem mistura aleatória; manter período de teste intocado para seleção, ou usar avaliação temporal adicional devido a resultados V1/V2 já conhecidos.
18. **Normalizar entre 0 e 1:** usar `MinMaxScaler` ajustado só em treino, para entradas (e alvo se escolhido), explicar como reverter escala. Valores de validação/teste podem sair fora do intervalo 0–1 se excederem os limites vistos no treino; não recortar automaticamente.
19. **Baselines:** persistência e Ridge, explicar por que modelo simples precisa ser comparado.
20. **Arquiteturas MLP:** pelo menos um neurônio linear, MLP 32 e MLP 64–32, seleção exclusivamente na validação; informar parâmetros, neurônios, funções de ativação e saída.
21. **Treinamento:** épocas, lotes, função de perda, Adam/RMSprop, EarlyStopping, curvas de treino/validação; registrar cada configuração.
22. **Previsões e regressão:** inverter escala, MAE, RMSE, R²; comparar erros por faixa e direção do erro.
23. **Gráficos individuais J7, J15 e J30:** cota observada × prevista no **mesmo intervalo**; preferir recortes legíveis e incluir trechos altos e baixos.
24. **Gráfico conjunto:** barras ou pontos de MAE, RMSE e R² por janela, com modelos/entradas identificados. Nunca comparar métricas de teste obtidas em datas diferentes como equivalentes.
25. **Classificação por regras sobre o valor previsto:** intervalos do item 2, após validação de unidade/referencial; caso contrário, apenas percentis. Comparar **classe prevista** contra **classe derivada da cota real diária**, usando matriz de confusão, precisão/recall/F1 por classe e exemplos de erros. Mostrar frequência de classes para expor desbalanceamento; não tratar precisão alta na classe usual como prova de confiabilidade nos raros extremos.
26. **Erros perto dos limiares:** mostrar previsões a ambos os lados de 500/900/1200/1500 cm, falsos alertas e eventos não detectados; relatar escassez de exemplos e confusão possível entre médias e picos instantâneos.
27. **Baixa cota e estiagem:** análise de duração e vazão reduzida, percentis derivados só de treino; **não** classificar racionamento nem emitir recomendação automática de racionar água.
28. **Conclusões e próximos passos:** hipóteses confirmadas/não confirmadas, comparação com persistência, limitações e interesse futuro em previsões meteorológicas emitidas antes do evento para múltiplos dias à frente.
29. **Exportação:** salvar resultados e figuras, manifesto e instrução de reexecução, atualizar `resultados.md` existente.

## 6. Regras científicas de avaliação

- Não selecionar apenas dias extremos para treinar regressão: incluir também dias usuais, para aprender distinção. O estudo retrospectivo dos episódios extremos é um módulo separado da previsão.
- Não criar características usando conhecimento posterior a `t`; nunca usar chuva futura observada como se fosse previsão meteorológica disponível hoje.
- Divisão cronológica, com validação para seleção e teste para avaliação final. Como V1 e V2 já analisaram parte do teste, rotular V3 como **exploratória** e propor validação walk-forward / novo período futuro para confirmação.
- Fixar as mesmas datas para comparações justas; gerar log de descartes quando 30 dias completos reduzem muito a amostra.
- Calcular limiares estatísticos (P10 etc.) **só com treino** e aplicar sem recalcular em validação/teste.
- Evitar comparação direta de limiares SGB com médias diárias enquanto referência, unidade e tipo de medição não estiverem validados.
- Saídas de regressão não devem ter teto artificial em 15 m: é um limite de **categoria experimental**, não um máximo físico. Valores negativos possíveis devem ser analisados com referência da régua, não simplesmente truncados a zero.
- Avaliar falsos negativos de níveis altos, falsos positivos e precisão/recall para classes raras. Para estiagem, analisar persistência temporal e vazão sem afirmar necessidade de racionamento.
- A classificação por faixas é uma camada determinística sobre regressão; chamar de `classificação baseada em regras`, não dizer que treinamos uma segunda rede classificadora. Se no futuro treinarmos uma classificação diretamente, documentar e avaliar separadamente.
- Conferir se categorias definidas por boletins contemporâneos são aplicáveis aos dados históricos anteriores, que podem ter outro referencial.
- Quando não houver exemplos suficientes numa classe, escrever “métrica indisponível ou instável”, não inventar taxas.

## 7. Requisitos de figuras e interpretação

Gerar, quando houver dados suficientes:

1. **7 dias:** gráfico real × previsto em um trecho histórico comum.
2. **15 dias:** gráfico real × previsto no mesmo trecho.
3. **30 dias:** gráfico real × previsto no mesmo trecho.
4. **Comparação J7/J15/J30:** MAE, RMSE e R², com contexto de modelos e número de amostras.
5. **Chuva acumulada por evento:** precipitação histórica 7/15/30 dias anteriores a altos/baixos, com distribuição e nº de episódios.
6. **Episódios de níveis altos e baixos:** cota, vazão (quando disponível), datas, lacunas e evento identificado.
7. **Faixas previstas × observadas:** matriz de confusão com nomes das categorias e tabela de precisão/recall/F1.
8. **Erros por faixa:** MAE e viés assinado, com número de observações em cada faixa.

Cada figura deve ter título, unidades (apenas quando comprovadas), legenda, fonte e um pequeno comentário de interpretação. Os limites oficiais usados deverão ter fonte/data e ressalvas.

## 8. Saída escrita obrigatória em `resultados.md`

**Atualizar o arquivo existente `resultados.md`. Não criar `resultados_v3.md` separado nem substituir resultados anteriores.** Acrescentar ao final, em UTF-8:

- `## Versão 3 — Objetivos e protocolo`.
- `## Versão 3 — Auditoria de dados e unidades`.
- `## Versão 3 — Episódios históricos altos e baixos`.
- `## Versão 3 — Entradas de chuva acumulada e janelas 7/15/30`.
- `## Versão 3 — Arquiteturas e parâmetros explicados` (seed, neurônios, épocas, lote, perda, critérios de escolha).
- `## Versão 3 — Resultados da regressão` (tabela de MAE/RMSE/R², baselines, períodos e N, figuras).
- `## Versão 3 — Classificação por limites` (fontes, definição das classes, matriz de confusão, precision/recall/F1 por classe, disponibilidade de dados).
- `## Versão 3 — Cheias e níveis baixos persistentes` (interpretar sem afirmar risco real de racionamento).
- `## Versão 3 — Limitações, conclusões e próximos passos`.

Todas as tabelas devem derivar de execução real; não escrever números fictícios. Referenciar os gráficos e tabelas de `resultados/v3/` por links relativos válidos. Manter histórico comparável da V1 e V2, com ressalvas de diferenças de amostra.

## 9. Processo de execução pedido ao Codex

1. Ler `AGENTS.md`, `README.md`, docs existentes, notebook V1/V2 e `resultados.md` antes de alterar arquivos.
2. Apresentar o plano de células e confirmar presença/esquema dos CSVs; sinalizar eventuais bloqueios.
3. Criar novo notebook V3 **sem apagar trabalhos anteriores**.
4. Implementar e executar gradualmente; incluir Markdown explicativo **antes de cada célula de código**.
5. Conferir saídas, contagens, vazamento temporal e gráficos. Reportar advertências em vez de ocultá-las.
6. Salvar artefatos em `resultados/v3/` e consolidar **somente no `resultados.md` existente**.
7. Resumir o que foi implementado, o que foi validado e o que continua pendente, com caminhos dos arquivos.

## 10. Critérios de conclusão

- [ ] Novo notebook V3 independente e executável por ordem de células.
- [ ] Toda célula de código precedida por explicação didática em Markdown.
- [ ] Seeds, camadas, neurônios, épocas, lotes e métricas explicados em termos simples.
- [ ] Origem e qualidade dos dados auditadas; instantâneo ≠ média diária documentado.
- [ ] Limiares oficiais/contextuais e experimentais inequivocamente diferenciados.
- [ ] Estudo retrospectivo de episódios extremos e baixos com janelas 7/15/30.
- [ ] Regressão `t+1` comparada com baselines, datas e métricas auditáveis.
- [ ] Classificação **por regras** sobre previsões avaliada com matriz de confusão e métricas por classe.
- [ ] Figuras de J7/J15/J30 e comparativa com mesma base de teste.
- [ ] Nenhuma afirmação automática de desastre, inundação instantânea ou necessidade de racionamento.
- [ ] Novos resultados adicionados ao **mesmo `resultados.md`**, preservando V1/V2.

---

**Nota final:** os limites de 5, 9 e 12 m são referências documentadas para a estação e precisam ser contextualizados por data, regime de medição e referencial. O corte de **15 m é inteiramente experimental**. O “nível baixo” inicialmente será definido por percentil do treino; não existe no material analisado uma cota oficial que por si determine racionamento em Encantado. A V3 é um estudo acadêmico e não um sistema de decisão operacional.
