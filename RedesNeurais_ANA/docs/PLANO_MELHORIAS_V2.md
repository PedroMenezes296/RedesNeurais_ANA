# Plano de melhorias V2 — Redes Neurais ANA / Rio Taquari

**Data:** 20/10/2026  
**Destino:** `docs/PLANO_MELHORIAS_V2.md`  
**Executor:** Codex no repositório `RedesNeurais_ANA`  
**Status:** planejamento; resultados da V2 ainda não foram calculados.

> **Regra obrigatória de entrega:** ao terminar cada etapa com resultados verificáveis, atualizar **`resultados.md`**, preservando os resultados anteriores e acrescentando uma seção identificada como **Versão 2 — comparações de janelas e análise hidrológica**. Se o repositório já mantiver `resultados.md` dentro de `resultados/`, atualizar esse arquivo existente, sem criar versões conflitantes. O notebook e as figuras devem ser consistentes com os números documentados. Não inventar dados, métricas ou conclusões.

## 1. Contexto e objetivo

Projeto acadêmico de regressão supervisionada para prever a **cota média diária do Rio Taquari em Encantado (RS) no dia seguinte (`t+1`)** a partir de cotas e precipitações disponíveis até `t`. Executado em Jupyter Notebook (`notebooks/analise_redes_neurais.ipynb`) pelo VS Code. Seguir o enunciado do professor Sandro e usar seu notebook de regressão com Keras como referência didática, sem copiar soluções inapropriadas para séries temporais.

Bases originais ANA/HidroWeb:

- `86720000_Cotas.csv`: nível/cota, estação Encantado, código 86720000;
- `2951010_Chuvas.csv`: chuva diária observada, estação 2951010;
- `86720000_Vazoes.csv`: vazão, estação 86720000 (análise complementar, não assumir medição independente da cota).

**Importante:** não transformar a tarefa em classificação nem declarar que o modelo prevê enchentes ou racionamento. O alvo primário continua sendo a **cota do dia seguinte**, não o pico instantâneo, nem a porcentagem de armazenamento de barragem. Cheias e estiagens são análises de episódios e faixas de nível.

## 2. Estado atual: não perder a versão 1

Conforme o `resultados.md` fornecido anteriormente pelo estudante, a V1 utilizou janelas de 7 dias, divisão temporal e MLP 64–32 com cotas + chuvas. No teste final reportou: MLP (MAE 32,112; RMSE 71,777; R² 0,769), persistência (32,842; 83,625; 0,686) e Ridge (31,375; 74,371; 0,752). O erro médio da MLP nos níveis estatisticamente altos foi maior que nas outras faixas. **Esses números são referência histórica da V1, não resultados da V2.** Antes de reutilizá-los verificar se são reproduzíveis no código e nos arquivos locais.

Antes de editar:

1. Ler `README.md`, `AGENTS.md`, `docs/`, `resultados.md` (ou `resultados/resultados.md`), `requirements.txt`, o notebook atual e o notebook de referência do professor, se existirem.
2. Conferir nomes e caminhos reais dos CSVs; não supor que estejam presentes.
3. Criar checkpoint com Git ou cópia versionada do notebook **sem sobrescrever o original sem possibilidade de recuperação**.
4. Registrar estado inicial e eventuais problemas de reprodutibilidade em `resultados.md`.
5. Fazer modificações progressivas, mantendo Markdown explicativo e execução de cima para baixo.

## 3. Fase A — Auditoria das cotas e das fontes

### 3.1 Unidade e referências

Verificar no cabeçalho dos CSVs, metadados da estação e documentação institucional:

- unidade de `CotaXX` (a documentação geral da ANA costuma apresentar cotas em **centímetros**; confirmar a exportação efetivamente usada);
- a definição de `MediaDiaria`, `TipoMedicaoCotas`, `Status`, `NivelConsistencia` e outros campos pertinentes;
- coincidência de código, localização e referencial entre ANA/HidroWeb e SGB, evitando assumir que um mesmo código prova referência imutável durante toda a série;
- histórico de mudança de zero da régua, deslocamento da estação ou mudança do procedimento de medição. Se não houver evidência documental, registrar **não confirmado**.

O levantamento do SGB sobre zeros ortométricos informa **28,160 m para o zero da régua da estação Encantado 86720000**. Tratar essa informação como referencial documentado pelo estudo, **não como prova de invariância de 1941 a 2026**.

### 3.2 Comparação de valores e eventos

- Inspecionar o trecho dos dados do início de maio de 2024, em especial a data 02/05/2024, e reportar valores e status realmente presentes, sem preenchimentos artificiais.
- O SGB documentou marca de cheia de **23,14 m** nesse evento e há referências oficiais de cota de inundação de aproximadamente **12 m** para Encantado. **A marca de cheia é pico/levantamento de evento, não necessariamente a média diária observada no CSV.** Confirmar fonte, definição, data de vigência e equivalência de referência antes de sobrepor limiares a dados diários.
- Registrar separadamente limiares **estatísticos** (percentis definidos a partir do treino) e **operacionais** (atenção/alerta/inundação com fonte oficial e data). Não chamar P90 de “cota de inundação”.
- Usar centímetros nos cálculos enquanto for a unidade validada; opcionalmente converter para metros nos gráficos, informando a conversão `m = cm / 100`. Calcular métricas e legendas de forma coerente.
- Se média diária e valores instantâneos não forem comparáveis, explicar a diferença explicitamente no notebook e em `resultados.md`; **não concluir que a rede antecipa máximos instantâneos**.

### Fontes para verificar e documentar no repositório

- ANA HidroWeb: <https://www.snirh.gov.br/hidroweb/serieshistoricas>
- ANA, manual de estações fluviométricas: <https://www.gov.br/ana/pt-br/assuntos/monitoramento-e-eventos-criticos/monitoramento-hidrologico/monitoramento-hidrologico-do-setor-eletrico/resolucao-conjunta-ana-aneel-127-2022/ManualInstalaoEstaesFluviomtricas.pdf>
- SGB, levantamento de zeros ortométricos: <https://rigeo.sgb.gov.br/bitstream/doc/25518/1/marcuzzo_silva_Mendes_zero%20ortometrico_artigo.pdf>
- SGB, marcas de cheia de 2024: <https://rigeo.sgb.gov.br/server/api/core/bitstreams/3c354e4a-7035-4e2e-890b-fb3fb8becec1/content>
- SGB, alerta hidrológico Taquari: <https://www.sgb.gov.br/sace/taquari_apresentacao.php>

Não declarar que cada documento foi revalidado em tempo de execução se a consulta não foi realizada; diferenciar referências anteriores de verificações feitas pelo Codex.

## 4. Fase B — Comparação justa das janelas de 7, 15 e 30 dias

### Experimentos

Criar exemplos que usam somente observações disponíveis até `t`, com alvo `cota(t+1)`:

| Experimento | Entradas | Pergunta |
|---|---|---|
| J7 | 7 cotas e 7 chuvas anteriores | Desempenho-base |
| J15 | 15 cotas e 15 chuvas anteriores | Histórico intermediário melhora? |
| J30 | 30 cotas e 30 chuvas anteriores | Histórico mensal melhora? |
| J30-resumo (opcional) | Últimos 7 dias + acumulados/indicadores de 15/30 dias | Resumos superam histórico bruto? |

Definir com precisão as datas de entrada `t-(n-1)` a `t` e o alvo `t+1`, incluindo a questão da disponibilidade operacional da chuva do dia `t`. Se a precipitação medida em `t` não estiver disponível no instante da previsão, testar entradas que terminem em `t-1` e documentar o horizonte resultante.

Regras obrigatórias:

1. **Não atravessar lacunas:** exigir datas consecutivas e medições válidas; chuva ausente não é igual a zero.
2. Construir a **interseção de datas elegíveis** às três janelas para avaliação justa. Se a base de 30 dias reduzir muito os exemplos, reportar a perda e a cobertura por ano.
3. Manter datas cronologicamente separadas entre treino, validação e teste, com limites claramente definidos. Evitar embaralhamento das amostras temporais na avaliação. Ajustar `MinMaxScaler` **somente no treino**; reaplicar no restante. Se escalar `y`, inverter a escala antes de reportar MAE/RMSE.
4. Selecionar hiperparâmetros/arquitetura por **validação**, não pelo teste. Como o teste V1 já foi inspecionado, caracterizar comparações V2 nesse mesmo bloco como **exploratórias**; considerar walk-forward ou novo holdout para avaliação confirmatória.
5. Para cada janela, usar um protocolo comparável de treinamento (sementes documentadas, otimizador, early stopping, orçamento de épocas e conjunto de arquiteturas). A arquitetura MLP 64–32 pode ser um ponto de partida, não pressupor que seja a melhor para todas.
6. Guardar o número de amostras, faixas de datas, parâmetros, tempo/épocas e métricas por experimento. Nunca afirmar que 30 dias são melhores antes de executar e comparar.

## 5. Fase C — A precipitação realmente ajuda?

Para janelas equivalentes, comparar:

- MLP **apenas com cotas**;
- MLP **com cotas + chuva**;
- baseline de **persistência** `cota(t+1) = cota(t)`;
- modelo linear **Ridge** (com features correspondentes, quando aplicável).

Escolher modelos com dados de validação e avaliar com datas de teste idênticas. Reportar MAE, RMSE e R²; adicionalmente, erro por faixas de nível, se houver amostras suficientes. Distinguir **melhoria de validação** de **ganho demonstrado no teste**. Não confundir correlação entre chuva e cota com relação causal.

## 6. Fase D — Gráficos obrigatórios

Gerar e salvar imagens em `resultados/figuras/` ou no diretório de figuras já existente. Usar títulos, datas, unidades e legendas legíveis; sinalizar períodos sem medição.

**Quatro gráficos principais:**

1. **`comparacao_7_dias.png`** — cota real e prevista no mesmo intervalo representativo de teste para J7.
2. **`comparacao_15_dias.png`** — cota real e prevista nas **mesmas datas** para J15.
3. **`comparacao_30_dias.png`** — cota real e prevista nas **mesmas datas** para J30.
4. **`comparacao_janelas_metricas.png`** — comparação das três janelas por **MAE, RMSE e R²** (subgráficos ou visualização legível com escalas adequadas). Indicar qual métrica deve ser minimizada/maximizada.

**Gráficos adicionais recomendados:**

- sobreposição das previsões J7/J15/J30 e cota real para um mesmo episódio de teste;
- erros separados por cota estatisticamente baixa, intermediária e alta;
- gráficos de chuva acumulada em 7/15/30/60 dias e relação com cota/vazão;
- análise dos períodos documentados de estiagem (2020 e 2022) e cheias (2023 e 2024), somente quando os CSVs tiverem cobertura real nesses intervalos;
- curva de perda treino/validação e distribuição/direção dos erros, principalmente nos picos.

Para episódios e limites oficiais, adicionar notas metodológicas quanto a médias diárias versus picos, faltas de dados e comparabilidade de referência. Não usar gráficos esquemáticos como se fossem medições reais.

## 7. Fase E — Cheias, níveis baixos e estiagens

O modelo **já inclui níveis baixos e altos no treino**, desde que haja dados válidos, mas não é um classificador de risco operacional. Expandir análise:

- caracterizar episódios consecutivos acima de P90 e abaixo de P10, definindo percentis **apenas no treino**;
- descrever duração, chuva antecedente, variação de cota e vazão (lembrando que vazão pode ser derivada da cota);
- considerar janelas de chuva acumulada de 7, 15, 30 e opcionalmente 60 dias, sem completar lacunas como zero;
- estimar erros da MLP separadamente em níveis baixos e altos; verificar **subestimação de picos** e **superestimação de mínimos** com resíduos sinalizados;
- contextualizar episódios reais de 2020, 2022, 2023 e 2024 quando houver evidências documentais e cobertura suficiente;
- **não inferir racionamento** a partir de cota baixa: decisões de abastecimento dependem de captação, reservatórios, demanda e medidas operacionais. Registrar risco de escassez apenas como hipótese contextual, nunca como rótulo supervisionado sem dados oficiais.

## 8. Requisitos de engenharia, execução e validação

- Preservar e documentar tratamento ANA: cabeçalhos, separador `;`, vírgula decimal, formato mensal `Cota01..31` / `Chuva01..31`, nível de consistência, status, dias inexistentes e datas repetidas.
- Usar funções reutilizáveis para gerar janelas, fazer splits, treinar modelos e gráficos, sem ocultar a lógica pedagógica no notebook.
- Garantir execução sequencial de todas as células desde kernel limpo e apontar qualquer dependência ou arquivo ausente.
- Exportar resultados numéricos verificáveis a CSV, registrar sementes, versões e caminhos de saída.
- Evitar sobrescrever gráficos ou modelos da V1 sem backups ou nomes de versão.
- Não adicionar dados externos à matriz de treinamento sem experimento explícito e documentação da proveniência.
- Não executar operações destrutivas ou reestruturar o repositório desnecessariamente.

## 9. Regra de atualização do `resultados.md` (obrigatória)

Após executar os experimentos e **obter números reais**, atualizar o arquivo existente **`resultados.md`** (ou seu caminho atual no repositório), mantendo o histórico V1 e criando as seções abaixo:

```markdown
# Resultados e interpretação

## Versão 1 — Referência original
[Preservar metodologia, tabelas, métricas e ressalvas já existentes.]

## Versão 2 — Auditoria das cotas e das fontes
- Data da execução e arquivos consultados
- Unidade encontrada e evidência
- Zero da régua e limitações históricas
- Comparação de maio/2024: CSV x marca de cheia, com explicação sobre agregação

## Versão 2 — Dados utilizáveis e protocolo
- Quantidade de janelas elegíveis 7/15/30
- Datas comuns de treino/validação/teste
- Regras de qualidade, perdas por lacuna e normalização

## Versão 2 — Resultados dos modelos
| Janela | Entradas | Arquitetura | N treino | N validação | N teste | MAE | RMSE | R² |
| ... |
- Baselines e comparações cota versus cota+chuva
- Critério de escolha na validação e ressalva sobre reutilização do teste V1

## Versão 2 — Figuras
- Links relativos aos quatro gráficos principais e figuras extras
- Interpretação breve de cada gráfico

## Versão 2 — Níveis altos, baixos e eventos históricos
- Erro por faixa e resíduos
- Chuva acumulada e episódios relevantes
- Diferenças entre nível estatístico e limiares oficiais
- Limites para inferência de estiagem, inundação e racionamento

## Versão 2 — Conclusões e próximos passos
- O que os resultados realmente demonstram
- O que permanece inconclusivo
- Dificuldades, aprendizados e recomendações para apresentação
```

**Nunca preencher a tabela com métricas fictícias.** Quando uma fase não puder ser executada, registrar **“não executado”** e a causa. Ao final, incluir a lista de arquivos gerados e indicar como reproduzir a execução pelo VS Code.

## 10. Critérios de aceite e ordem sugerida ao Codex

- [ ] V1 preservada e notebook V2 reproduzível.
- [ ] Unidade de cota explicitada com fonte; zero histórico não presumido invariável.
- [ ] Distinção documentada entre média diária e pico instantâneo.
- [ ] Janelas de 7, 15 e 30 dias montadas sem vazamento temporal.
- [ ] Três modelos avaliados nas mesmas datas de teste.
- [ ] Comparação MLP cota vs. cota+chuva realizada ou motivo documentado.
- [ ] Persistência e Ridge comparados em protocolo consistente.
- [ ] Quatro gráficos principais criados a partir de resultados reais.
- [ ] Níveis baixos, altos e extremos discutidos com cautela.
- [ ] **`resultados.md` atualizado** com números, referências, tabelas, figuras e limitações.
- [ ] A apresentação de cinco minutos continua possível, com metodologia e resultados essenciais resumíveis.

### Primeira instrução recomendada ao Codex

> Leia `AGENTS.md`, `README.md`, os documentos em `docs/`, o notebook atual, os CSVs locais e o arquivo existente `resultados.md`. Implemente o `docs/PLANO_MELHORIAS_V2.md` por fases, preservando a V1. Comece conferindo a estrutura e a auditoria dos dados; só treine modelos após validar as janelas e a separação temporal. Ao executar cada experimento, salve as figuras e **atualize `resultados.md`** com dados reais, sem inventar resultados. Indique claramente alterações, verificações pendentes e qualquer etapa que não pôde executar.
