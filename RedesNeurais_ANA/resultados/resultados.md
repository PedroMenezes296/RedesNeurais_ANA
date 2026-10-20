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
