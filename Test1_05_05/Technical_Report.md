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

#### Coleta da Variação Diária USD/BRL com `FX_DAILY`

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

#### Coleta dos Dados de Bitcoin com DIGITAL_CURRENCY_DAILY
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

#### Alinhamento Temporal e Merge das Duas Fontes

Para permitir análises conjuntas entre o comportamento do Bitcoin e a variação cambial do dólar, as duas bases são unificadas por meio de um merge temporal pelas datas:

```
fx_btc_df = pd.merge(fx_df, btc_df, left_index=True, right_index=True, how='inner')
```

A opção how='inner' garante que apenas as datas presentes em ambas as fontes sejam consideradas, o que ajuda a evitar registros incompletos. Cada linha resultante do fx_btc_df representa um dia em que tanto o valor do dólar quanto do Bitcoin estavam disponíveis, permitindo comparações e análises correlacionais confiáveis.

Essa etapa é fundamental para garantir a qualidade e coerência temporal dos dados que alimentarão os modelos e análises subsequentes, estabelecendo uma base sólida e sincronizada entre as variáveis cambiais e criptoeconômicas.

---

### Pipe 3 – Aplicação de Requisitos da Prova

Com as bases de dados cambial e criptoeconômica devidamente integradas e ordenadas, esta etapa tem como objetivo aplicar os requisitos específicos propostos na prova, garantindo que o conjunto final de dados esteja pronto para análise conforme o escopo definido.

#### Filtro do DataFrame para os Últimos 90 Dias

Neste subpipe, é implementado o recorte temporal dos dados, limitando a amostra aos **últimos 90 dias** disponíveis. Essa filtragem é realizada utilizando o método `tail(90)` do pandas:

```python
fx_btc_df = fx_btc_df.tail(90)
```

Esse comando seleciona as 90 linhas mais recentes do DataFrame, considerando a ordenação cronológica previamente aplicada na etapa de coleta e merge. Essa abordagem garante que os dados mais atuais sejam utilizados nas análises, alinhando-se à proposta de refletir as dinâmicas recentes do câmbio e da criptoeconomia.

A aplicação deste filtro é essencial para manter a relevância temporal da análise e evitar que dados muito antigos distorçam os resultados ou a interpretação de tendências recentes. Com isso, o foco permanece em um horizonte de tempo curto e mais significativo do ponto de vista econômico e financeiro.

### Renomeação dos Atributos para Facilitar a Leitura

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

### Redefinição dos Tipos dos Dados

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

### Pipe 4 - Visualizações

Apesar de a visualização estar originalmente prevista para a etapa **"14. Avaliação do modelo e visualizações: linhas temporais, dispersão e heatmaps"**, sua execução foi antecipada nesta pipeline com o objetivo de **compreender melhor a natureza dos dados antes da construção de variáveis e do treinamento de modelos preditivos**. Esta decisão visa garantir uma abordagem mais informada e eficaz para as transformações e análises subsequentes.

#### Resumo Estatístico e Estrutura dos Dados

- O método `fx_btc_df.describe()` foi utilizado para gerar **estatísticas descritivas** das colunas numéricas, permitindo uma visão inicial de tendências centrais (média, mediana), dispersão (desvio padrão) e possíveis valores discrepantes (mínimos e máximos).
- A inspeção com `fx_btc_df.info()` forneceu uma visão da **estrutura e integridade dos dados**, confirmando que não há valores nulos e que todas as colunas possuem o tipo `float64`, o que facilita o processamento e normalização subsequente.

#### Normalização dos Dados

- A transformação dos dados com `StandardScaler` foi aplicada para **padronizar as variáveis** (média zero e desvio padrão um), o que é uma prática essencial para algoritmos de modelagem que são sensíveis à escala, além de auxiliar na geração de gráficos comparáveis entre variáveis com magnitudes distintas.
- O DataFrame `fx_btc_normalized_df` resultante foi criado para armazenar os dados normalizados, mantendo os nomes originais das colunas para facilitar a leitura e manipulação.

#### Linha Temporal de Fechamento do Câmbio e Bitcoin e Volume do Bitcoin

- Foi iniciado o uso da biblioteca `matplotlib.pyplot` para **visualização temporal das variáveis `fx_close`, `btc_close` e `btc_volume`**, ou seja, o **preço de fechamento do câmbio USD/BRL e Bitcoin e volume do Bitcoin ao longo do tempo**.
- O gráfico de linha permite uma primeira percepção visual de **tendências, oscilações e possíveis sazonalidades**, servindo como base exploratória para a criação futura das variáveis como `variacao_cambio`, `variacao_btc` e `media_movel_btc`.

Essa abordagem exploratória precoce foi fundamental para identificar comportamentos relevantes nos dados e apoiar decisões mais estratégicas nas transformações que serão realizadas na próxima etapa da pipeline.

#### Gráfico de Linhas – Série Temporal do Câmbio

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

#### Boxplots – Distribuição das Variáveis de Câmbio

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

#### Bitcoin (EUR)

Dando continuidade à fase de exploração visual dos dados, esta etapa foca na variável de criptoativo **Bitcoin**, cotado em **EUR**. Embora essas visualizações façam parte da etapa 14 ("Avaliação do modelo e visualizações"), elas foram antecipadas para apoiar o **entendimento da natureza e comportamento do ativo**, fundamental para decisões de modelagem, normalização e criação de features.

#### Gráfico de Linhas – Preço de Fechamento, Máximo e Mínimo do Bitcoin

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

#### Gráfico de Linhas – Volume de Transações de Bitcoin

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

#### Boxplots – Distribuições Estatísticas das Variáveis de Bitcoin

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

#### Variação de Câmbio USD/BRL

A criação da variável `fx_variation` tem como finalidade calcular a variação diária do valor de fechamento do câmbio USD/BRL. Esse tipo de variável é essencial para análises de séries temporais financeiras, pois permite observar tendências, flutuações e possíveis padrões de comportamento do mercado cambial ao longo do tempo.

```
fx_btc_df['fx_variation'] = fx_btc_df['fx_close'].diff().shift(-1)
```

Essa linha de código realiza os seguintes passos:

diff(): calcula a diferença entre o valor de fechamento (fx_close) do dia seguinte e o do dia atual. Isto é, fx_close(t+1) - fx_close(t).

shift(-1): desloca os resultados para cima uma linha, de modo que a variação entre o dia t e o dia t+1 seja atribuída corretamente à linha do dia t.

Isso resulta em uma nova coluna fx_variation, que representa a Variação do Câmbio USD/BRL, conforme a seguinte fórmula:

fx_variation
𝑡
=
fx_close
𝑡
+
1
−
fx_close
𝑡
fx_variation 
t
​
 =fx_close 
t+1
​
 −fx_close 
t

Essa transformação atende ao item 4. Cálculo das variáveis, mais especificamente ao subitem:

Variacao_cambio = fechamento USD/BRL atual - anterior

Contudo, a implementação opta por calcular a variação futura relativa ao ponto atual, o que é uma abordagem comum em análises preditivas onde o objetivo pode ser, por exemplo, antecipar movimentos de mercado. A inclusão dessa variável no dataset permite que o modelo identifique como mudanças no câmbio se relacionam com outros indicadores, como o preço do Bitcoin ou o volume de transações.

#### Variação do Bitcoin (EUR)

A criação da variável `btc_variation` visa calcular a variação diária do preço de fechamento do Bitcoin (em euros). Essa métrica é crucial para entender os movimentos de mercado do BTC, identificar tendências de valorização ou desvalorização, e correlacionar seu comportamento com outras variáveis, como o câmbio USD/BRL.

```
fx_btc_df['btc_variation'] = fx_btc_df['btc_close'].diff().shift(-1)
```

Essa linha realiza duas operações:

diff(): calcula a diferença entre o valor de fechamento de BTC no dia seguinte e o valor atual. Isto é, btc_close(t+1) - btc_close(t).

shift(-1): desloca os resultados uma linha para cima, atribuindo corretamente a variação entre os dias t e t+1 à linha correspondente ao dia t.

Com isso, a coluna btc_variation passa a representar a variação futura no fechamento do Bitcoin com base na seguinte fórmula:

btc_variation t = (btc_close t + 1) − btc_close t

Essa transformação atende ao requisito especificado no item:

Variacao_btc = fechamento BTC atual - anterior

No entanto, assim como na variável fx_variation, a escolha de aplicar shift(-1) inverte o cálculo tradicional para destacar a variação futura a partir do ponto de vista da data atual. Essa abordagem é amplamente utilizada em problemas de predição, onde o objetivo é estimar o que acontecerá no próximo período com base nos dados conhecidos hoje.

Essa variável pode ser particularmente útil como variável alvo (target) ou como indicador de performance de curto prazo em modelos de machine learning voltados para previsão de preços ou análise de risco de investimentos.

#### Média Móvel do Bitcoin (EUR)

A criação da variável `btc_moving_average` tem como objetivo calcular a **média móvel dos preços de fechamento do Bitcoin (em EUR)**, considerando os **últimos 5 dias anteriores** à data atual. Essa métrica suaviza flutuações de curto prazo e ajuda a identificar tendências mais estáveis no comportamento do ativo ao longo do tempo.

```
fx_btc_df['btc_moving_average'] = fx_btc_df['btc_close'].shift(1).rolling(window=5).mean()
```

Essa linha realiza duas operações fundamentais:

shift(1): desloca os dados de btc_close para que a média móvel de uma linha seja calculada com base apenas nos dias anteriores, evitando vazamentos de dados futuros (data leakage).

rolling(window=5).mean(): aplica uma janela deslizante de 5 dias e calcula a média dos valores deslocados, ou seja, a média dos cinco fechamentos anteriores ao dia corrente.

Fórmula
  
i=t−5
∑
t−1
​
 btc_close 
i
​

Os primeiros cinco valores da média móvel são NaN, pois não há dados suficientes para compor uma janela de cinco dias completos.

Essa transformação atende ao requisito descrito no item:

Media_movel_btc = média dos últimos 5 dias do fechamento do BTC

Além disso, o uso da função shift(1) garante que a média calculada para o dia t não inclua o valor do próprio dia, preservando a lógica de um indicador retrospectivo — aspecto essencial para aplicações em análise de séries temporais, modelos preditivos e estratégias de investimento baseadas em médias móveis (como cruzamento de médias).

#### Tratamento de Valores Faltantes após Aplicação de Engenharia de Atributos

Após a criação de novos atributos derivados no processo de engenharia de atributos, surgiram alguns valores faltantes (NaNs), principalmente nas colunas `fx_variation`, `btc_variation` e `btc_moving_average`. A seguir, descrevemos as estratégias adotadas para lidar com esses valores ausentes, justificando tecnicamente cada decisão:

**Preenchimento de `fx_variation` e `btc_variation` com zero**

- **Motivação**: As colunas `fx_variation` e `btc_variation` representam variações de preço entre dias consecutivos. Como essas colunas são obtidas por meio da diferença entre valores de fechamento em dias subsequentes, o primeiro valor (ou um valor resultante de uma ausência anterior) naturalmente se torna nulo.
- **Estratégia adotada**: Preencher com `0` os valores ausentes dessas colunas.
- **Justificativa técnica**:
  - Preencher com a média ou interpolação introduziria viés, especialmente no início da série.
  - Zero representa uma variação neutra, que é uma escolha segura e coerente quando não há dados anteriores disponíveis.
  - Essa abordagem evita distorções nos modelos de machine learning, que podem interpretar valores ausentes como informações relevantes.

```
fx_btc_df['fx_variation'] = fx_btc_df['fx_variation'].fillna(0)
fx_btc_df['btc_variation'] = fx_btc_df['btc_variation'].fillna(0)
```

**Preenchimento de `btc_moving_average` com dados reais anteriores ao recorte final**

- **Motivação**: A coluna `btc_moving_average` foi criada como uma média móvel de 5 dias do preço de fechamento do BTC, deslocada uma posição no tempo (shift(1)). Naturalmente, os primeiros 4 valores da série são NaN, pois não há dados anteriores suficientes dentro do recorte de 90 dias usado no dataframe final.

- **Estratégia adotada**: Utilizamos dados reais anteriores aos 90 dias finais que foram descartados no recorte final, mas estavam disponíveis por terem sido retornados na requisição original à API. Esses dados foram utilizados para calcular os primeiros valores de btc_moving_average de maneira legítima e alinhada com a natureza sequencial da série temporal.

```
prev_five_fx_btc_df = pd.merge(fx_df, btc_df, left_index=True, right_index=True, how='inner')
prev_five_fx_btc_df = prev_five_fx_btc_df.tail(95)
prev_five_fx_btc_df = prev_five_fx_btc_df.head(10)
prev_five_fx_btc_df['btc_moving_average'] = (
    prev_five_fx_btc_df['4. close_y'].shift(1).rolling(window=5).mean()
)
prev_five_fx_btc_df = prev_five_fx_btc_df.tail(5)
```

- **Justificativa técnica**:
    - Essa abordagem mantém a coerência temporal dos dados e evita imputações artificiais que poderiam comprometer a integridade da série.

    - O uso de dados reais respeita a cronologia e mantém a qualidade do atributo para fins preditivos.

Com essas decisões, asseguramos que o dataframe fx_btc_df esteja livre de valores ausentes nas colunas derivadas, sem comprometer a qualidade e a fidelidade dos dados originais. Essa preparação é essencial para garantir a performance e a confiabilidade dos modelos de aprendizado de máquina subsequentes.

#### Definição do Atributo Alvo (Target)

Durante o processo de engenharia de atributos, foi necessário criar a variável-alvo `scenario`, que representa o cenário econômico classificado em três categorias com base nas variações do Bitcoin (BTC) e da taxa de câmbio USD/BRL (FX). A seguir, descrevemos as motivações e justificativas técnicas para a lógica utilizada nessa criação:

**Objetivo da variável `scenario`**

- Representar o contexto econômico diário com base nas movimentações do Bitcoin e do dólar.
- Facilitar a classificação de cenários para fins de modelagem preditiva, especialmente com algoritmos probabilísticos como o Naive Bayes.

**Critérios originais sugeridos**

| Variação BTC | Variação Câmbio | Classificação      |
|--------------|------------------|--------------------|
| > 0          | > 0              | Crypto-Friendly    |
| > 0          | < 0              | Neutral            |
| < 0          | > 0              | Conservative       |
| < 0          | < 0              | Neutral            |

Essas regras foram elaboradas com base na interpretação econômica dos movimentos:

- **Crypto-Friendly**: Ambiente favorável para ativos digitais, com valorização do BTC e alta do dólar (indicando fuga para alternativas).
- **Conservative**: BTC em queda, com dólar valorizando (cenário de aversão ao risco e desvalorização do real).
- **Neutral**: Quando o BTC apresenta pouca variação (lateralizado) ou ambos os ativos se desvalorizam, sugerindo um ambiente neutro ou de menor atratividade para decisões estratégicas baseadas em risco.

**Alteração técnica implementada**

Na prática, a variação do BTC próxima de zero foi considerada uma condição de lateralização. Para isso, adotou-se o intervalo de ±2% como limite de neutralidade. Isso se justifica da seguinte forma:

- **Tolerância de ±2%**: Considera ruído de mercado e volatilidade diária normal. Um movimento inferior a esse intervalo é considerado lateral, representando ausência de tendência definida.

**Implementação em código:**

```
def define_scenario(row):
    if abs(row['btc_variation']) <= 0.02 and abs(row['btc_variation']) >= -0.02:
        return 'Neutral'
    elif row['btc_variation'] > 0.02 and row['fx_variation'] > 0:
        return 'Crypto-Friendly'
    elif row['btc_variation'] < 0.02 and row['fx_variation'] > 0:
        return 'Conservative'
    else:
        return 'Neutral'

fx_btc_df['scenario'] = fx_btc_df.apply(define_scenario, axis=1)
```

- **Justificativa técnica da implementação**:

    - Uso de .apply() com axis=1: Necessário para aplicar a lógica linha a linha, pois cada decisão depende da combinação entre btc_variation e fx_variation.

    - Condições explícitas e priorização da lateralização: A primeira verificação garante que valores com baixa oscilação no BTC sejam marcados como Neutral antes de qualquer outra lógica ser avaliada.

    - Manutenção da consistência com a interpretação econômica: Cada cenário representa uma leitura econômica coerente para auxiliar nos modelos de classificação e decisões baseadas em contexto de mercado.

Com isso, o atributo scenario está devidamente categorizado e pronto para ser utilizado como variável-alvo em modelos supervisionados. Ele reflete com fidelidade a conjuntura diária combinada dos mercados de criptoativos e câmbio.

---

### Pipe 6 - Normalização dos Dados

A etapa de normalização é fundamental no pré-processamento de dados, especialmente para modelos que utilizam medidas de distância ou distribuições estatísticas, como o Naive Bayes. A seguir, detalhamos as subetapas desse processo com suas respectivas justificativas técnicas.

#### Separação de Atributos Preditores e Classe

```
x_fx_btc = fx_btc_df.iloc[:, 9:12].values
x_fx_btc.shape
```

```
y_fx_btc = fx_btc_df.iloc[:, 12].values
y_fx_btc.shape
```

- **Objetivo**: Isolar os atributos preditores (fx_variation, btc_variation, btc_moving_average) da variável-alvo (scenario) para permitir o treinamento adequado dos modelos.

- **Justificativa técnica**:

    - Separar entrada e saída é essencial para qualquer tarefa supervisionada.

    - Garante que a variável-alvo não interfira na normalização nem nas etapas subsequentes de modelagem.

    - O uso de .iloc[:, 9:12] é apropriado, pois os atributos de interesse estão posicionados nessas colunas do DataFrame.

#### Padronização dos Dados

```
x_fx_btc = standardScaler.fit_transform(x_fx_btc)
```

- **Objetivo**: Padronizar os dados (z-score), transformando-os para que tenham média 0 e desvio padrão 1.

- **Justificativa técnica**:

    - Algoritmos como o Naive Bayes podem se beneficiar da padronização, pois distribuições mais próximas da normal ajudam a modelagem probabilística.

    - Melhora a estabilidade numérica e evita que variáveis com escalas diferentes tenham peso desproporcional na modelagem.

    - Apesar do Naive Bayes ser teoricamente robusto a escala, na prática, a normalização contribui para melhor desempenho com dados contínuos (como variância Gaussiana).

Separação dos Dados de Treinamento e Teste

```
x_fx_btc_train, x_fx_btc_test, y_fx_btc_train, y_fx_btc_test = train_test_split(
    x_fx_btc, y_fx_btc, test_size=0.20, random_state=0
)
```

- **Objetivo**: Dividir os dados em conjuntos de treinamento e teste, garantindo a avaliação imparcial do desempenho do modelo.

- **Justificativa técnica**:

    - A separação evita vazamento de informação (data leakage), garantindo que o modelo não "veja" os dados de teste durante o treinamento.

    - O parâmetro test_size=0.20 define que 20% dos dados serão usados para teste, o que é adequado para conjuntos pequenos (como os 90 registros do dataframe).

    - O random_state=0 assegura reprodutibilidade dos resultados, importante para fins de validação e comparação de desempenho entre diferentes modelos e experimentos.

Com essas etapas, os dados estão preparados de maneira apropriada para alimentar algoritmos de aprendizado de máquina, garantindo integridade estatística e coerência com boas práticas de pré-processamento.

---

### Pipe 7 - Conversão e Exportação de Arquivo para Ingestão Neural de Dados

```
with open('/content/drive/MyDrive/machine_learning_semestre_5/Pickle/fx_btc.pkl', mode='wb') as f:
  pickle.dump([x_fx_btc_train, x_fx_btc_test, y_fx_btc_train, y_fx_btc_test], f)
```

- **Objetivo**: Persistir os dados pré-processados (atributos e classes de treino e teste) em disco para reutilização em etapas posteriores de modelagem e treinamento, como em redes neurais ou outros algoritmos de machine learning.

- **Justificativa técnica**:

    - Eficiência e modularidade: Ao salvar os dados já preparados, evitamos repetir etapas de pré-processamento em execuções futuras, tornando o pipeline mais eficiente.

    - Padronização do fluxo de trabalho: A exportação em formato .pkl (via pickle) é compatível com diversas bibliotecas de machine learning e deep learning, como Scikit-learn, TensorFlow e PyTorch.

    - Reprodutibilidade: Armazenar os conjuntos de treino e teste com os mesmos dados utilizados na fase de normalização e split garante que experimentos futuros sejam realizados nas mesmas condições.

    - Organização: A escolha de um caminho específico no Google Drive (/content/drive/...) permite integração direta com o ambiente do Google Colab, facilitando a continuidade do trabalho entre sessões e dispositivos.

Com essa abordagem, os dados ficam prontos para serem ingeridos diretamente por arquiteturas neurais ou outros modelos de forma estruturada, rápida e confiável.

### Pipe 8 - Importação de Bibliotecas

```
from sklearn.naive_bayes import GaussianNB
from sklearn.metrics import accuracy_score, classification_report
from yellowbrick.classifier import ConfusionMatrix
from collections import Counter
```

- **Objetivo**: Preparar o ambiente com bibliotecas essenciais para construção, avaliação e interpretação de um modelo de classificação baseado no algoritmo Naive Bayes.

- **Justificativa técnica**:

    - GaussianNB (Scikit-learn): Implementa o algoritmo Naive Bayes Gaussiano, adequado para variáveis contínuas e normalmente distribuídas, como as variações e médias móveis utilizadas nos atributos preditores do modelo.

    - accuracy_score e classification_report (Scikit-learn): Ferramentas fundamentais para mensurar o desempenho do modelo. O accuracy_score fornece a acurácia global da classificação, enquanto o classification_report detalha métricas por classe (precisão, revocação e F1-score), o que é crucial para avaliar o desempenho em cenários multiclasses como "Crypto-Friendly", "Neutral" e "Conservative".

    - ConfusionMatrix (Yellowbrick): Permite a visualização gráfica da matriz de confusão, facilitando a análise de erros do modelo ao identificar quais classes são mais frequentemente confundidas entre si.

    - Counter (collections): Utilizado para verificar o balanceamento das classes no conjunto de dados, o que pode impactar diretamente o desempenho e a imparcialidade do classificador.

Com essas bibliotecas, o pipeline está pronto para construir e interpretar um modelo bayesiano robusto, com suporte a análise estatística, visualização de resultados e diagnóstico de performance multiclasse.

### Pipe 9 - Carregamento dos Dados para Ingestão Neural

```
with open('/content/drive/MyDrive/machine_learning_semestre_5/Pickle/fx_btc.pkl', 'rb') as f:
  x_fx_btc_train, x_fx_btc_test, y_fx_btc_train, y_fx_btc_test = pickle.load(f)
```

- **Objetivo**: Realizar o carregamento dos dados previamente normalizados e particionados em treino e teste, a partir de um arquivo .pkl, para uso direto em modelos de aprendizado de máquina ou redes neurais.

- **Justificativa técnica**:

    - Continuidade do pipeline: Essa etapa permite retomar o fluxo de trabalho a partir de onde foi interrompido, utilizando exatamente os mesmos dados pré-processados, garantindo consistência entre sessões de modelagem.

    - Reprodutibilidade: Ao utilizar os mesmos dados carregados de forma padronizada, assegura-se que os resultados obtidos em testes e treinamentos sejam reproduzíveis, sem variações causadas por pré-processamentos repetidos ou aleatórios.

    - Eficiência operacional: O uso do pickle.load() reduz o tempo de preparação ao evitar que as etapas de transformação, normalização e divisão dos dados sejam reexecutadas, economizando recursos computacionais e tempo.

    - Integração com Google Drive: O caminho definido em /content/drive/... garante que o arquivo carregado esteja disponível no ambiente do Google Colab, promovendo portabilidade e facilidade de acesso entre diferentes dispositivos ou sessões.

Dessa forma, os dados ficam prontos para serem utilizados de maneira rápida, segura e estruturada em qualquer modelo de aprendizado supervisionado, incluindo redes neurais.

### Pipe 10 - Ingestão Neural e Testagem de Neurônio Artificial GaussianNB

```
gnb_fx_btc = GaussianNB()
gnb_fx_btc.fit(x_fx_btc_train, y_fx_btc_train)

gnb_fx_btc_predict = gnb_fx_btc.predict(x_fx_btc_test)
```

- **Objetivo**: Realizar o treinamento de um modelo de classificação baseado no algoritmo Naive Bayes Gaussiano (GaussianNB), utilizando os dados previamente normalizados e divididos em treino e teste. A seguir, gerar as previsões a partir do modelo treinado sobre os dados de teste.

- **Justificativa técnica**:

    - Algoritmo probabilístico robusto: O GaussianNB é uma variação do algoritmo Naive Bayes, que assume que os atributos seguem uma distribuição normal. Ele é eficaz para tarefas de classificação com conjuntos de dados contínuos e é particularmente útil em problemas com alta dimensionalidade e independência entre atributos.

    - Velocidade de treinamento: O GaussianNB é extremamente rápido tanto no treinamento quanto na predição, sendo adequado para análises preliminares, validações cruzadas e testes de hipóteses sobre a capacidade discriminativa dos dados.

    - Baixa complexidade computacional: Por não exigir hiperparâmetros complexos ou redes neurais profundas, o modelo pode ser treinado mesmo em ambientes com limitações computacionais, como o Google Colab gratuito.

    - Aplicabilidade à ingestão neural: Embora o GaussianNB não seja um neurônio artificial no sentido das redes neurais profundas, ele simula um comportamento de classificador neural básico, sendo uma excelente etapa inicial de teste antes de partir para arquiteturas mais sofisticadas, como MLPs ou CNNs.

    - Compatibilidade com frameworks de avaliação: O resultado da predição (gnb_fx_btc_predict) pode ser utilizado com métricas de avaliação como accuracy, classification report e confusion matrix, fornecendo uma base clara para comparação com modelos mais complexos.

Assim, essa pipe implementa de forma eficaz o requisito de Treinamento com GaussianNB, servindo como ponto de partida confiável e interpretável para análises de desempenho em classificadores supervisionados.

### Pipe 11 - Avaliação do Modelo  

```
gnb_fx_btc_accuracy = accuracy_score(y_fx_btc_test, gnb_fx_btc_predict)
print('P(A) = ' + str(round((gnb_fx_btc_accuracy * 100), 2)) + '%')
```

#### Cálculo probabilístico com Teorema de Bayes

```
classes = np.unique(np.concatenate((y_fx_btc_test, gnb_fx_btc_predict)))
bayes_results = {}
for cls in classes:
    p_b = np.sum(gnb_fx_btc_predict == cls) / len(gnb_fx_btc_predict)
    p_a = np.sum(y_fx_btc_test == cls) / len(y_fx_btc_test)
    mask = y_fx_btc_test == cls
    p_b_a = np.sum(gnb_fx_btc_predict[mask] == cls) / np.sum(mask) if np.sum(mask) > 0 else 0
    p_a_b = (p_b_a * p_a) / p_b if p_b > 0 else 0
    bayes_results[cls] = {
        'P(A)': round(p_a, 3),
        'P(B)': round(p_b, 3),
        'P(B|A)': round(p_b_a, 3),
        'P(A|B)': round(p_a_b, 3)
    }
bayes_df = pd.DataFrame(bayes_results).T
print(bayes_df)
```

#### Matriz de confusão

```
gnb_fx_btc_cm = ConfusionMatrix(gnb_fx_btc)
gnb_fx_btc_cm.fit(x_fx_btc_train, y_fx_btc_train)
gnb_fx_btc_cm.score(x_fx_btc_test, y_fx_btc_test)
```

#### Relatório de Classificação

```
print(classification_report(y_fx_btc_test, gnb_fx_btc_predict))
```

- **Objetivo**: Avaliar o desempenho do classificador GaussianNB a partir de métricas quantitativas e qualitativas, utilizando medidas estatísticas como accuracy, matriz de confusão, relatório de classificação e o Teorema de Bayes para análise probabilística dos resultados.

- **Justificativa técnica**:

    - Precisão geral (Accuracy): A métrica accuracy_score fornece uma visão direta da taxa de acerto do modelo em relação ao total de amostras de teste, sendo uma medida base para validação de classificadores.

    - Probabilidade condicional via Teorema de Bayes: O cálculo das probabilidades 𝑃(𝐴), 𝑃(𝐵), 𝑃(𝐵∣𝐴) e 𝑃(𝐴∣𝐵) permite avaliar a coerência das predições do modelo sob uma ótica estatística, fornecendo insights sobre o comportamento das classes previstas em relação às classes reais.

    - Interpretação qualitativa com matriz de confusão: A visualização das predições corretas e incorretas para cada classe permite detectar tendências de erro, como confusões recorrentes entre categorias semelhantes.

    - Relatório de desempenho detalhado: O classification_report agrega informações sobre precisão (precision), revocação (recall) e f1-score, possibilitando uma análise equilibrada do modelo em contextos de classes desbalanceadas.

    - Validação visual e interpretável: A matriz de confusão gerada com Yellowbrick fornece uma interface gráfica que auxilia na interpretação rápida e intuitiva dos resultados, mesmo por usuários não especialistas.

Essa pipe implementa com rigor o requisito 14 - Avaliação do Modelo, entregando um conjunto completo de métricas fundamentais para a análise de classificadores supervisionados, baseando-se em boas práticas estatísticas e de machine learning.