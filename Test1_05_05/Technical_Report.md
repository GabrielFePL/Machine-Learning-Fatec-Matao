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

---

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

---

### Pipe 2 – Coleta e Conversão das Fontes de Dados

Nesta etapa, são realizadas as requisições à API **Alpha Vantage** para coleta de duas fontes de dados distintas: a variação diária do câmbio **USD/BRL** e os dados históricos diários do **Bitcoin em EUR**. Ambas as fontes serão posteriormente ajustadas para refletir apenas os **últimos 90 dias** na próxima etapa da pipeline.

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

#### Subpipe 2.3 – Alinhamento Temporal e Merge das Duas Fontes

Para permitir análises conjuntas entre o comportamento do Bitcoin e a variação cambial do dólar, as duas bases são unificadas por meio de um merge temporal pelas datas:

```
fx_btc_df = pd.merge(fx_df, btc_df, left_index=True, right_index=True, how='inner')
```

A opção how='inner' garante que apenas as datas presentes em ambas as fontes sejam consideradas, o que ajuda a evitar registros incompletos. Cada linha resultante do fx_btc_df representa um dia em que tanto o valor do dólar quanto do Bitcoin estavam disponíveis, permitindo comparações e análises correlacionais confiáveis.

Essa etapa é fundamental para garantir a qualidade e coerência temporal dos dados que alimentarão os modelos e análises subsequentes, estabelecendo uma base sólida e sincronizada entre as variáveis cambiais e criptoeconômicas.

---

### Pipe 3 – Aplicação de Requisitos da Prova

Com as bases de dados cambial e criptoeconômica devidamente integradas e ordenadas, esta etapa tem como objetivo aplicar os requisitos específicos propostos na prova, garantindo que o conjunto final de dados esteja pronto para análise conforme o escopo definido.

#### Subpipe 3.1 – Filtro do DataFrame para os Últimos 90 Dias

Neste subpipe, é implementado o recorte temporal dos dados, limitando a amostra aos **últimos 90 dias** disponíveis. Essa filtragem é realizada utilizando o método `tail(90)` do pandas:

```python
fx_btc_df = fx_btc_df.tail(90)
```

Esse comando seleciona as 90 linhas mais recentes do DataFrame, considerando a ordenação cronológica previamente aplicada na etapa de coleta e merge. Essa abordagem garante que os dados mais atuais sejam utilizados nas análises, alinhando-se à proposta de refletir as dinâmicas recentes do câmbio e da criptoeconomia.

A aplicação deste filtro é essencial para manter a relevância temporal da análise e evitar que dados muito antigos distorçam os resultados ou a interpretação de tendências recentes. Com isso, o foco permanece em um horizonte de tempo curto e mais significativo do ponto de vista econômico e financeiro.

### Subpipe 3.2 – Renomeação dos Atributos para Facilitar a Leitura

Após o recorte temporal, os nomes das colunas no DataFrame ainda seguem a nomenclatura original da API da Alpha Vantage, o que pode comprometer a legibilidade e a clareza do conjunto de dados. Assim, realiza-se a renomeação dos atributos com identificadores mais intuitivos e padronizados:

```
fx_btc_df = fx_btc_df.rename(columns={
    '1. open_x': 'fx_open',
    '2. high_x': 'fx_high',
    '3. low_x': 'fx_low',
    '4. close_x': 'fx_close',
    '1. open_y': 'btc_open',
    '2. high_y': 'btc_high',
    '3. low_y': 'btc_low',
    '4. close_y': 'btc_close',
    '5. volume': 'btc_volume'
})
```

Essa transformação tem como objetivo melhorar a usabilidade do dataset em análises subsequentes, visualizações e geração de relatórios. A renomeação adota o prefixo fx_ para atributos relativos ao câmbio USD/BRL e btc_ para os atributos referentes ao Bitcoin, promovendo uma identificação clara e rápida da origem de cada variável.

A clareza na nomenclatura é um fator importante para a manutenção e compreensão do código, especialmente em contextos de trabalho colaborativo ou avaliações acadêmicas, como é o caso desta prova.

### Subpipe 3.3 – Redefinição dos Tipos dos Dados

Com os dados já filtrados e os nomes das colunas devidamente ajustados, esta subetapa trata da conversão dos tipos de dados para formatos numéricos adequados às análises quantitativas que serão realizadas posteriormente.

Inicialmente, ao importar os dados da API e realizar o *merge*, todas as colunas numéricas permanecem como **strings** (tipo `object`), o que pode impedir a execução de operações matemáticas, cálculos estatísticos e visualizações apropriadas.

A verificação do tipo de dados é realizada com o comando:

```python
fx_btc_df.info()
```

Que apresenta a seguinte saída:

```
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 90 entries, 2024-12-30 to 2025-05-02
Data columns (total 9 columns):
 #   Column      Non-Null Count  Dtype 
---  ------      --------------  ----- 
 0   fx_open     90 non-null     object
 1   fx_high     90 non-null     object
 2   fx_low      90 non-null     object
 3   fx_close    90 non-null     object
 4   btc_open    90 non-null     object
 5   btc_high    90 non-null     object
 6   btc_low     90 non-null     object
 7   btc_close   90 non-null     object
 8   btc_volume  90 non-null     object
```

Para garantir a correta manipulação dessas colunas, realiza-se a conversão explícita para o tipo float, utilizando o método astype(float):

```
fx_btc_df = fx_btc_df.astype(float)
```

Após a conversão, uma nova verificação confirma a redefinição bem-sucedida dos tipos:

```
fx_btc_df.info()
```

Com a seguinte saída:

```
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 90 entries, 2024-12-30 to 2025-05-02
Data columns (total 9 columns):
 #   Column      Non-Null Count  Dtype  
---  ------      --------------  -----  
 0   fx_open     90 non-null     float64
 1   fx_high     90 non-null     float64
 2   fx_low      90 non-null     float64
 3   fx_close    90 non-null     float64
 4   btc_open    90 non-null     float64
 5   btc_high    90 non-null     float64
 6   btc_low     90 non-null     float64
 7   btc_close   90 non-null     float64
 8   btc_volume  90 non-null     float64
```

Essa etapa é fundamental para garantir a integridade e funcionalidade do DataFrame em análises, modelagens e visualizações futuras. Trabalhar com tipos numéricos permite o uso pleno das bibliotecas do ecossistema Python, além de evitar erros de execução decorrentes de tipos incompatíveis.

---

## Pipe 4 - Visualizações

Apesar de a visualização estar originalmente prevista para a etapa **"14. Avaliação do modelo e visualizações: linhas temporais, dispersão e heatmaps"**, sua execução foi antecipada nesta pipeline com o objetivo de **compreender melhor a natureza dos dados antes da construção de variáveis e do treinamento de modelos preditivos**. Esta decisão visa garantir uma abordagem mais informada e eficaz para as transformações e análises subsequentes.

### Resumo Estatístico e Estrutura dos Dados

- O método `fx_btc_df.describe()` foi utilizado para gerar **estatísticas descritivas** das colunas numéricas, permitindo uma visão inicial de tendências centrais (média, mediana), dispersão (desvio padrão) e possíveis valores discrepantes (mínimos e máximos).
- A inspeção com `fx_btc_df.info()` forneceu uma visão da **estrutura e integridade dos dados**, confirmando que não há valores nulos e que todas as colunas possuem o tipo `float64`, o que facilita o processamento e normalização subsequente.

### Normalização dos Dados

- A transformação dos dados com `StandardScaler` foi aplicada para **padronizar as variáveis** (média zero e desvio padrão um), o que é uma prática essencial para algoritmos de modelagem que são sensíveis à escala, além de auxiliar na geração de gráficos comparáveis entre variáveis com magnitudes distintas.
- O DataFrame `fx_btc_normalized_df` resultante foi criado para armazenar os dados normalizados, mantendo os nomes originais das colunas para facilitar a leitura e manipulação.

### Linha Temporal de Fechamento do Câmbio e Bitcoin e Volume do Bitcoin

- Foi iniciado o uso da biblioteca `matplotlib.pyplot` para **visualização temporal das variáveis `fx_close`, `btc_close` e `btc_volume`**, ou seja, o **preço de fechamento do câmbio USD/BRL e Bitcoin e volume do Bitcoin ao longo do tempo**.
- O gráfico de linha permite uma primeira percepção visual de **tendências, oscilações e possíveis sazonalidades**, servindo como base exploratória para a criação futura das variáveis como `variacao_cambio`, `variacao_btc` e `media_movel_btc`.

Essa abordagem exploratória precoce foi fundamental para identificar comportamentos relevantes nos dados e apoiar decisões mais estratégicas nas transformações que serão realizadas na próxima etapa da pipeline.

### Gráfico de Linhas – Série Temporal do Câmbio

```
plt.figure(figsize=(12, 5))
plt.plot(fx_btc_df.index, fx_btc_df['fx_close'], label='Preço de Fechamento de Câmbio USD/BRL', color='blue')
plt.plot(fx_btc_df.index, fx_btc_df['fx_high'], label='Preço Máximo de Câmbio USD/BRL', color='green')
plt.plot(fx_btc_df.index, fx_btc_df['fx_low'], label='Preço Mínimo de Câmbio USD/BRL', color='red')

plt.xlabel('Data')
plt.ylabel('Preço de Fechamento')
plt.title('Tendência do Preço de Fechamento de Câmbio USD/BRL')
plt.legend()
plt.grid()
plt.show()
```

O gráfico de linhas mostra a variação diária do câmbio USD/BRL ao longo do tempo.

Exibir os valores de máximo (fx_high), mínimo (fx_low) e fechamento (fx_close) permite identificar:

* Picos de volatilidade

* Períodos de estabilidade ou tendência

* Comportamentos sazonais ou atípicos (outliers visuais)

Essas informações são relevantes para criar variáveis derivadas, como variação percentual, amplitude ou média móvel.

### Boxplots – Distribuição das Variáveis de Câmbio

```
sns.boxplot(x=fx_btc_df['fx_open']);
sns.boxplot(x=fx_btc_df['fx_high']);
sns.boxplot(x=fx_btc_df['fx_low']);
sns.boxplot(x=fx_btc_df['fx_close']);
```

Os boxplots fornecem uma visão clara da distribuição estatística das variáveis de câmbio (fx_open, fx_high, fx_low, fx_close).

São úteis para:

* Identificar valores discrepantes (outliers)

* Avaliar a assimetria das distribuições

* Detectar possível necessidade de transformação (como normalização ou padronização)

Essa visualização detalhada dos dados de câmbio reforça a importância de entender a variabilidade e dispersão dos preços, o que pode impactar diretamente no desempenho de modelos preditivos, principalmente os sensíveis à escala ou à presença de valores extremos.

### Bitcoin (EUR)

Dando continuidade à fase de exploração visual dos dados, esta etapa foca na variável de criptoativo **Bitcoin**, cotado em **EUR**. Embora essas visualizações façam parte da etapa 14 ("Avaliação do modelo e visualizações"), elas foram antecipadas para apoiar o **entendimento da natureza e comportamento do ativo**, fundamental para decisões de modelagem, normalização e criação de features.

### Gráfico de Linhas – Preço de Fechamento, Máximo e Mínimo do Bitcoin

```
plt.figure(figsize=(12, 5))
plt.plot(fx_btc_df.index, fx_btc_df['btc_close'], label='Preço de Fechamento', color='orange')
plt.plot(fx_btc_df.index, fx_btc_df['btc_high'], label='Preço Máximo', color='green')
plt.plot(fx_btc_df.index, fx_btc_df['btc_low'], label='Preço Mínimo', color='red')

plt.xlabel('Data')
plt.ylabel('Preço de Fechamento')
plt.title('Tendência do Preço de Bitcoin EUR')
plt.legend()
plt.grid()
plt.show()
```

O gráfico temporal evidencia a volatilidade diária do Bitcoin.

Exibir os valores de fechamento, máximo e mínimo em conjunto permite observar:

* Oscilações abruptas

* Tendências de alta/baixa

* Eventos atípicos (como spikes)

Essa visualização é essencial para decisões sobre normalização, redução de variância e detecção de outliers.

### Gráfico de Linhas – Volume de Transações de Bitcoin

```
plt.figure(figsize=(12, 5))
plt.plot(fx_btc_df.index, fx_btc_df['btc_volume'], label='Volume de Transação', color='green')

plt.xlabel('Data')
plt.ylabel('Preço de Fechamento')
plt.title('Tendência do Volume de Transação de Bitcoin EUR')
plt.legend()
plt.grid()
plt.show()
```

O volume de transações é uma variável complementar e explicativa relevante.

Flutuações no volume podem antecipar movimentos de preço ou indicar picos de interesse/comércio, úteis para modelagem preditiva.

### Boxplots – Distribuições Estatísticas das Variáveis de Bitcoin

```
sns.boxplot(x=fx_btc_df['btc_open']);
sns.boxplot(x=fx_btc_df['btc_high']);
sns.boxplot(x=fx_btc_df['btc_low']);
sns.boxplot(x=fx_btc_df['btc_close']);
sns.boxplot(x=fx_btc_df['btc_volume']);
```

Os boxplots expõem a dispersão e presença de valores extremos nas variáveis relacionadas ao Bitcoin.

A variável btc_volume, por exemplo, tende a apresentar distribuição assimétrica com outliers visíveis, o que pode impactar modelos sensíveis à escala.

Essas visualizações antecipadas fornecem subsídios importantes para pré-processamento dos dados e definição de estratégias de engenharia de atributos. A observação cuidadosa das flutuações de preço e volume do Bitcoin contribui diretamente para o desenho de modelos mais robustos, além de facilitar a interpretação dos padrões temporais complexos desse ativo digital.

---

### Pipe 5 - Engenharia de Atributos
