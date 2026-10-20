# Requisitos da disciplina — checklist de conformidade

## 1. Enunciado informado pelo aluno
**Objetivo:** construir um modelo de **classificação ou regressão** para uma base de dados. O enunciado cita o repositório UCI, porém **o professor confirmou ao aluno que essa origem era somente uma sugestão**; nosso trabalho pode utilizar séries da ANA.

### Tarefas obrigatórias
- [ ] Carregar os dados em um notebook Python (o enunciado menciona Google Colab; projeto será desenvolvido no VS Code como `.ipynb`).
- [ ] Verificar se as colunas foram carregadas corretamente.
- [ ] Normalizar dados no intervalo **0 a 1**.
- [ ] Dividir os dados em **treino e teste**.
- [ ] Criar o modelo de rede neural.
- [ ] Treinar com dados de treino.
- [ ] Avaliar com os dados de teste.

> Nota de execução: divisão cronológica e conjunto separado de validação são **adaptações metodológicas nossas** para séries temporais; o professor não exigiu explicitamente validação, mas ela serve para escolher arquitetura sem usar o conjunto de teste. O enunciado cita Colab; confirmar com o professor se o ambiente de execução for obrigatório na entrega, pois o arquivo `.ipynb` é compatível.

## 2. Apresentação individual — 5 minutos
O professor pediu:

| Parte | Conteúdo exigido | Sugestão de tempo |
| --- | --- | --- |
| Capa | Nome do trabalho e nome do aluno | 20 s |
| Introdução | Problema e base: quantidade de **instâncias/amostras** e **características/colunas** | 50 s |
| Metodologia | **Como foi definida a melhor arquitetura** | 1 min 20 s |
| Resultados e discussão | Arquitetura construída e **resultados em dados de teste** | 1 min 40 s |
| Conclusões | **Correlação ou acurácia**, dificuldades e principais aprendizados | 50 s |

Para nosso problema de **regressão**, o desempenho deve usar MAE, RMSE, R² e, se útil, coeficiente de correlação (em vez de acurácia classificatória). Não confundir correlação com erro absoluto ou precisão operacional. As contagens de instâncias e características **devem sair do dataset final supervisionado** depois da preparação; contar linhas mensais dos CSVs originais não responde a essa exigência.

## 3. Evidências mínimas a produzir
- [ ] Mostrar `head()`, nomes das colunas, tipos, quantidades de registros e tratamento de ausências.
- [ ] Descrever as features (`X`) e o alvo (`y = cota(t+1)`).
- [ ] Mostrar separação cronológica entre treino, validação e teste.
- [ ] Mostrar normalização ajustada no treino e aplicada ao restante.
- [ ] Registrar a arquitetura (número de entradas, camadas, neurônios, funções de ativação, parâmetros), otimizador, perda e quantidade de épocas.
- [ ] Explicar **o critério** de escolha entre modelos (métrica no conjunto de validação) e não apenas mostrar o vencedor.
- [ ] Mostrar curva de perda e gráfico real × previsto em teste.
- [ ] Apresentar as métricas de teste, comparar com baseline e interpretar erros.
- [ ] Descrever dificuldades reais, limitações dos dados e aprendizados.

## 4. O que o notebook de referência do professor exemplifica
Arquivo: `referencias/dl_class03_LinearRegressionSingleNeuron.ipynb`.

O exemplo apresenta `pandas.read_csv`, inspeção com `data.info`, separação treino/teste, arquitetura Keras `Sequential` com **um neurônio** (`Dense(1)`), compilação, treino, gráfico do MSE, inspeção de pesos e comparação gráfico real × previsto. Emprega a base Algerian Forest Fires e é **referência didática**, não solução específica para dados temporais. Nossa implementação deverá adaptar principalmente a **divisão cronológica** e a **normalização 0–1** exigida no trabalho.

## 5. Pendências que não bloqueiam a estrutura inicial
- Confirmar se o professor exige **execução no Colab**, ou se aceita desenvolver e apresentar o `.ipynb` no VS Code.
- Definir datas concretas de treino/validação/teste após conhecer lacunas.
- Confirmar unidades e quantidade real de instâncias treináveis antes dos slides.
