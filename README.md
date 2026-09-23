# Análise de Desempenho e Predição de Falhas em Baterias

Projeto de análise exploratória e modelagem preditiva sobre dados de ciclagem de baterias de íon-lítio. A ideia é observar como temperatura, tensão e corrente se comportam ao longo da vida útil de cada bateria e treinar uma rede neural (MLP) capaz de estimar quanto tempo, ou quantos ciclos, uma bateria dura até falhar.

## Dataset

Os notebooks usam o **Accelerated Life Testing Dataset for Lithium-Ion Batteries with Constant and Variable Loading Conditions** (Fricke, Nascimento e Viana, 2023), disponibilizado pela NASA. Ele traz um CSV por pack de bateria, separado em três grupos:

| Pasta | Conteúdo |
|---|---|
| `regular_alt_batteries` | baterias cicladas com o mesmo nível (ou faixa) de carga durante toda a vida |
| `recommissioned_batteries` | baterias cicladas com níveis de carga diferentes em cada fase da vida |
| `second_life_batteries` | baterias de segunda vida, cicladas com corrente constante |

Principais colunas usadas: `time` (tempo relativo em segundos), `mode` (-1 descarga, 0 repouso, 1 carga), `temperature_battery`, `voltage_charger`, `voltage_load`, `current_load`, `temperature_mosfet` e `temperature_resistor`.

> Os dados **não estão versionados** (a pasta `Predicão_falhas/Data` está no `.gitignore`). Para rodar os notebooks, baixe o dataset e coloque as três pastas dentro de `Predicão_falhas/Data/`.

## Estrutura

```
Analise_Desempenho_Predicao_Falhas/
└── Predicão_falhas/
    ├── data_analises.ipynb   # análise exploratória e gráficos
    ├── data_trainer.ipynb    # engenharia de atributos e treino da MLP
    └── Data/                 # dataset (não versionado)
```

## Notebooks

### `data_analises.ipynb`: análise exploratória

- Gráficos da variação, ao longo do tempo, da temperatura da bateria, da tensão (`voltage_charger` e `voltage_load`), da corrente, das temperaturas do resistor e dos MOSFETs e do modo de operação.
- Cálculo do tempo gasto em cada transição de modo (descarga → outro modo) e da corrente média em cada intervalo.

### `data_trainer.ipynb`: predição de vida útil

1. **Segmentação em ciclos:** cada CSV é dividido em blocos consecutivos de `mode`, que são agrupados de 4 em 4, formando os "quadrantes" que representam um ciclo.
2. **Vetor de atributos por ciclo:** tempo em repouso, em carga e em descarga, além de máximo, mínimo, média e desvio padrão de temperatura e tensão, entre outras medidas.
3. **Alvo (y)**, testado de duas formas:
   - tempo total de vida da bateria (último valor de `time`, normalizado);
   - número total de ciclos da bateria.
4. **Modelo:** MLP em Keras (camadas densas `[64, 32, 64, 32, 64]`, ReLU, saída linear, otimizador Adam, perda MSE e métrica MAE), com divisão de 80/20 entre treino e teste. O treino usa 15 épocas para o alvo de tempo de vida e 200 épocas para o alvo de número de ciclos.
5. Gráficos das curvas de perda e comparação entre valor previsto e valor real em amostras aleatórias.

## Como executar

```bash
pip install pandas numpy matplotlib scikit-learn tensorflow jupyter
cd Predicão_falhas
jupyter notebook
```

Os caminhos dos CSVs nos notebooks usam o formato do Windows (`Data\recommissioned_batteries\...`). Execute a partir da pasta `Predicão_falhas`.

## Tecnologias

Python · Pandas · NumPy · Matplotlib · scikit-learn · TensorFlow/Keras · Jupyter
