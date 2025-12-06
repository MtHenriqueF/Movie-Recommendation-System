# Sistema de Recomendação de Filmes com ALS e PySpark

## Problema e motivação

Sistemas de recomendação estão em praticamente todo lugar: serviços de streaming (Netflix, Prime Video, Paramount+), e-commerce (Amazon), plataformas de música e assim por diante. A ideia central é sempre a mesma: **usar o histórico de interações dos usuários com itens** (filmes, produtos, músicas) para sugerir automaticamente novos conteúdos relevantes.

Esses sistemas lidam com grandes volumes de dados: milhões de usuários, milhões de itens e bilhões de interações. Isso exige:

- um bom **pré-processamento de dados**;
- algoritmos de **machine learning escaláveis**;
- ferramentas que suportem **Big Data**.

Neste projeto, exploramos esse cenário construindo um sistema de recomendação de filmes usando o algoritmo **ALS (Alternating Least Squares)** implementado no **MLlib do PySpark**, a partir do dataset MovieLens.

---

## O que é o ALS (visão geral)

O ALS é um algoritmo de **fatoração de matriz** usado em sistemas de recomendação colaborativos. De forma bem resumida:

- Começamos com uma matriz de ratings `R`, onde cada linha é um usuário, cada coluna é um filme, e cada célula (quando preenchida) é a nota que o usuário deu para aquele filme.
- Essa matriz é **enorme e esparsa** (cheia de buracos, pois a maioria dos usuários avalia poucos filmes).
- O ALS tenta aproximar `R` como o produto de duas matrizes menores:
  - uma matriz de **fatores de usuário** (vetores que representam o “gosto” de cada usuário);
  - uma matriz de **fatores de item** (vetores que representam as “características latentes” dos filmes).
- Ao multiplicar esses vetores, obtemos uma **nota prevista** para qualquer par usuário–filme, incluso onde não existia rating.

O nome *Alternating Least Squares* vem do processo de treino:

- fixa os vetores dos filmes e calcula os vetores dos usuários por mínimos quadrados;
- depois fixa os vetores dos usuários e recalcula os vetores dos filmes;
- alterna esses passos até a convergência.

No fim, conseguimos:

- prever a nota que um usuário provavelmente daria para filmes não avaliados;
- gerar **listas de recomendação personalizadas** (Top-N filmes por usuário).

---

## Descrição do conjunto de dados (MovieLens ml-latest)

Para treinar e avaliar o modelo, utilizamos o dataset **MovieLens ml-latest**, disponibilizado pelo GroupLens (Universidade de Minnesota).

Algumas características importantes:

- **33.832.162 ratings** (avaliações de 0.5 a 5.0 estrelas)
- **2.328.315 tags aplicadas**
- Aproximadamente **86.537 filmes**
- **330.975 usuários** distintos

No ambiente do projeto:

- A pasta completa do MovieLens descompactada ocupa cerca de **1,5 GB**.
- Neste trabalho, utilizamos apenas os arquivos:
  - `ratings.csv` (todas as avaliações usuário–filme)
  - `movies.csv` (metadados dos filmes: título e gêneros)
- Esses dois arquivos juntos somam aproximadamente **894,6 MB**, o que já é suficiente para:
  - perceber limitações de desempenho (tempo de execução, uso de memória),
  - justificar o uso de ferramentas de processamento em larga escala, como **PySpark**.

Estrutura dos arquivos principais utilizados:

- **`ratings.csv`**
  - `userId`: identificador anônimo do usuário  
  - `movieId`: identificador do filme  
  - `rating`: nota de 0.5 a 5.0 (passo de 0.5)  
  - `timestamp`: instante da avaliação em segundos desde 01/01/1970 (epoch)

- **`movies.csv`**
  - `movieId`: identificador do filme (chave que liga com `ratings.csv`)  
  - `title`: título do filme, geralmente com o ano entre parênteses (ex.: `Toy Story (1995)`)  
  - `genres`: lista de gêneros separados por `|` (ex.: `Adventure|Animation|Children|Comedy|Fantasy`)

Esses dois arquivos formam a base do nosso sistema de recomendação:  
`ratings.csv` fornece as interações usuário–filme para treinar o ALS, e `movies.csv` enriquece as recomendações com os títulos e gêneros para exibição no dashboard.

---

## Ferramentas utilizadas e por quê

- **PySpark + Spark MLlib**  
  - Utilizado para o pré-processamento em larga escala e para treinar o modelo **ALS**.  
  - O ALS do MLlib é otimizado para grandes volumes de dados e possui integração nativa com estruturas distribuídas (DataFrames/RDDs), o que é adequado ao tamanho do MovieLens.

- **Matplotlib e Seaborn**  
  - Utilizados para criar gráficos e visualizações descritivas, como:
    - distribuição de notas (ratings);
    - filmes mais bem avaliados;
    - gêneros mais frequentes / melhor avaliados.
  - O Seaborn foi usado para deixar o visual mais amigável, construído em cima do Matplotlib.

- **Streamlit**  
  - Utilizado para criar um **dashboard interativo**, permitindo:
    - escolher um usuário e visualizar seus filmes recomendados;
    - explorar filmes com melhor recomendação global;
    - visualizar estatísticas da base (gêneros, distribuições, etc.)
  - Facilita a apresentação dos resultados de forma visual e navegável, sem precisar de uma aplicação web complexa.

---

## Metodologia (resumo do pipeline)

1. **Ingestão de dados (Spark)**
   - Leitura de `ratings.csv` e `movies.csv` usando PySpark.
   - Conversão de tipos (`userId`, `movieId`, `rating`, `timestamp` etc.).

2. **Pré-processamento**
   - Remoção de linhas com valores nulos em campos essenciais.
   - Filtragem de notas fora do intervalo [0.5, 5.0].
   - Tratamento de duplicatas:
     - se um mesmo usuário avaliou o mesmo filme mais de uma vez, é feita a **média das notas**.
   - Escrita dos dados processados em formato **Parquet** para acelerar leituras futuras.

3. **Treino do modelo ALS (MLlib)**
   - Separação em **treino** e **teste**.
   - Ajuste de alguns hiperparâmetros (por exemplo: `rank`, `regParam`, `maxIter`) utilizando validação CrossValidator.
   - Avaliação do modelo com métricas como:
     - **RMSE** (Root Mean Squared Error);
     - **MAPE** (Mean Absolute Percentage Error).
   - Uso do melhor modelo treinado para gerar predições e Top-N recomendações.

4. **Geração de recomendações**
   - Para um usuário específico, o modelo ALS é usado para prever ratings e listar os Top-N filmes mais recomendados.
   - As recomendações são combinadas com `movies.csv` para mostrar:
     - título do filme;
     - gêneros correspondentes.

5. **Visualização e dashboard (Streamlit)**
   - Criação de um app Streamlit com:
     - gráficos de análise exploratória;
     - interface para selecionar um usuário e ver as recomendações;
     - listagens de filmes e gêneros mais bem avaliados.

---

## Principais resultados e visualizações

No dashboard interativo (Streamlit), é possível observar, por exemplo:

- **Filmes com melhores recomendações globais**  
  A partir das predições do ALS, mostramos os filmes com maiores ratings previstos em geral.

- **Top 20 filmes recomendados para um usuário específico**  
  Dado um `userId`, o sistema lista os filmes que esse usuário ainda não avaliou, ordenados pela nota prevista pelo modelo.

- **Análise de gêneros mais bem avaliados**  
  Cruzando as recomendações/predições com os gêneros dos filmes, é possível ver quais gêneros tendem a ter maiores médias de rating.

Por questões de tempo de execução em sala de aula, gravamos vídeos de demonstração, disponíveis na pasta `videos`, que serão utilizados na apresentação. Rodar o pipeline completo (especialmente o treino do ALS com validação) pode levar alguns minutos em ambiente de notebook.

Esses resultados fornecem **insights práticos** sobre:

- que filmes podem ser destacados para cada usuário;
- quais gêneros parecem mais promissores para campanhas ou destaques na plataforma.

---

## Considerações sobre desempenho e Big Data

Embora o dataset tenha “apenas” um pouco mais de 1 GB em disco, ele já é suficiente para:

- perceber o custo de:
  - ler dezenas de milhões de linhas;
  - processar e agregar grandes tabelas (ex.: `groupBy` em milhões de ratings);
  - treinar modelos de fatoração de matriz como o ALS;
- motivar o uso de ferramentas como **PySpark/MLlib** em vez de processar tudo com bibliotecas de análise puramente locais.

Ao longo do projeto, foi possível observar:

- tempos de treino significativos para o ALS, especialmente quando usamos validação cruzada e grids de hiperparâmetros maiores;
- a importância de formatos de armazenamento mais eficientes (como **Parquet**) para acelerar leituras sucessivas;
- aspectos clássicos de Big Data, como:
  - **Volume**: milhões de linhas de ratings;
  - **Variedade**: ratings, metadados de filmes, tags, genome;
  - **Velocidade** (no sentido de custo de processamento e tempo de resposta desejado);
  - **Veracidade**: dados reais, com ruídos e sparsidade;
  - **Valor**: geração de recomendações úteis a partir desses dados.

---

## Conclusão

O projeto demonstrou, em escala reduzida, como funciona um sistema de recomendação baseado em **filtro colaborativo com ALS**, utilizando o ecossistema **Spark + MLlib**.  

Mesmo em ambiente de notebook, foi possível perceber:

- desafios de desempenho ao lidar com dezenas de milhões de registros;
- a importância de pré-processamento adequado e de formatos otimizados (Parquet);
- como modelos relativamente simples, como o ALS, já conseguem gerar **recomendações úteis e interpretáveis**.

Ferramentas como PySpark, Matplotlib/Seaborn e Streamlit se complementaram bem para cobrir os principais aspectos da disciplina: **processamento em larga escala**, **aprendizado de máquina** e **visualização/interação com dados**.
