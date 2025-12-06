# Descrição sobre o código
Caso queira as informações sobre a entrega técniga, veja o arquivo `README.md`, 
Caso queira ver o código funcionando, temos 2 videos na pasta videos

- Pasta`checkpoint_spark` é uma pasta gerada por uma feature do PySpark para garantir a tolerância a falhas e otimizar o desempenho de aplicações de processamento de dados em larga escala.
  
- Pasta `dados_brutos` é uma pasta onde guardamos os datasets que foram pegos do site: [Data set](https://grouplens.org/datasets/movielens/)
  
- Pasta `dados_processados` são os dados resultantes depois de etapas de pré processamento, como eliminação de colunas, avaliação de nulls, etc.

- Pasta `metricas` salvamos as métricas que tivemos do modelo como RMSE, MAPE para verificar se o modelo está funcionando adequadamente, além de algumas informações sobre o dataset, por exemplo nota média por gênero, filmes por gênero, se há alguma duplicata, contagem de nulls, etc.
  
- Pasta `modelos` é onde foram salvados os modelos treinados, foi uma idéia que tivemos para poder utilizar o modelo em tempo de execução no streamlit, fazendo regressões just-in-time, ou seja não regredimos tudo, somente quando pedido e necessário.
  
- `Aplicação Streamlit + Ngrok (front-end).ipynb`é basicamente nosso front-end, como o nome diz é onde o app do streamlit está, os dashboards são montados em tempo de execução, carregando o modelo, e métricas para utilizar seaborn e matplotlib dentro dele, diminuindo a necessidade de retreinar, ou recarregar o dataset
  
- `Engenharia de Dados + Treino ALS (back-end).ipynb` é o back-end do nosso sistema, é onde é feito os “groupby’s” para insights, o pré processamento e o salvamento do modelo
- `README.md` é o arquivo contendo as informações sobre a entrega técnica:
  * O problema abordado e a motivação
  * As ferramentas utilizadas e por quê
  * Principais resultados obtidos ou visualizações geradas



