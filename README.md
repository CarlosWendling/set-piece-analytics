# Set Piece Analytics

Prova de conceito de análise de dados aplicada ao futebol, utilizando dados reais do StatsBomb Open Data para avaliar a eficiência ofensiva de diferentes tipos de bolas paradas.

## Fonte de Dados

**StatsBomb Open Data** — dados gratuitos disponibilizados publicamente pelo StatsBomb.

> Se você publicar, compartilhar ou distribuir qualquer análise baseada nesses dados, cite a fonte como **StatsBomb** e utilize o logotipo oficial disponível no [Media Pack](https://statsbomb.com/media-pack/).

## Tipos de Bola Parada Analisados

| Tipo           | Origem no dado             |
| -------------- | -------------------------- |
| Escanteio      | `pass_type == "Corner"`    |
| Falta Indireta | `pass_type == "Free Kick"` |
| Lateral        | `pass_type == "Throw-in"`  |
| Falta Direta   | `shot_type == "Free Kick"` |
| Pênalti        | `shot_type == "Penalty"`   |

## KPIs

| KPI                      | Fórmula                                         |
| ------------------------ | ----------------------------------------------- |
| Taxa de Finalização (TF) | Finalizações Geradas / Bolas Paradas Executadas |
| Taxa de Conversão (TC)   | Gols Marcados / Bolas Paradas Executadas        |
| xG Médio                 | Soma dos xG / Bolas Paradas Executadas          |

## Setup

```bash
pip install -r requirements.txt
```

## Como Rodar o ETL

1. Certifique-se de ter conexão com a internet
2. Abra o notebook: `jupyter notebook etl.ipynb`
3. Execute as células em ordem — as células 1–4 são exploratórias, o processamento começa na célula 5
4. Os arquivos serão gerados em `data_processed/` ao final

> Se o processo for interrompido, re-rodar a célula 5 retoma de onde parou — competições já processadas são puladas automaticamente.

## Estrutura de Arquivos

```
set-piece-analytics/
├── data_processed/            # datasets tratados (gerados pelo ETL)
│   ├── la_liga/
│   │   ├── 2017_2018.parquet
│   │   └── 2018_2019.parquet
│   ├── uefa_champions_league/
│   │   └── 2015_2016.parquet
│   ├── ...                    # uma pasta por competição
│   └── set_pieces.parquet     # centralizado com todas as competições
├── etl.ipynb                  # notebook de extração e tratamento
├── requirements.txt
└── README.md
```

### Sobre os arquivos em `data_processed/`

- **`<competicao>/<temporada>.parquet`** — dados de uma competição/temporada específica. Use para carregar um subconjunto sem precisar filtrar o arquivo completo.
- **`set_pieces.parquet`** — concatenação de todos os arquivos individuais. Use como fonte principal no Streamlit.

## Schema do Dataset (`set_pieces.parquet`)

| Coluna             | Tipo  | Descrição                                                       |
| ------------------ | ----- | --------------------------------------------------------------- |
| `match_id`         | int   | ID da partida                                                   |
| `competition_name` | str   | Nome da competição (ex: "La Liga")                              |
| `season_name`      | str   | Temporada (ex: "2017/2018")                                     |
| `team_name`        | str   | Time que executou a bola parada                                 |
| `set_piece_type`   | str   | Tipo da bola parada                                             |
| `period`           | int   | Período do jogo (1 ou 2)                                        |
| `minute`           | int   | Minuto do evento                                                |
| `minute_band`      | str   | Faixa de minuto ("0-15", "16-30", ..., "76-90+")                |
| `shots_generated`  | int   | 1 se a posse gerou pelo menos uma finalização, 0 caso contrário |
| `goals_scored`     | int   | 1 se a posse resultou em gol, 0 caso contrário                  |
| `xg_sum`           | float | Soma do xG de todos os chutes na posse                          |
| `origin_x`         | float | Coordenada X de onde a bola parada foi cobrada                  |
| `origin_y`         | float | Coordenada Y de onde a bola parada foi cobrada                  |
| `shot_x`           | float | Coordenada X do chute (NaN se não houve finalização)            |
| `shot_y`           | float | Coordenada Y do chute (NaN se não houve finalização)            |

O campo de coordenadas segue o padrão StatsBomb: campo de 120x80, origem no canto inferior esquerdo.
