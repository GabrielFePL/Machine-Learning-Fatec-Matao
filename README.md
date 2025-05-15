# Machine-Learning-Fatec-Matao
Este repositório contém materiais, códigos e anotações da disciplina Aprendizagem de Máquina da Fatec Matão - Luiz Marchesan.

## Descrição do Conteúdo Abordado

### 1. **Pré-Processamento de Dados**
O pré-processamento é uma etapa essencial para preparar os dados antes de aplicá-los aos modelos de Machine Learning. As técnicas utilizadas incluem:

#### 1.1 **Importação de Bibliotecas**
Foram utilizadas diversas bibliotecas para manipulação, análise e visualização de dados, como:
- **Pandas**: Manipulação de dados em DataFrames.
- **NumPy**: Operações matemáticas e manipulação de arrays.
- **Seaborn e Matplotlib**: Visualização de dados.
- **Plotly**: Criação de gráficos interativos.
- **Scikit-learn**: Pré-processamento, divisão de dados e construção de modelos.
- **Pickle**: Serialização de objetos para salvar e carregar dados.

#### 1.2 **Análise Exploratória**
- Estatísticas descritivas com `describe()` e `info()`.
- Verificação de valores nulos.
- Renomeação de colunas para corrigir inconsistências.
- Visualizações como histogramas, boxplots e gráficos de categorias paralelas para identificar padrões e distribuições.

#### 1.3 **Transformação de Dados**
- **Label Encoding**: Conversão de variáveis categóricas em valores numéricos inteiros.
- **One-Hot Encoding**: Representação binária de variáveis categóricas.
- **Padronização**: Uso de `StandardScaler` para centralizar os dados em média 0 e variância 1.

#### 1.4 **Divisão de Dados**
Os dados foram divididos em conjuntos de treinamento e teste utilizando `train_test_split`.

---

### 2. **Modelos de Machine Learning**
Três modelos principais foram treinados e avaliados:

#### 2.1 **Gaussian Naive Bayes**
- Modelo probabilístico baseado na suposição de independência entre os atributos.
- Implementado com `GaussianNB` do Scikit-learn.

#### 2.2 **Decision Tree Classifier**
- Modelo baseado em árvores de decisão, utilizando o critério de entropia para divisão.
- Implementado com `DecisionTreeClassifier`.

#### 2.3 **Random Forest Classifier**
- Conjunto de árvores de decisão para melhorar a precisão e reduzir o overfitting.
- Implementado com `RandomForestClassifier` com 196 estimadores.

---

### 3. **Avaliação de Modelos**
Os modelos foram avaliados com base em métricas de desempenho e visualizações:

#### 3.1 **Métricas de Avaliação**
- **Acurácia**: Proporção de previsões corretas.
- **Relatório de Classificação**: Inclui precisão, recall, F1-score e suporte para cada classe.
- **Probabilidade Condicional (P(A|B))**: Cálculo probabilístico para análise de desempenho.

#### 3.2 **Matriz de Confusão**
- Visualização gráfica das previsões corretas e incorretas.
- Implementada com `ConfusionMatrix` da biblioteca Yellowbrick.

---

### 4. **Visualizações Interativas**
- Gráficos de categorias paralelas e treemaps foram criados com Plotly para explorar relações entre variáveis categóricas e a variável alvo.

---

### 5. **Exportação de Dados**
Os dados pré-processados e divididos foram salvos em um arquivo `.pkl` para reutilização em diferentes etapas do projeto.

---

### 6. **Ferramentas e Bibliotecas Adicionais**
- **Yellowbrick**: Visualização de métricas de classificação.
- **Google Colab**: Ambiente de execução para o desenvolvimento do projeto.
- **API Alpha Vantage**: API para consumo e análise de dados (https://www.alphavantage.co/documentation/).

---

Este repositório contém projetos que demonstram fluxos completos de Machine Learning, desde o pré-processamento de dados até a avaliação de modelos, utilizando técnicas modernas e ferramentas amplamente adotadas na área.