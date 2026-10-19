# Previsão do nível do Rio Taquari com redes neurais artificiais

> **Projeto acadêmico da disciplina de Redes Neurais** — estudo de regressão supervisionada a partir de séries históricas hidrológicas da Agência Nacional de Águas e Saneamento Básico (ANA). Desenvolvimento local no **VS Code com Jupyter Notebook**.

## 1. Visão geral

O objetivo é investigar se medições históricas do **nível (cota) do Rio Taquari na estação de Encantado (RS)** e da **precipitação registrada em estação pluviométrica** permitem estimar a **cota do rio no dia seguinte**. O estudo usa uma rede neural do tipo **MLP (perceptron multicamadas)** como modelo inicial de **regressão**. A vazão será avaliada como variável complementar, caso a qualidade e a disponibilidade dos dados justifiquem seu uso.

Este é um **projeto de pesquisa e de aprendizagem**, não um sistema de previsão operacional nem de emissão de alertas. A proposta é relacionada ao Projeto Integrador 5 (PI5) de monitoramento hidrológico, mas a implementação, os resultados e a avaliação nesta disciplina são independentes da infraestrutura do PI5. Não é correto interpretar a cota medida no rio como percentual da capacidade de uma barragem.

**Situação inicial:** o repositório contém a estrutura e um notebook de orientação; **nenhuma base foi inserida, limpa, analisada ou treinada**. As métricas só serão conhecidas após executar os experimentos sobre dados reais.

## 2. Compatibilidade com a atividade da disciplina

O enunciado solicita: carregar e conferir os dados, normalizar entradas entre **0 e 1**, separar treino e teste, criar e treinar uma rede neural, avaliar no teste e apresentar os resultados em cinco minutos. O professor confirmou que o repositório UCI foi **sugerido, não obrigatório**, permitindo utilizar as séries da ANA.

| Exigência | Como será atendida |
| --- | --- |
| Classificação ou regressão | **Regressão**: estimar a cota no próximo dia |
| Carregamento e conferência | Leitura dos CSVs, inspeção de cabeçalhos, colunas, tipos e datas |
| Normalização 0–1 | `MinMaxScaler`, **ajustado somente nos dados de treino** |
| Divisão dos dados | Treino, validação e teste em **ordem cronológica** |
| Modelo de rede neural | MLP com camadas densas e saída numérica |
| Treinamento | Ajuste de pesos somente com amostras do conjunto de treino |
| Avaliação | MAE, RMSE e R² no conjunto de teste; gráficos de valores reais e previstos |
| Escolha da arquitetura | Comparar configurações usando validação, sem escolher a partir do teste |
| Apresentação | Capa, introdução, metodologia, resultados/discussão e conclusões (5 min) |

**Observação:** no material da disciplina consta Google Colab; o `.ipynb` é compatível com Colab e VS Code, mas a aceitação da execução **local** deve ser confirmada com o professor se ele exigir a plataforma especificamente.

## 3. Bases de dados previstas

**Coloque os arquivos originais manualmente dentro da pasta `dados/`** (ela vem intencionalmente vazia). Não renomeie nem altere os CSVs brutos.

| Arquivo esperado | Estação | Papel |
| --- | --- | --- |
| `86720000_Cotas.csv` | 86720000 | Cota histórica; alvo principal e entradas defasadas |
| `2951010_Chuvas.csv` | 2951010 | Precipitação observada; entradas defasadas e acumulados |
| `86720000_Vazoes.csv` | 86720000 | Vazão histórica; análise e experimento opcional |

Fonte: exportações do **HidroWeb / ANA**. Consulte https://www.snirh.gov.br/hidroweb/serieshistoricas . As estações de chuva e cota são distintas; a chuva local pode não representar toda a bacia a montante. As observações precisam de alinhamento temporal antes da modelagem.

**Atenção ao formato:** os CSVs da ANA podem conter textos antes do cabeçalho, separador `;`, codificação `cp1252`, números com vírgula decimal e **uma linha por mês com colunas de dias 01 a 31**. Será necessário expandir as colunas diárias e validar datas, valores e indicadores de consistência/status. Valores `0` de precipitação não significam ausência de medição; lacunas não podem ser convertidas automaticamente em zero.

O relatório de pesquisa que fundamenta a proposta encontrou preliminarmente **18.531 datas com valores das três séries após critérios exploratórios de qualidade**, mas esse número **não representa 18.531 amostras treináveis**: lacunas e janelas temporais completas ainda precisam ser verificadas. Não é uma métrica recalculada por este projeto.

## 4. Pergunta de pesquisa e hipótese

**Pergunta principal:** em que medida o histórico diário de cota e chuva permite prever a cota do próximo dia em Encantado?

Hipótese a testar: adicionar chuva observada aos níveis recentes pode melhorar a previsão da cota futura. Trata-se de uma hipótese, não de uma conclusão. Também queremos estudar os erros da rede nos períodos de níveis estatisticamente altos e baixos, sem tratá-los automaticamente como enchentes ou racionamento.

### Problema supervisionado

Para uma data `t`, a entrada candidata contém **cotas e chuvas observadas até `t`** (por exemplo, os últimos sete dias) e o alvo é **`cota(t+1)`**. A observação de `t` só pode ser usada se já estiver disponível no instante em que a previsão seria emitida; caso contrário, o limite das entradas deve recuar para `t-1`.

Etapas experimentais previstas:

1. **Baseline de persistência:** prever a cota de amanhã como a cota mais recente.
2. **MLP apenas com histórico de cotas:** medir quanto a rede aprende do passado do nível.
3. **MLP com cotas e chuvas:** avaliar se a chuva acrescenta informação útil.
4. **Opcional:** acrescentar vazão histórica e/ou testar um horizonte de três dias.

A arquitetura ilustrativa inicial é `Dense(64, relu) → Dense(32, relu) → Dense(1)`; será **comparada com alternativas**, não declarada previamente como a melhor. Possíveis funções de perda: MAE, MSE ou Huber; otimizador Adam e `EarlyStopping` na validação.

## 5. Estrutura de diretórios

```text
RedesNeurais_ANA/
├── dados/                         # CSVs originais inseridos pelo estudante (vazio ao receber)
│   └── .gitkeep
├── notebooks/
│   └── analise_redes_neurais.ipynb # Notebook inicial, sem análises executadas
├── resultados/                    # Gráficos, base tratada e métricas futuras
│   └── .gitkeep
├── modelos/                       # Pesos e modelos treinados futuros
│   └── .gitkeep
├── .vscode/
│   └── settings.json              # Configuração local de ambiente Python/Jupyter
├── .gitignore
├── requirements.txt
└── README.md
```

Os arquivos `.gitkeep` são **arquivos vazios de controle de versão**; não são CSVs nem resultados.

## 6. Requisitos e instalação (Windows + VS Code)

Instale **Python 3.11 (64 bits)**, **VS Code** e as extensões **Python** e **Jupyter**, publicadas pela Microsoft. O Python 3.11 é uma escolha de compatibilidade para o ambiente proposto, não uma exigência do projeto. A instalação do TensorFlow depende do sistema/versão; se surgir conflito, verifique a compatibilidade atual e ajuste a versão do Python antes de continuar.

Abra a pasta `RedesNeurais_ANA` no VS Code e use o terminal PowerShell:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Se o Windows não encontrar `py -3.11`, verifique a instalação e a versão do Python (`py --list`). Se a ativação do PowerShell estiver bloqueada por política, é possível chamar diretamente `.venv\Scripts\python.exe` nos comandos ou selecionar esse interpretador no VS Code; não é necessário mudar as políticas globais do computador.

Abra `notebooks/analise_redes_neurais.ipynb`, clique em **Select Kernel** e escolha o interpretador **`.venv`**. Execute uma célula de cada vez. Use a pasta raiz como diretório de trabalho (o notebook de orientação explica como localizar os CSVs independentemente do diretório corrente).

### Bibliotecas

- `pandas`, `numpy`: preparação e análise dos dados;
- `matplotlib`: gráficos;
- `scikit-learn`: normalização, modelos de referência e métricas;
- `tensorflow`: treinamento de MLP com Keras;
- `ipykernel`: kernel Jupyter no VS Code.

## 7. Roteiro de execução do estudo

### Fase A — leitura e auditoria

1. Conferir a presença e os nomes dos três CSVs em `dados/`.
2. Localizar o cabeçalho real de cada arquivo e inspecionar colunas/tipos.
3. Converter do formato mensal para registros diários, com datas válidas.
4. Investigar valores ausentes, duplicidades, status e preferências por dados consistidos.
5. Quantificar a cobertura temporal, lacunas e interseção entre séries.

### Fase B — análise e variáveis

6. Unir séries pela data, verificando horário/convenção das medições.
7. Explorar gráficos e estatísticas de chuva, cotas, vazões e eventos.
8. Criar entradas de dias anteriores sem atravessar períodos com dados faltantes.
9. Definir uma única saída inicial: `cota(t+1)`.

### Fase C — treino e comparação

10. Separar amostras **cronologicamente** em treino, validação e teste.
11. Ajustar o `MinMaxScaler` **somente no treino** e aplicar o mesmo ajuste aos demais conjuntos; o teste pode conter valores fora de `[0, 1]` caso exceda o intervalo observado no treino, o que não deve ser ocultado por novo ajuste.
12. Treinar baseline de persistência e redes MLP com diferentes configurações.
13. Selecionar hiperparâmetros a partir da **validação**.
14. Avaliar **uma vez o modelo escolhido no teste**, reportando MAE, RMSE, R² e análises gráficas. Comparar o erro em períodos comuns e extremos.
15. Exportar resultados em `resultados/` e modelos em `modelos/`.

**Nunca** embaralhar aleatoriamente toda a série para testar previsão futura, ajustar escaladores nos dados completos, usar chuva futura observada como entrada, ou interpolar grandes lacunas sem justificar a metodologia.

## 8. Critérios de avaliação dos resultados

- **MAE:** média do erro absoluto da previsão, na unidade da cota.
- **RMSE:** penaliza mais fortemente erros grandes.
- **R²:** indicador de ajuste relativo, que exige interpretação cautelosa em séries temporais.
- **Gráficos:** curvas de treino/validação, linha da cota real vs. prevista, erros e comparação dos modelos.

As **unidades específicas da cota** (frequentemente centímetros em séries de régua) devem ser **confirmadas nos metadados da estação** antes de rotular gráficos. Valores negativos de cota não devem ser removidos automaticamente, pois podem refletir o referencial da régua.

O resultado será acadêmico mesmo se a rede não superar a persistência: nesse caso, a conclusão precisa explicar o desempenho e as limitações sem inventar eficácia.

## 9. Apresentação de 5 minutos

| Seção solicitada | Conteúdo sugerido |
| --- | --- |
| Capa | Nome do projeto e estudante |
| Introdução | Problema, estações, quantidade **real de amostras finais** e características utilizadas |
| Metodologia | Tratamento, normalização 0–1, separação temporal, teste de arquiteturas |
| Resultados e discussão | Arquitetura escolhida, métricas de teste, comparação com persistência e gráficos |
| Conclusões | Desempenho, dificuldades encontradas, aprendizados e limites |

A apresentação deverá informar **as instâncias e colunas efetivamente usadas no dataset supervisionado**, não apenas as linhas mensais dos CSVs brutos.

## 10. Limitações e escopo futuro

- Não confundir **nível de rio** com **volume de reservatório**.
- Sem cotas oficiais de alerta, valores estatisticamente altos não constituem diagnóstico de enchente.
- Chuvas locais não descrevem necessariamente toda a bacia.
- Vazão e cota podem não ser informações independentes, pois a vazão pode ser estimada a partir da cota.
- Uma avaliação retrospectiva não valida alertas em tempo real.
- O PI5 pode aproveitar futuramente conhecimento de variáveis relevantes, mas **nenhuma integração, treinamento ou implantação de modelo no PI5 é presumida concluída**.

## 11. Próximo passo

Copiar os arquivos CSV originais para `dados/`, abrir o notebook e iniciar pela **inspeção da estrutura de cada arquivo**. As próximas células de tratamento, exploração e redes neurais devem ser acrescentadas gradualmente, explicando as decisões a partir dos dados reais.

---

**Referências:** ANA/HidroWeb (https://www.snirh.gov.br/hidroweb/serieshistoricas); relatório de pesquisa fornecido pelo estudante em 19/10/2026, *Estudo de séries hidrológicas da ANA com redes neurais*.
