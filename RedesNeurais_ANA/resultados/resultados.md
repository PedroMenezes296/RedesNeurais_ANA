# Resultados e interpretação

**Execução:** 20/10/2026  
**Pergunta do estudo:** o histórico recente de cotas e chuvas ajuda a estimar a cota do Rio Taquari no dia seguinte, e a MLP melhora uma previsão de persistência?

## Dados e método

As exportações da ANA foram expandidas de colunas mensais para datas diárias. Foram mantidos valores com status diário `1` (real); cota e vazão também foram limitadas às observações marcadas como média diária. Quando havia mais de uma observação elegível no mesmo dia, priorizou-se nível de consistência `2` e usou-se a mediana em empates. Valores ausentes não foram preenchidos, e zeros observados de chuva foram preservados.

Após a limpeza, havia 21.193 dias de cota, 28.844 de chuva e 20.103 de vazão. Foram encontradas 18.585 datas em comum entre as três séries. A vazão foi mantida na análise de dados, mas ficou fora do modelo principal porque pode ser derivada da cota.

Cada exemplo usou as sete cotas diárias e, nos modelos com precipitação, as sete chuvas observadas de `t-6` a `t`. O alvo foi a cota de `t+1`. Janelas com dias ausentes ou não consecutivos foram descartadas. A divisão foi cronológica:

| Parte | Amostras | Percentual do total | Período do alvo |
| --- | ---: | ---: | --- |
| Treino efetivo | 12.411 | 68% | 26/04/1943 a 09/03/1993 |
| Validação | 2.190 | 12% | 10/03/1993 a 24/11/2008 |
| Teste final | 3.651 | 20% | 25/11/2008 a 30/04/2026 |
| Desenvolvimento (treino + validação) | 14.601 | 80% | 26/04/1943 a 24/11/2008 |

Assim, os 80% solicitados são o bloco de desenvolvimento; seus 15% finais (12% da base completa) são validação para selecionar a rede. Os 20% mais recentes ficam reservados ao teste. Os scalers foram ajustados apenas no treino efetivo.

## Seleção e desempenho

Entre quatro MLPs, a menor MAE de validação foi da rede com camadas ocultas de 64 e 32 neurônios, usando cota e chuva: MAE `34,419`. A melhor MLP usando apenas cota obteve MAE de validação `35,301`. A diferença favoreceu a inclusão da chuva na validação, mas não basta para concluir que ela melhora o desempenho em outros períodos.

No teste final, a rede selecionada foi comparada com persistência e Ridge usando o mesmo conjunto de entradas:

| Modelo no teste | MAE | RMSE | R² |
| --- | ---: | ---: | ---: |
| MLP 64–32 — cota + chuva | 32,112 | 71,777 | 0,769 |
| Persistência (`cota(t+1) = cota(t)`) | 32,842 | 83,625 | 0,686 |
| Ridge — cota + chuva | 31,375 | 74,371 | 0,752 |

Nesta divisão, a MLP teve MAE aproximadamente 2,2% menor que persistência, além de RMSE menor e R² maior. Isso indica ganho modesto sobre a referência de persistência neste recorte temporal. A Ridge teve a menor MAE, enquanto a MLP teve menor RMSE e maior R²; portanto, não há um vencedor em todas as métricas. A conclusão correta é que a MLP selecionada melhora a persistência neste teste, mas não demonstra superioridade geral sobre modelos mais simples.

## O que os erros e a chuva antecedente sugerem

Os limites de nível baixo e alto foram definidos pelos percentis 10 e 90 do alvo no treino: 96 e 358, respectivamente, na unidade não confirmada do arquivo. No teste, a MAE da MLP foi 113,651 nos dias acima do percentil 90, 26,934 abaixo do percentil 10 e 20,795 na faixa central. O erro é bem maior nos níveis estatisticamente altos, indicando dificuldade para representar os picos. Esses percentis não são cotas oficiais de inundação ou estiagem.

Também comparamos a chuva acumulada nos sete dias completos anteriores a cada alvo de teste. A mediana foi 70,9 nos dias de nível alto, 15,4 na faixa central e 7,3 nos dias de nível baixo; havia precipitação antecedente completa para todos os dias desses grupos. Isso mostra uma associação descritiva entre níveis altos e maior chuva antecedente nesta base e janela. Não demonstra causalidade: há apenas uma estação pluviométrica, os dias de um mesmo evento são dependentes e outras condições da bacia não foram modeladas.

## Conclusões para a pergunta do projeto

- O histórico recente de cota oferece uma previsão de referência forte; a MLP com cota e chuva superou a persistência nas três métricas deste teste, com ganho mais claro no RMSE e no R².
- A chuva antecedente ajudou a MLP escolhida a obter a menor MAE de validação entre as configurações testadas. Como o teste reporta apenas o modelo selecionado e os baselines, ainda não há uma comparação final no teste entre MLP com chuva e MLP sem chuva; a contribuição isolada da precipitação permanece inconclusiva.
- A rede apresenta erros muito maiores nos níveis altos. O modelo pode suavizar ou subestimar picos, então a média geral de erro não descreve sozinha o comportamento em períodos críticos.
- Os resultados valem para esta estação, regra de qualidade, janela de sete dias e único corte temporal testado. Eles não validam alertas em tempo real nem generalização para outras bacias, rios ou reservatórios.

## O que falta para fechar a entrega

1. Confirmar nos metadados da ANA a unidade e a referência da cota antes de rotular números e gráficos.
2. Confirmar com o professor se o notebook precisa ser executado no Google Colab ou se a execução em VS Code é aceita.
3. Confirmar que a chuva diária de `t` estaria disponível no fim do próprio dia `t`, como pressupõe a previsão deste experimento. Se não estiver, as entradas precisam terminar em `t-1` e o modelo deve ser recalculado.
4. Preencher o nome do estudante no roteiro de apresentação e atualizar os slides com os resultados desta divisão 80/20.

O notebook, as tabelas CSV e as figuras que sustentam este texto estão em `notebooks/analise_redes_neurais.ipynb` e nesta pasta `resultados/`. A unidade de precipitação também deve ser rotulada a partir dos metadados da estação, sem assumir unidade que não conste na fonte consultada.


## Vers?o 2 ? Auditoria das cotas e das fontes

**Execu??o:** 20/10/2026. **Arquivos locais consultados:** `dados/86720000_Cotas.csv`, `dados/86720000_Vazoes.csv` e `dados/2951010_Chuvas.csv`. Os cabe?alhos e metadados ANA indicam `NivelConsistencia` (1 bruto, 2 consistido), `MediaDiaria` (0 instant?nea, 1 m?dia di?ria), `TipoMedicaoCotas` e os status di?rios (0 branco, 1 real, 2 estimado, 3 duvidoso, 4 r?gua seca). O pipeline manteve apenas status 1, m?dia di?ria para cota e vaz?o, priorizou consist?ncia 2 e usou mediana em empates.

A unidade n?o est? declarada nos metadados desses CSVs. Os dados e m?tricas V2 s?o, portanto, apresentados na **unidade num?rica da exporta??o**, ainda n?o confirmada para esta esta??o. O estudo do [SGB sobre zeros ortom?tricos](https://rigeo.sgb.gov.br/bitstream/doc/25518/1/marcuzzo_silva_Mendes_zero%20ortometrico_artigo.pdf) documenta 28,160 m para o zero da r?gua de Encantado; isso n?o prova que o referencial tenha permanecido invari?vel ao longo de toda a s?rie. A unidade n?o foi convertida nos c?lculos.

Em 02/05/2024, a exporta??o de cotas tem uma leitura de 17:00 com `Cota02=2314`, status 1 e `MediaDiaria=0`; a linha `MediaDiaria=1` n?o tem valor di?rio eleg?vel (`Cota02Status=0`). O [levantamento do SGB](https://rigeo.sgb.gov.br/bitstream/doc/24939.10/6/2024-11-04_nota_tecnica_levantamento_cheia_2024_v.8.pdf) registra uma marca nivelada de 23,14 m para o evento. A semelhan?a num?rica ap?s divis?o por 100 n?o transforma a leitura instant?nea numa m?dia di?ria nem comprova por si s? a equival?ncia do referencial. O arquivo de chuva n?o cont?m linha mensal de maio/2024 (`Data=01/05/2024`), ent?o n?o foi poss?vel comparar a precipita??o desse dia.

Como exemplo hist?rico de limiar operacional, o boletim do [SGB de 12/05/2024](https://www.sgb.gov.br/sace/boletins/Taquari/20240512_02-20240512%20-%20032619.pdf) lista 1.200 cm como cota de inunda??o em Encantado. Esse valor ? contextual e datado; n?o foi sobreposto ?s m?dias di?rias nem tratado como limiar universal. O [SACE Taquari](https://www.sgb.gov.br/sace/taquari_apresentacao.php) re?ne boletins e refer?ncias de eventos. Os percentis abaixo s?o exclusivamente estat?sticos.

## Vers?o 2 ? Dados utiliz?veis e protocolo

As janelas completas eleg?veis antes da interse??o foram 18.252 (J7), 17.162 (J15) e 15.673 (J30) para chuva at? `t`; para chuva at? `t?1`, 18.245, 17.155 e 15.667. A interse??o comum usada em todas as janelas teve 15.673 datas no protocolo at? `t` e 15.667 no protocolo at? `t?1`. Assim, a exig?ncia de J30 retirou 2.579 das amostras eleg?veis de J7 (14,1%) no protocolo at? `t`; no protocolo at? `t?1`, retirou 2.578 (14,1%). As contagens e cobertura anual completas est?o em [`cobertura_anual.csv`](v2/cobertura_anual.csv).

A avalia??o mant?m os limites cronol?gicos da V1: treino at? 09/03/1993; valida??o de 10/03/1993 a 24/11/2008; teste a partir de 25/11/2008. Depois da regra de dados v?lidos, o teste efetivamente come?a em 28/11/2008 para chuva at? `t` (2.251 alvos) e em 29/11/2008 para chuva at? `t?1` (2.247 alvos); ambos terminam em 30/04/2026. Todas as tr?s janelas dentro de cada protocolo usam exatamente essas mesmas datas. O n?mero de amostras ? menor que na V1 porque cada exemplo agora tamb?m precisa pertencer ? interse??o eleg?vel de J7, J15 e J30.

As entradas de cota usam `t?(n?1)?t`; a chuva usa esse mesmo intervalo at? `t`, ou `t?n?t?1` na variante conservadora. O alvo permanece cota em `t+1`. Nenhuma janela atravessa datas ausentes; chuva ausente n?o vira zero. Os scalers MinMax foram ajustados apenas no treino e as m?tricas usam o alvo revertido para a unidade num?rica original. A semente foi 42; arquitetura selecionada por menor MAE de valida??o entre MLP 32 e MLP 64?32. O hist?rico de candidatos, ?pocas e parti??es est? nos CSVs de V2.

## Vers?o 2 ? Resultados dos modelos

A avalia??o reutiliza o per?odo de teste j? consultado na V1 e ? **explorat?ria**, n?o confirmat?ria. Valores na tabela s?o `MAE / RMSE / R?`; MAE e RMSE devem ser minimizadas e R? maximizado. A unidade num?rica da cota n?o foi confirmada.

| Protocolo | Janela | N teste | MLP s? cota | MLP cota + chuva | Ridge cota + chuva | Persist?ncia |
| --- | ---: | ---: | --- | --- | --- | --- |
| Chuva at? `t` | 7 | 2.251 | 29,239 / 70,407 / 0,726 | 31,515 / 70,140 / 0,728 | 29,694 / 71,964 / 0,714 | 31,013 / 78,737 / 0,658 |
| Chuva at? `t` | 15 | 2.251 | 29,317 / 74,532 / 0,693 | 32,358 / 71,406 / 0,718 | 29,667 / 71,605 / 0,717 | 31,013 / 78,737 / 0,658 |
| Chuva at? `t` | 30 | 2.251 | 29,887 / 73,152 / 0,704 | 34,921 / 74,764 / 0,691 | 30,006 / 71,738 / 0,716 | 31,013 / 78,737 / 0,658 |
| Chuva at? `t?1` | 7 | 2.247 | 29,757 / 71,020 / 0,724 | 32,043 / 72,676 / 0,711 | 28,751 / 75,287 / 0,690 | 31,519 / 79,605 / 0,654 |
| Chuva at? `t?1` | 15 | 2.247 | 29,701 / 75,240 / 0,691 | 32,344 / 74,361 / 0,698 | 29,055 / 75,067 / 0,692 | 31,519 / 79,605 / 0,654 |
| Chuva at? `t?1` | 30 | 2.247 | 30,552 / 73,919 / 0,701 | 35,605 / 77,448 / 0,672 | 29,754 / 75,129 / 0,691 | 31,519 / 79,605 / 0,654 |

A MLP somente com cotas teve menor MAE que a persist?ncia nas tr?s janelas e nos dois protocolos. A MLP com chuva n?o reduziu MAE em rela??o ? MLP somente com cota em nenhuma janela; tamb?m teve MAE maior que a persist?ncia nestes cortes. A Ridge com cotas e chuva teve menor MAE que a MLP correspondente, mas as m?tricas n?o indicam um vencedor ?nico em todas as dimens?es. A tabela completa, incluindo Ridge somente com cota, est? em [`metricas_modelos.csv`](v2/metricas_modelos.csv).

Nesta divis?o, aumentar a janela de 7 para 30 dias n?o melhorou a MAE: para MLP somente com cota, a MAE passou de 29,239 para 29,887 (chuva at? `t`) e de 29,757 para 30,552 (chuva at? `t?1`). A compara??o n?o demonstra benef?cio da chuva; s?o resultados explorat?rios em uma ?nica esta??o e um ?nico corte temporal.

## Vers?o 2 ? Figuras

As figuras principais foram produzidas do teste com chuva at? `t`, em datas coincidentes entre J7/J15/J30:

- [Compara??o J7](v2/figuras/comparacao_7_dias.png)
- [Compara??o J15](v2/figuras/comparacao_15_dias.png)
- [Compara??o J30](v2/figuras/comparacao_30_dias.png)
- [M?tricas das janelas](v2/figuras/comparacao_janelas_metricas.png)

Tamb?m foram geradas sobreposi??o das tr?s previs?es, res?duos, chuva acumulada de 7/15/30/60 dias e tabelas CSV correspondentes, todas sob `resultados/v2/`.

## Vers?o 2 ? N?veis altos, baixos e eventos hist?ricos

Os percentis calculados apenas no treino foram P10=96 e P90=358, na unidade num?rica da exporta??o. Para a MLP J7 com cotas+chuva at? `t`, a MAE no teste foi 108,762 acima de P90 (239 alvos), 28,139 abaixo de P10 (42 alvos) e 22,215 na faixa central (1.970 alvos). O erro m?dio `previsto ? observado` foi ?46,748 na faixa alta e ?26,886 na faixa baixa: h? subestima??o m?dia em ambos os extremos neste corte, n?o superestima??o m?dia dos m?nimos. Percentis n?o s?o cotas oficiais.

Os epis?dios consecutivos acima/abaixo dos percentis, com dura??o e extremos de cota, est?o em [`episodios_p10_p90.csv`](v2/episodios_p10_p90.csv); a tabela inclui cobertura de 2020, 2022, 2023 e 2024. Chuva acumulada antecedente e gr?ficos descritivos est?o nos arquivos `chuva_antecedente_teste.csv` e `figuras/chuva_acumulada_niveis.png`. A vaz?o foi preservada na s?rie di?ria, mas n?o entrou no modelo e n?o foi usada para afirmar medi??o independente da cota. A an?lise n?o rotula estiagem, inunda??o ou racionamento operacionalmente.

## Vers?o 2 ? Conclus?es e pr?ximos passos

- Neste teste explorat?rio, J7 foi ligeiramente melhor que J30 em MAE para as MLPs; a janela longa n?o trouxe ganho demonstrado.
- A precipita??o n?o reduziu a MAE da MLP neste corte. Correla??o descritiva n?o estabelece causalidade, e a esta??o pluviom?trica n?o representa toda a bacia.
- O erro em n?veis altos foi muito maior e teve vi?s m?dio de subestima??o. A m?dia di?ria e os dados instant?neos n?o s?o intercambi?veis; n?o h? evid?ncia de previs?o de picos instant?neos.
- A unidade da cota, o referencial hist?rico e a disponibilidade operacional da chuva de `t` continuam como limita??es. Seria necess?ria avalia??o temporal futura/walk-forward antes de qualquer conclus?o confirmat?ria ou operacional.

**Arquivos e reprodu??o:** notebook execut?vel [`analise_redes_neurais_v2.ipynb`](../notebooks/analise_redes_neurais_v2.ipynb); checkpoint V1 [`analise_redes_neurais_v1.ipynb`](../notebooks/analise_redes_neurais_v1.ipynb); sa?das em `resultados/v2/`, com lista em [`manifesto_arquivos.txt`](v2/manifesto_arquivos.txt). No VS Code, abrir o notebook V2, selecionar o kernel `.venv` e executar todas as c?lulas em ordem.


## Versão 3 — Objetivos e protocolo

Execucao em 20/10/2026 com as tres exportacoes ANA locais. A V3 estima cota media diaria em t+1, compara janelas de cotas de 7/15/30 dias e chuva acumulada nos 7/15/30 dias anteriores. Foram executados protocolos com chuva observada ate t e ate t-1. O teste coincide com periodo ja consultado nas versoes anteriores; os resultados sao exploratorios, nao confirmacao independente.

Seed=42; Adam (taxa 0,001); perda Huber; ate 120 epocas; batch_size=128; EarlyStopping com paciencia 12. Candidatos: neuronio linear, MLP 32 e MLP 64-32; selecao por MAE de validacao. MinMaxScaler ajustado somente no treino. N treino/validacao/teste: 11.660/1.762/2.251 (chuva t) e 11.659/1.761/2.247 (chuva t-1).

## Versão 3 — Auditoria de dados e unidades

Apos status real e regra MediaDiaria=1 para cota/vazao, restaram 21.193 dias de cota, 20.103 de vazao e 28.844 de chuva. O cabecalho ANA define MediaDiaria=0 como instantanea e MediaDiaria=1 como media diaria. A unidade da cota e a estabilidade historica do zero da regua nao estao confirmadas pelos campos exportados. Em 02/05/2024, 2314 ocorre com status real e MediaDiaria=0; a linha de media diaria nao tem valor elegivel. O valor instantaneo nao foi inserido na serie diaria. A exportacao nao tem chuva nesse dia.

## Versão 3 — Episodios historicos altos e baixos

P10=96 e P90=358, calculados apenas no treino, definiram faixas estatisticamente baixa/alta. Foram agrupados 618 episodios altos e 161 episodios baixos consecutivos; sao grupos temporais, nao rotulos oficiais nem observacoes independentes.

Medianas de chuva acumulada completa antes do inicio do episodio (unidade numerica da exportacao ANA nao confirmada):

| Grupo | Episodios | Duracao mediana (dias) | Chuva 7d | Chuva 15d | Chuva 30d | N com chuva 30d |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Alta | 618 | 3.0 | 44.1 | 77.8 | 139.3 | 538 |
| Baixa | 161 | 6.0 | 3.6 | 27.7 | 73.5 | 147 |

A vazao e complementar e pode ser derivada da cota. A chuva vem de uma unica estacao e nao representa toda a bacia; associacao descritiva nao demonstra causalidade. Cota baixa nao comprova seca/estiagem nem necessidade de racionamento.

## Versão 3 — Entradas de chuva acumulada e janelas 7/15/30

Cada protocolo usa a intersecao comum entre J7/J15/J30. Chuva ate t: 15.673 datas; exigir J30 removeu 2.579 exemplos originalmente elegiveis de J7 (14,1%). Chuva ate t-1: 15.667 datas comuns. O teste vai de 28/11/2008 a 30/04/2026 (N=2.251) e de 29/11/2008 a 30/04/2026 (N=2.247), respectivamente. Janelas nao atravessam lacunas; chuva ausente nao e zero.

## Versão 3 — Arquiteturas e parametros explicados

Seed 42 inicia os sorteios pseudoaleatorios. O neuronio linear tem saida Dense linear sem camada oculta; MLP 32 tem uma camada de 32 neuronios; MLP 64-32 tem duas camadas ocultas. Neuronios nao sao o numero de pesos treinaveis. Uma epoca percorre o treino; lote 128 atualiza pesos em grupos. Adam otimiza, Huber e a perda e EarlyStopping restaura pesos segundo validacao. MAE/RMSE menores sao melhores; R2 maior e melhor. Todos os candidatos e epocas estao em [selecao_modelos_validacao.csv](v3/selecao_modelos_validacao.csv).

## Versão 3 — Resultados da regressao

Tabela do protocolo de chuva ate t; valores na unidade numerica exportada da cota, ainda nao confirmada. [metricas_modelos.csv](v3/metricas_modelos.csv) contem todas as 36 linhas dos dois protocolos.

| Janela | Modelo | Entradas | MAE | RMSE | R2 | N teste |
| ---: | --- | --- | ---: | ---: | ---: | ---: |
| 7 | MLP (64, 32) cota | cota | 29.239 | 70.407 | 0.726 | 2251 |
| 7 | Persistência | cota(t) | 31.013 | 78.737 | 0.658 | 2251 |
| 7 | Ridge cota | cota | 27.760 | 74.015 | 0.697 | 2251 |
| 7 | MLP (32,) cota+chuva | cota+chuva | 31.515 | 70.140 | 0.728 | 2251 |
| 7 | Persistência | cota(t) | 31.013 | 78.737 | 0.658 | 2251 |
| 7 | Ridge cota+chuva | cota+chuva | 29.694 | 71.964 | 0.714 | 2251 |
| 15 | MLP (64, 32) cota | cota | 29.317 | 74.532 | 0.693 | 2251 |
| 15 | Persistência | cota(t) | 31.013 | 78.737 | 0.658 | 2251 |
| 15 | Ridge cota | cota | 28.083 | 73.832 | 0.699 | 2251 |
| 15 | MLP () cota+chuva | cota+chuva | 30.474 | 72.188 | 0.712 | 2251 |
| 15 | Persistência | cota(t) | 31.013 | 78.737 | 0.658 | 2251 |
| 15 | Ridge cota+chuva | cota+chuva | 29.667 | 71.605 | 0.717 | 2251 |
| 30 | MLP (32,) cota | cota | 29.887 | 73.152 | 0.704 | 2251 |
| 30 | Persistência | cota(t) | 31.013 | 78.737 | 0.658 | 2251 |
| 30 | Ridge cota | cota | 28.284 | 73.721 | 0.700 | 2251 |
| 30 | MLP () cota+chuva | cota+chuva | 30.413 | 72.414 | 0.710 | 2251 |
| 30 | Persistência | cota(t) | 31.013 | 78.737 | 0.658 | 2251 |
| 30 | Ridge cota+chuva | cota+chuva | 30.006 | 71.738 | 0.716 | 2251 |

Neste corte, MLP somente com cotas teve menor MAE que MLP com chuva nas tres janelas. A MLP com chuva teve RMSE menor que persistencia, mas MAE maior. Ridge com cota e chuva teve menor MAE que a MLP com chuva. Janelas maiores nao mostraram melhoria consistente. Figuras e curvas estao em [figuras](v3/figuras/); protocolo t-1 no CSV de metricas.

## Versão 3 — Classificacao por limites

Como unidade/referencial nao foram validados, a classificacao usa regras estatisticas: baixa < P10=96, usual entre P10 e P90, alta > P90=358. O boletim SGB de 12/05/2024 lista atencao 500 cm, alerta 900 cm e inundacao 1.200 cm em Encantado; esses limites contextuais nao foram aplicados ao historico. >1.500 cm e corte experimental, nao classe de desastre. [Boletim SGB](https://www.sgb.gov.br/sace/boletins/Taquari/20240512_02-20240512%20-%20032619.pdf), [SACE Taquari](https://www.sgb.gov.br/sace/taquari_apresentacao.php).

No teste J7 com chuva ate t, houve 42 casos baixos, 1.973 usuais e 236 altos.

| Classe | Precisao | Recall | F1 | Suporte |
| --- | ---: | ---: | ---: | ---: |
| Baixa (<P10 treino) | 1.000 | 1.000 | 1.000 | 42 |
| Usual (P10-P90) | 0.974 | 0.965 | 0.969 | 1973 |
| Alta (>P90 treino) | 0.725 | 0.784 | 0.754 | 236 |

A acuracia total foi 0.946; devido ao desbalanceamento, suporte e recall da classe alta importam mais que acuracia isolada. O resultado e exploratorio, nao valida alerta. Consulte [matriz de confusao](v3/matriz_confusao_faixas.csv) e [metricas por classe](v3/metricas_classificacao_faixas.csv).

## Versão 3 — Cheias e niveis baixos persistentes

Na MLP J7 com chuva, os alvos observados >=P90 tiveram MAE 108,762 (N=239) e vies previsto-observado -46,748, indicando subestimacao media dos niveis estatisticamente altos. Abaixo/igual a P10: MAE 28,139 (N=42), vies -26,886; faixa central: MAE 22,215 (N=1.970), vies +3,210. Isto nao e previsao de inundacao nem diagnostico de estiagem/racionamento.

## Versão 3 — Limitacoes, conclusoes e proximos passos

Estudo retrospectivo e exploratorio num corte ja consultado por V1/V2. Disponibilidade operacional da chuva de t permanece hipotese; t-1 e comparacao conservadora. Unidade e referencial historico da cota seguem pendentes. Medias diarias nao representam maximas instantaneas. Confirmacao requer revisar metadados e avaliar periodo futuro/walk-forward.

Artefatos: [notebook V3](../notebooks/analise_redes_neurais_v3.ipynb), [cobertura](v3/cobertura_e_splits.csv), [episodios](v3/episodios_p10_p90.csv), [chuva por episodio](v3/episodios_com_chuva.csv); demais tabelas e figuras em resultados/v3/. Execute no VS Code com kernel .venv, em ordem.
