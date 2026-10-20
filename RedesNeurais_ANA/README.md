# Previsão da cota do Rio Taquari com redes neurais

Projeto acadêmico de regressão supervisionada para estudar se cotas recentes e chuva observada ajudam a estimar a cota do Rio Taquari em Encantado (RS) no dia seguinte. Os dados são exportações do HidroWeb/ANA para as estações 86720000 (cota e vazão) e 2951010 (chuva).

O estudo é retrospectivo e didático. Não é um sistema operacional de previsão ou alerta; cota de rio não representa capacidade de reservatório. A integração com o PI5 está fora do escopo desta entrega.

## Estado e resultados da execução atual

Os três CSVs já estão em `dados/`. O notebook lê o cabeçalho ANA após as linhas de metadados, expande colunas mensais para dias, preserva ausências e zeros de chuva observados, e aplica a regra de status documentada no notebook.

Foram geradas 18.252 janelas completas usando sete dias de histórico até `t` para prever `cota(t+1)`. A divisão cronológica reserva **20% finais para teste**; os 80% de desenvolvimento contêm 68% treino efetivo e 12% validação. A rede escolhida na validação foi a MLP 64–32 com cotas e chuva.

| Modelo no teste | MAE | RMSE | R² |
| --- | ---: | ---: | ---: |
| MLP 64–32 — cota + chuva | 32,112 | 71,777 | 0,769 |
| Persistência | 32,842 | 83,625 | 0,686 |
| Ridge — cota + chuva | 31,375 | 74,371 | 0,752 |

A MLP superou persistência nas três métricas deste corte, com ganho modesto em MAE. A Ridge teve menor MAE, mas RMSE maior e R² menor que a MLP. A conclusão e os limites estão detalhados em [resultados/resultados.md](resultados/resultados.md). Esses resultados não validam uso operacional.

## Executar

Requer Python 3.13 (64 bits), VS Code com extensões Python e Jupyter, e os pacotes de `requirements.txt`.

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Abra `notebooks/analise_redes_neurais.ipynb`, selecione o kernel `.venv` e execute as células em ordem. Os resultados derivados são regravados em `resultados/`, sem alterar os CSVs brutos. O notebook roda em Jupyter/VS Code e pode ser aberto no Colab se os dados forem disponibilizados lá; ainda falta confirmar com o professor se Colab é obrigatório.

## Artefatos

- `notebooks/analise_redes_neurais.ipynb`: auditoria, análise, preparação, treino e avaliação.
- `resultados/resultados.md`: interpretação desta execução e itens restantes para fechar a entrega.
- `docs/ROTEIRO_APRESENTACAO.md`: roteiro de cinco minutos; atualize-o se os dados forem reexecutados.
- `resultados/`: séries diárias e supervisionadas, métricas, seleção de modelos, previsões e figuras (incluindo chuva antecedente).

Os dados brutos e os resultados de máquina são ignorados pelo Git. `resultados/resultados.md` é mantido como artefato rastreável; os demais CSVs e figuras podem ser anexados separadamente na entrega.

## Método e limitações

- Entradas: cotas dos sete dias até o fim de `t`; alguns candidatos incluem também chuva nesses mesmos dias. O alvo é a cota em `t+1`.
- Janelas com lacunas são descartadas. Não há interpolação nem embaralhamento aleatório.
- A divisão é 68% treino, 12% validação e 20% teste em ordem temporal; a normalização MinMax é ajustada somente no treino.
- A seleção da MLP usa MAE de validação. O teste compara a MLP escolhida com persistência e Ridge.
- A chuva é de uma estação distinta, não representa toda a bacia e sua disponibilidade no fim do próprio dia precisa ser confirmada.
- A unidade/referência da cota ainda precisa ser confirmada nos metadados da estação. Percentis de nível são estatísticos, não limiares oficiais de cheia ou estiagem.


## Vers?o 2 ? compara??es de janelas

A V1 est? preservada em `notebooks/analise_redes_neurais.ipynb` e no checkpoint `notebooks/analise_redes_neurais_v1.ipynb`. A an?lise com janelas de 7, 15 e 30 dias est? em `notebooks/analise_redes_neurais_v2.ipynb`; ela grava m?tricas, previs?es e figuras em `resultados/v2/` sem sobrescrever os artefatos da V1. A interpreta??o e as limita??es da execu??o V2 est?o na se??o correspondente de `resultados/resultados.md`. Para reproduzir, abra o notebook V2 no VS Code, selecione o kernel `.venv` e execute todas as c?lulas em ordem.

O roteiro de apresenta??o de cinco minutos foi atualizado para a V2 em `docs/ROTEIRO_APRESENTACAO.md`; a vers?o anterior est? preservada em `docs/ROTEIRO_APRESENTACAO_V1.md`.
