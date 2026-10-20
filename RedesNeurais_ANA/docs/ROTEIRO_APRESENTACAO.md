# Roteiro da apresentação — 5 minutos

## 1. Capa e pergunta — 20 s

**Previsão da cota do Rio Taquari com redes neurais**  
Disciplina de Redes Neurais — [nome do estudante]

“Investigamos se o histórico de cotas e a chuva observada ajudam a estimar a cota do dia seguinte em Encantado. O estudo é retrospectivo, não um sistema de alerta.”

## 2. Dados e preparação — 50 s

“Usamos exportações do HidroWeb/ANA: cotas e vazões da estação 86720000 e chuva da estação 2951010. Expandimos as colunas mensais para observações diárias, mantivemos medições com status real e médias diárias de cota, priorizamos dados consistidos e preservamos lacunas e zeros de chuva observados.”

“A formulação gerou 18.252 janelas completas de sete dias. A divisão cronológica reservou 80% para desenvolvimento — 12.411 treino e 2.190 validação — e os 20% finais, 3.651 exemplos, para teste.”

## 3. Método — 1 min 20 s

“Para cada data `t`, usamos cotas dos sete dias até o fim de `t` para prever `cota(t+1)`. Comparamos também entradas que incluem a chuva dos mesmos dias. Janelas com lacunas foram descartadas. A normalização MinMax foi ajustada apenas no treino.”

“Na validação comparamos quatro MLPs: 32 ou 64–32 neurônios ocultos, com cota apenas ou cota e chuva. A menor MAE foi da MLP 64–32 com chuva, 34,42. Persistência e Ridge foram referências.”

## 4. Resultados — 1 min 40 s

Apresentar `resultados/observado_previsto_teste.png`, `resultados/metricas_teste.csv` e `resultados/chuva_antecedente_niveis.png`.

“No teste de novembro de 2008 a abril de 2026, a MLP teve MAE 32,11, RMSE 71,78 e R² 0,769. Persistência teve 32,84, 83,63 e 0,686. A Ridge teve a menor MAE, 31,37, mas RMSE e R² piores que a MLP. Portanto, a MLP superou persistência neste período, mas não venceu a Ridge em todas as métricas.”

“Nos dias de nível alto, a mediana de chuva dos sete dias anteriores foi 70,9; na faixa central foi 15,4 e nos dias de nível baixo foi 7,3. É uma associação descritiva, não prova causalidade.”

## 5. Conclusões e limites — 50 s

“A MLP com chuva teve o melhor resultado de validação e modestamente superou a persistência no teste. A contribuição isolada da chuva ainda é inconclusiva, pois o teste reporta o modelo selecionado e as referências, não uma ablação final entre MLPs. Os erros foram maiores nos níveis estatisticamente altos: MAE 113,65, contra 20,80 na faixa central. A unidade da cota e a disponibilidade da chuva de `t` ainda precisam ser confirmadas. Os percentis não são alertas oficiais e o resultado não valida operação em tempo real.”

## Antes de apresentar

- Preencher o nome do estudante.
- Confirmar a unidade/referência da cota nos metadados e se o professor exige Colab.
- Atualizar este roteiro se o notebook for reexecutado com outros dados ou parâmetros.
