# MVP - Engenharia de Dados: análise de séries de TV

**Autor:** [RicCesar](https://github.com/RicCesar) 
**Aluno:** Ricardo César Santos Mendes Costa
**Curso:** Pós-graduação em Data Science & Analytics - PUC-Rio  
**Plataforma:** Databricks, Unity Catalog, PySpark e Delta Lake

## Sobre o projeto

Gosto de séries de TV e, neste MVP da Sprint de Engenharia de Dados, encontrei uma ótima oportunidade de transformar esse interesse em uma análise de dados. O projeto compara nove séries a partir de informações sobre episódios, datas de exibição, audiência nos Estados Unidos e avaliações do IMDb.

O objetivo foi construir um pipeline de dados no Databricks seguindo a arquitetura medalhão, da ingestão dos arquivos até tabelas analíticas e respostas a perguntas de negócio. Além de observar diferenças entre as séries, o trabalho explora a qualidade e as limitações das fontes.

## Perguntas de negócio

1. Quais séries tiveram o maior intervalo entre a primeira e a última exibição registrada?
2. Quais séries possuem mais episódios?
3. Quais séries têm as maiores notas médias no IMDb?
4. Como a nota média IMDb varia entre as temporadas?
5. Como a audiência média nos Estados Unidos varia entre as temporadas?
6. Existe associação entre a audiência nos Estados Unidos e a nota IMDb dos episódios?

Como análises complementares, também foram avaliadas a variação percentual de audiência entre temporadas consecutivas e a dispersão das notas IMDb em cada série.

## Fontes de dados

Os datasets foram obtidos no Kaggle, na página do usuário [bcruise](https://www.kaggle.com/bcruise/datasets). Cada conjunto disponibiliza informações de episódios e/ou avaliações IMDb para uma das séries:

| Série | Dataset no Kaggle |
|---|---|
| The Big Bang Theory | [Big Bang Theory Episodes](https://www.kaggle.com/datasets/bcruise/big-bang-theory-episodes) |
| Brooklyn Nine-Nine | [Brooklyn 99 Episode Data](https://www.kaggle.com/datasets/bcruise/brooklyn-99-episode-data) |
| Lost | [Lost Episodes](https://www.kaggle.com/datasets/bcruise/lost-episodes) |
| The Office | [The Office Episodes Data](https://www.kaggle.com/datasets/bcruise/the-office-episodes-data) |
| Friends | [Friends Episode Data](https://www.kaggle.com/datasets/bcruise/friends-episode-data) |
| How I Met Your Mother | [How I Met Your Mother Episodes Data](https://www.kaggle.com/datasets/bcruise/how-i-met-your-mother-episodes-data) |
| Game of Thrones | [Game of Thrones Episodes](https://www.kaggle.com/datasets/bcruise/game-of-thrones-episodes) |
| Breaking Bad | [Breaking Bad Episode Data](https://www.kaggle.com/datasets/bcruise/breaking-bad-episode-data) |
| The Walking Dead | [The Walking Dead Episodes](https://www.kaggle.com/datasets/bcruise/the-walking-dead-episodes) |

Os arquivos foram baixados e carregados no volume do Databricks. Os dados brutos não estão incluídos neste repositório. Consulte cada página do Kaggle para verificar os termos e a licença aplicáveis ao respectivo dataset.

## Pipeline e modelagem

O pipeline está documentado no notebook [`NB_Final_Analise_Series.ipynb`](./NB_Final_Analise_Series.ipynb) e foi organizado em três camadas:

- **Bronze:** leitura dos CSVs como texto, com identificação da série, caminho do arquivo de origem e horário de ingestão.
- **Silver:** tipagem dos campos, padronização de datas, preservação dos valores originais relevantes e validações de qualidade e relacionamento.
- **Gold:** tabelas agregadas para análise por série e por temporada, além de métricas de correlação, variação de audiência e dispersão de notas.

O grão das principais tabelas Gold é:

| Tabela | Grão |
|---|---|
| `gold.series_metrics` | Uma linha por série |
| `gold.season_metrics` | Uma linha por série e temporada |
| `gold.audience_rating_correlation` | Uma linha por série, usando episódios pareados válidos |
| `gold.audience_season_change` | Uma linha por série e temporada |
| `gold.rating_consistency` | Uma linha por série |

As tabelas de episódios e IMDb foram agregadas separadamente antes de serem combinadas por série ou temporada. Para a correlação, o pareamento entre as fontes usa série, título normalizado e data de exibição, exigindo que essa combinação seja única em cada fonte. Isso reduz o risco de associar episódios diferentes apenas porque suas numerações coincidem.

## Qualidade dos dados e decisões de tratamento

As verificações de qualidade orientaram ajustes necessários para produzir análises confiáveis:

- **Identificação das séries:** durante a ingestão, os registros chegaram inicialmente agrupados sob o identificador `datfiles`. A extração do identificador a partir do nome dos arquivos foi corrigida para atribuir a cada registro a série correspondente.
- **Campos ausentes:** foram encontrados valores vazios, incluindo 433 valores em `prod_code` e 5 em audiência. Como não havia uma fonte que justificasse preenchê-los, foram mantidos como nulos.
- **Referências em números de episódios:** valores como `6[98]` continham uma referência entre colchetes junto ao número. A referência foi removida antes da conversão numérica; valores ainda não conversíveis são identificados com conversão tolerante e podem ser investigados.
- **Numeração de *Lost*:** `LA X - Part 2`, da temporada 6, estava registrado como episódio 12, embora a sequência e a comparação com IMDb indicassem que era o episódio 2. A correção foi aplicada na Silver, preservando a Bronze como registro original e marcando a correção.
- **Formatos de data:** as fontes apresentavam datas ISO, como `2008-01-20`, e datas textuais, como `13 Apr. 2010`. Os dois padrões foram tratados para possibilitar a conversão e o pareamento.
- **Diferenças entre fontes:** as numerações e as quantidades de episódios não coincidem em todas as séries e temporadas. Os registros sem pareamento seguro continuam nas tabelas de origem, mas não entram na correlação audiência-nota.
- **Piloto não exibido:** o episódio “Unaired Pilot” de *The Big Bang Theory* foi excluído do cálculo da média IMDb por não ser um episódio exibido.

## Resultados principais

Os valores abaixo resumem as métricas calculadas pelas regras deste projeto. As contagens representam os registros e identificadores presentes nos datasets utilizados; diferenças de granularidade, episódios divididos em partes e divergências entre fontes devem ser consideradas na interpretação.

| Série | Episódios | Intervalo de exibição (anos) | Nota média IMDb |
|---|---:|---:|---:|
| The Big Bang Theory | 279 | 11,64 | 7,82 |
| Breaking Bad | 62 | 5,69 | 9,03 |
| Brooklyn Nine-Nine | 153 | 8,00 | 8,13 |
| Friends | 236 | 9,62 | 8,42 |
| Game of Thrones | 73 | 8,09 | 8,75 |
| How I Met Your Mother | 208 | 8,53 | 8,14 |
| Lost | 121 | 5,66 | 8,55 |
| The Office | 201 | 8,15 | 8,22 |
| The Walking Dead | 177 | 12,05 | 7,92 |

Entre as observações por temporada, *Game of Thrones* teve nota média IMDb de 6,43 na oitava temporada, ante 9,03 na sétima. *Breaking Bad* alcançou média 9,43 na quinta temporada. *Friends* teve sua maior audiência média na segunda temporada, cerca de 31,72 milhões de espectadores nos Estados Unidos.

### Associação entre audiência e nota

A tabela apresenta a correlação de Pearson calculada nos episódios com audiência e nota disponíveis e pareados com segurança. A correlação descreve associação linear; não demonstra que uma variável cause a outra.

| Série | Episódios pareados válidos | Correlação de Pearson |
|---|---:|---:|
| Breaking Bad | 57 | 0,540 |
| The Office | 175 | 0,511 |
| The Walking Dead | 177 | 0,280 |
| Friends | 234 | 0,276 |
| How I Met Your Mother | 208 | 0,110 |
| Brooklyn Nine-Nine | 153 | 0,007 |
| Lost | 117 | -0,002 |
| The Big Bang Theory | 279 | -0,193 |
| Game of Thrones | 73 | -0,431 |

Os resultados variam entre séries. A análise é descritiva e não controla fatores como época de exibição, mudanças de audiência ao longo dos anos ou diferenças de cobertura entre as fontes.

## Gráficos

Os gráficos exportados do Databricks estão disponíveis abaixo:

1. [Intervalo entre primeira e última exibição](./01_series_by_duration.png)
2. [Quantidade de episódios por série](./02_series_by_episode_count.png)
3. [Nota média IMDb por série](./03_series_by_avg_imdb_rating.png)
4. [Nota média IMDb por temporada](./04_season_by_avg_imdb_rating.png)
5. [Audiência média nos Estados Unidos por temporada](./05_season_by_avg_viewers.png)
6. [Correlação entre audiência e nota IMDb](./06_serie_by_pearson_correlation.png)
7. [Variação percentual de audiência entre temporadas](./07_season_by_audience_change.png)
8. [Dispersão das notas IMDb por série](./08_serie_by_rating.png)

## Aprendizados

Este trabalho foi uma ótima oportunidade de unir meu interesse por séries à prática de Engenharia de Dados. Aprendi a organizar um fluxo completo no Databricks, desde a ingestão de arquivos até a criação de tabelas analíticas nas camadas Bronze, Silver e Gold.

Também aprendi que a qualidade dos dados precisa ser entendida antes de confiar nos resultados: formatos diferentes de data, campos ausentes, números malformados e divergências entre fontes podem afetar diretamente uma análise. Investigar cada ocorrência, justificar a regra de tratamento, preservar a origem e validar o resultado depois da transformação foram partes centrais do projeto.

Outro aprendizado foi que relacionar tabelas exige mais do que escolher uma coluna em comum. A reconciliação por título e data, junto à análise das diferenças de numeração, ajudou a evitar associações indevidas. Por fim, a correlação de Pearson reforçou a importância de comunicar amostra, limitações e a diferença entre associação e causalidade.

Por fim, utilizei o CODEX do ChatGPT para poder desenvolver e refinar os códigos, porém sempre revisando os resultados e buscando sempre otimizações, foi uma oportunidade de poder me desenvolver mais com esta ferramenta.

## Como reproduzir

O notebook foi desenvolvido para execução no Databricks com acesso ao Unity Catalog. Para reproduzir o pipeline:

1. Obtenha os datasets nas páginas do Kaggle listadas acima e confira seus termos de uso.
2. Crie o catálogo `mvp_engdados`, os schemas `bronze`, `silver` e `gold`, e o volume `bronze.datafiles`.
3. Carregue os CSVs das séries no volume, preservando os nomes de arquivo esperados pelo notebook.
4. Abra `NB_Final_Analise_Series.ipynb` no Databricks e execute as células em ordem.
5. Consulte as tabelas Delta e os resultados nos schemas correspondentes.

O caminho de ingestão utilizado no notebook é `/Volumes/mvp_engdados/bronze/datafiles/`. A execução também pressupõe permissões para ler o volume e criar ou substituir tabelas nos schemas.
