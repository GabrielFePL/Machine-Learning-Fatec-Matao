# Relatório de Conclusão – Avaliação Prática  
## Análise Econômica e Criptofinanceira com Alpha Vantage (Câmbio e Bitcoin)

Este relatório apresenta a conclusão da avaliação prática da disciplina, que consistiu na análise econômica e criptofinanceira utilizando a API Alpha Vantage. O foco foi a coleta, tratamento e análise de dados relacionados ao câmbio e ao Bitcoin, com a implementação de scripts em Python.

Ao longo do documento, são detaladas as justificativas para as escolhas técnicas realizadas, como bibliotecas utilizadas, estratégias de manipulação de dados e formas de visualização dos resultados. Também são discutidos os critérios adotados para garantir a precisão, reprodutibilidade e relevância das análises desenvolvidas.

O objetivo principal é demonstrar a aplicação prática dos conhecimentos adquiridos, integrando fundamentos de economia, criptoativos e programação para extrair insights relevantes a partir de dados financeiros.

## Introdução Geral das Escolhas e Implementações

A abordagem adotada na avaliação foi dividida em duas grandes etapas principais: **pré-processamento de dados** e **ingestão bayesiana de dados**. Essa separação teve como objetivo estruturar melhor o fluxo de trabalho e garantir uma análise robusta e confiável dos dados obtidos pela API Alpha Vantage.

### 1. Pré-processamento de Dados

O pré-processamento foi uma etapa fundamental para garantir a qualidade dos dados utilizados nas análises. Essa fase incluiu:

- **Coleta automatizada de dados** de câmbio e Bitcoin por meio da API Alpha Vantage.
- **Tratamento de dados ausentes e inconsistentes**, como valores nulos justificadas por eventos de mercado ou análises personalizadas inseridas.
- **Conversão de formatos de data e hora**, padronizando a granularidade temporal para facilitar análises agregadas.
- **Normalização de escalas** para comparação entre séries temporais heterogêneas (por exemplo, câmbio USD/BRL e preços de BTC/USD).
- **Criação de variáveis auxiliares**, como médias móveis, volatilidade e retornos logarítmicos, com o intuito de enriquecer a base analítica.

Essa etapa permitiu transformar os dados brutos em uma estrutura coerente e adequada para modelagens posteriores.

### 2. Ingestão Bayesiana de Dados

A segunda etapa consistiu na aplicação de uma abordagem **classificatória baseada no Teorema de Bayes**, utilizando o modelo **Naive Bayes Gaussiano (GaussianNB)** da biblioteca `scikit-learn`. Apesar do nome "ingestão neural" ter sido inicialmente mencionado, a implementação não se baseia em redes neurais artificiais, mas sim em um classificador estatístico probabilístico.

As justificativas para o uso dessa abordagem são:

- O **Naive Bayes** é eficiente para conjuntos de dados com múltiplas classes e apresenta ótimo desempenho em classificações simples e rápidas.
- A **versão Gaussiana** do algoritmo é adequada quando os atributos contínuos do conjunto de dados seguem uma distribuição normal — o que se aplica, em parte, às variáveis econômicas utilizadas.
- O modelo oferece **alta interpretabilidade**, permitindo análises diretas das probabilidades condicionais e da aplicação do Teorema de Bayes na prática.
- Ferramentas adicionais como `ConfusionMatrix` da biblioteca `yellowbrick` e `classification_report` do `sklearn.metrics` foram utilizadas para **avaliar a performance do classificador**, permitindo visualizar a acurácia, precisão, recall e F1-score do modelo.
- Também foi realizado um **cálculo detalhado das probabilidades condicionais**, com base no Teorema de Bayes, reforçando a fundamentação matemática da análise.

Essa abordagem demonstrou ser eficaz na identificação de padrões nos dados de câmbio e Bitcoin, com acurácia superior a 94%, o que reforça sua adequação para problemas de classificação financeira com múltiplas categorias como *Conservative*, *Crypto-Friendly* e *Neutral*.

## Justificativas das Etapas da Pipeline

A seguir, detalharemos individualmente cada uma das etapas (pipes) que compõem a pipeline completa desenvolvida neste trabalho. Cada pipe representa uma transformação ou processo específico aplicado aos dados, sendo essencial para garantir a robustez e a coerência dos resultados finais. As justificativas apresentadas explicam as escolhas metodológicas, técnicas utilizadas e o papel de cada componente na estrutura geral da solução.

### Pipe 1 – Importação de Bibliotecas e Montagem de Ambiente

A primeira etapa da pipeline consiste na **importação das bibliotecas** necessárias e na **configuração do ambiente de trabalho**, garantindo que todos os recursos estejam disponíveis para o restante do processamento e análise dos dados.

#### Instalação e Atualização de Bibliotecas

- O comando `!pip install plotly --upgrade` foi utilizado para garantir que a biblioteca `plotly`, responsável por visualizações interativas, estivesse na versão mais recente, prevenindo incompatibilidades com recursos gráficos utilizados posteriormente.

#### Bibliotecas Importadas

- `requests`: permite o consumo de APIs REST, utilizada para acessar dados da Alpha Vantage.
- `pandas` e `numpy`: essenciais para manipulação e análise de dados estruturados.
- `seaborn` e `matplotlib.pyplot`: utilizadas para a criação de gráficos estáticos e análise exploratória dos dados.
- `plotly.express`: biblioteca complementar para criação de gráficos interativos.
- `sklearn.preprocessing`: classes como `StandardScaler`, `LabelEncoder` e `OneHotEncoder` são fundamentais para padronização e transformação dos dados.
- `ColumnTransformer`: utilizada para aplicar diferentes transformações a colunas distintas em pipelines complexas.
- `train_test_split`: para divisão dos dados em conjuntos de treino e teste, essencial na avaliação de modelos.
- `pickle`: permite salvar e carregar objetos Python serializados, como datasets e modelos.

#### Instanciação de Transformadores

- `StandardScaler` e `LabelEncoder` foram instanciados para uso posterior nas transformações dos dados, visando normalização e codificação de variáveis categóricas.

#### Montagem do Google Drive

- A linha `drive.mount('/content/drive')` integra o ambiente do Google Colab com o Google Drive do usuário, permitindo acesso direto a arquivos armazenados na nuvem, como bases de dados em `.pkl` e modelos previamente salvos.

Essa etapa é fundamental para garantir que todas as ferramentas estejam devidamente carregadas e que o ambiente esteja preparado para executar as etapas seguintes da análise.

### Pipe 2 – Coleta e Conversão das Fontes de Dados

Nesta etapa, são realizadas as requisições à API **Alpha Vantage** para coleta de duas fontes de dados distintas: a variação diária do câmbio **USD/BRL** e os dados históricos diários do **Bitcoin em EUR**. Ambas as fontes serão posteriormente ajustadas para refletir apenas os **últimos 90 dias** na próxima etapa da pipeline.

---

#### Subpipe 2.1 – Coleta da Variação Diária USD/BRL com `FX_DAILY`

Utilizamos a função `FX_DAILY` da API para acessar a série temporal de câmbio entre o dólar americano (USD) e o real brasileiro (BRL). A requisição é feita por meio de uma URL específica:

```python
url_fx_currency = 'https://www.alphavantage.co/query?function=FX_DAILY&outputsize=full&from_symbol=USD&to_symbol=BRL&apikey=CHAVE'
```

A resposta em formato JSON é convertida para um DataFrame do pandas e ordenada cronologicamente pelas datas:

```
r = requests.get(url_fx_currency)
data = r.json()
fx_time_series = data['Time Series FX (Daily)']
fx_df = pd.DataFrame.from_dict(fx_time_series, orient='index')
fx_df.index = pd.to_datetime(fx_df.index)
fx_df = fx_df.sort_index()
```

A opção outputsize=full permite obter uma série histórica completa, e a filtragem para os últimos 90 dias será implementada na próxima pipe, garantindo maior flexibilidade no recorte temporal.

---

#### Subpipe 2.2 – Coleta dos Dados de Bitcoin com DIGITAL_CURRENCY_DAILY
A coleta dos dados do Bitcoin é feita utilizando o endpoint DIGITAL_CURRENCY_DAILY, com foco no par BTC/EUR:

```
url_btc = 'https://www.alphavantage.co/query?function=DIGITAL_CURRENCY_DAILY&symbol=BTC&market=EUR&apikey=CHAVE'
```

De forma semelhante à etapa anterior, a resposta é transformada em DataFrame com indexação temporal:

```
r = requests.get(url_btc)
data = r.json()
btc_time_series = data['Time Series (Digital Currency Daily)']
btc_df = pd.DataFrame.from_dict(btc_time_series, orient='index')
btc_df.index = pd.to_datetime(btc_df.index)
btc_df = btc_df.sort_index()
```

Essa estrutura permite o acesso direto às colunas de preços de abertura, fechamento, máxima, mínima e volume diário em EUR, sendo todos dados úteis para análise cruzada com a variação cambial.

---

#### Subpipe 2.3 – Alinhamento Temporal e Merge das Duas Fontes

Para permitir análises conjuntas entre o comportamento do Bitcoin e a variação cambial do dólar, as duas bases são unificadas por meio de um merge temporal pelas datas:

```
fx_btc_df = pd.merge(fx_df, btc_df, left_index=True, right_index=True, how='inner')
```

A opção how='inner' garante que apenas as datas presentes em ambas as fontes sejam consideradas, o que ajuda a evitar registros incompletos. Cada linha resultante do fx_btc_df representa um dia em que tanto o valor do dólar quanto do Bitcoin estavam disponíveis, permitindo comparações e análises correlacionais confiáveis.

Essa etapa é fundamental para garantir a qualidade e coerência temporal dos dados que alimentarão os modelos e análises subsequentes, estabelecendo uma base sólida e sincronizada entre as variáveis cambiais e criptoeconômicas.