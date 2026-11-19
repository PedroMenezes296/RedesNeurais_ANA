# Plano de implementação — roteiro para Codex e estudante

**Objetivo final:** entregar notebook Jupyter didático e reproduzível com limpeza dos dados ANA, treino de regressão neural para `cota(t+1)`, seleção da arquitetura, avaliação honesta em teste e resultados para apresentação de 5 minutos.

**Regra de execução:** avançar **uma fase de cada vez**, registrar descobertas, validar saídas e não pressupor que etapas futuras já estejam concluídas. Pastas de dados começam vazias.

## Fase 0 — Ambiente e repositório
**Implementar:** verificar estrutura atual, versão de Python, instalação de extensões Python/Jupyter, `.venv` e dependências de `requirements.txt`; garantir que notebook rode no VS Code. Não mudar arquivos de dados.

**Aceite:** kernel selecionável, importações básicas executam sem erro; se TensorFlow for incompatível com Python instalado, relatar versões e propor ajuste.

## Fase 1 — Leitura e inspeção (primeira implementação efetiva)
**Implementar:** localizar os três arquivos em `dados/` sem presumir que existem. Se presentes, identificar dinamicamente cabeçalho, inspecionar primeiras linhas, separador, codificação, colunas, tipos, registros mensais, datas e status. Documentar diferenças entre os arquivos e contagens brutas.

**Aceite:** tabelas/saídas verificáveis e explicação legível; se faltar CSV, instrução clara de onde colocá-lo, sem dados fictícios.

## Fase 2 — Expansão diária e qualidade
**Implementar:** converter as colunas de dias (`Cota01` etc.) para registros diários, validar datas existentes, formatos numéricos, registros brutos/consistidos, status e duplicatas. Definir regra de escolha e auditar exclusões. Preservar zeros de chuva e dados faltantes.

**Aceite:** cada série diária tem data única por variável/estação; relatório de duplicação, qualidade, datas extremas, cobertura e lacunas.

## Fase 3 — Integração e exploração
**Implementar:** juntar cota e chuva por datas compatíveis; vazão inicialmente opcional. Inspecionar horário de observação e disponibilidade de dados. Visualizar séries, lacunas, distribuições e chuva antecedente versus variações de cota, sem afirmar causalidade.

**Aceite:** base integrada com proveniência, cobertura descrita, gráficos legíveis, unidades verificadas/indicadas como pendentes, nenhuma interpolação silenciosa.

## Fase 4 — Dataset supervisionado
**Implementar:** fixar o instante de previsão e janela de histórico (proposta: sete dias); construir entradas com níveis e chuvas anteriores e saída `cota(t+1)`. Exigir datas realmente consecutivas e completas. Separar treino/validação/teste em ordem cronológica (divisão 70/15/15 é apenas ponto inicial, a confirmar pela cobertura).

**Aceite:** datas e quantidades em cada conjunto, número de características, checagem automatizada contra uso de informação futura e contra janelas com lacunas. Alvo t+1 corretamente alinhado.

## Fase 5 — Normalização e referências simples
**Implementar:** `MinMaxScaler` nas entradas (0–1 com parâmetros ajustados somente no treino), mesma transformação em validação/teste; escala separada para alvo apenas se necessária e inversão antes das métricas. Criar baseline de persistência `cota(t+1)=cota(t)`; opcionalmente regressão linear.

**Aceite:** transformações não usam validação/teste no ajuste; previsão de persistência computada no mesmo recorte de avaliação e unidade da cota.

## Fase 6 — Redes neurais e escolha da arquitetura
**Implementar:** primeiro modelo de um único neurônio no estilo do notebook de Sandro (Keras `Sequential` + `Dense(1)`); depois testar MLP pequena de uma camada oculta e, se justificável, duas. Documentar entrada, camadas, neurônios, ativação, perda, otimizador, epochs, batch size e semente. Usar curva de perda/validação e early stopping quando apropriado.

**Aceite:** comparar métricas de **validação** das configurações sob os mesmos dados e explicar por que uma arquitetura foi selecionada; não escolher arquitetura pelo teste.

## Fase 7 — Teste final e discussão
**Implementar:** congelar modelo escolhido, calcular previsões no teste cronologicamente posterior e reportar MAE, RMSE, R²; opcional correlação. Criar gráficos previsto × observado, série temporal de trecho do teste e erros em cotas altas/baixas (limites derivados do treino, com cautela).

**Aceite:** resultados reproduzíveis, comparação com persistência, interpretação de falhas e declaração de limitações; nunca fabricar métricas.

## Fase 8 — Documentação e apresentação
**Implementar:** registrar comandos de execução e bibliotecas no README, entradas usadas, amostras válidas, arquitetura selecionada e métricas reais. Preparar 5 minutos conforme `docs/REQUISITOS_PROFESSOR.md`.

**Aceite:** a apresentação contém capa, descrição de base e contagens, método de seleção da arquitetura, resultados no teste e conclusões (métricas, dificuldades e aprendizados).

## Entregas e diretórios
- `notebooks/analise_redes_neurais.ipynb`: principal artefato da disciplina.
- `dados/`: CSVs brutos adicionados pelo estudante; **não modificar**.
- `dados/processados/`: somente dados derivados, se necessário.
- `resultados/`: gráficos e métricas gerados após execução.
- `modelos/`: modelos treinados (se solicitado).
- `referencias/`: notebook original do professor, para consulta, sem alterações.
- `docs/`: justificativas, plano, requisitos e relatório original.

## Solicitação inicial sugerida ao Codex
> Leia AGENTS.md, README.md e os arquivos de docs/, examine o notebook de referência do professor Sandro, e faça um diagnóstico do estado do repositório. Proponha a primeira implementação **somente da Fase 1** (leitura e inspeção), sem treinar modelos nem gerar resultados fictícios. Após apresentar o plano, implemente apenas quando eu solicitar.

## Definições ainda abertas
- Qual o período contínuo mais confiável das séries para modelagem?
- A chuva de `t` está efetivamente disponível no instante da predição? Se não, como definir as defasagens?
- Unidades/referência da cota e qualidade das observações por status.
- A vazão agrega informação, considerando que pode ser derivada da própria cota?
- Quais arquiteturas efetivamente serão comparadas, segundo resultados de validação e tempo da disciplina?
