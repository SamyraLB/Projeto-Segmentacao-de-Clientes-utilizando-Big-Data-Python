# Projeto-Segmentacao-de-Clientes-utilizando-Big-Data-Python
#  Segmentação de Clientes por Comportamento de Compra

> Clusterização de clientes B2B de um distribuidor de insumos para gastronomia, confeitaria e panificação, a partir do histórico de compras por subcategoria de produto.

![Python](https://img.shields.io/badge/Python-3.10-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-KMeans%20%7C%20PCA-orange)
![Plotly](https://img.shields.io/badge/Plotly-Visualiza%C3%A7%C3%A3o%20interativa-3F4F75)
![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen)

---

##  Sumário

1. [Contexto de negócio](#-contexto-de-negócio)
2. [Objetivos](#-objetivos)
3. [Perguntas de negócio](#-perguntas-de-negócio)
4. [Dados](#-dados)
5. [Visão geral da base](#-visão-geral-da-base)
6. [Metodologia](#-metodologia)
7. [Resultados](#-resultados)
8. [Aplicações comerciais](#-aplicações-comerciais)
9. [Limitações e próximos passos](#-limitações-e-próximos-passos)
10. [Como reproduzir](#-como-reproduzir)
11. [Estrutura do repositório](#-estrutura-do-repositório)
12. [Autor](#-autor)

---

##  Contexto de negócio

A empresa atende mais de **2 mil clientes** com um portfólio amplo (12 seções, 134 categorias e 514 subcategorias), que vai de congelados e gastronomia a confeitaria e panificação. Tratar todos esses clientes como um bloco único leva a:

- campanhas de marketing genéricas, com baixa taxa de resposta;
- dificuldade de identificar oportunidades de **venda cruzada** (cross-sell);
- pouca visibilidade sobre quais perfis de cliente sustentam o faturamento e quais compram de forma limitada.

Este projeto responde a esse problema agrupando clientes por **semelhança de cesta de compras**, para que o time comercial atue de forma diferente em cada grupo.

##  Objetivos

**Objetivo geral:** segmentar a base de clientes em grupos com padrões de compra semelhantes, gerando insumos para decisões comerciais e de marketing.

**Objetivos específicos:**

- Entender o perfil da base: clientes, pedidos, seções, categorias e subcategorias.
- Transformar o histórico transacional em uma representação por cliente (presença/ausência de cada subcategoria comprada).
- Definir o número de segmentos com apoio de técnicas quantitativas (método do cotovelo e dendrograma).
- Treinar o modelo de clusterização e validá-lo com métrica de qualidade (coeficiente de silhueta).
- Perfilar cada segmento pelos produtos mais recorrentes e pelo tamanho na base.
- Traduzir os segmentos em ações práticas de análise comercial.

##  Perguntas de negócio

- Quantos perfis distintos de compra existem na base?
- Qual o peso de cada perfil (% de clientes)?
- Quais produtos "definem" cada perfil?
- Onde há espaço para ampliar o mix de clientes que compram pouco?
- Que ações de marketing e relacionamento fazem sentido por segmento?

##  Dados

Arquivo de entrada: `BASE_FINAL.xlsx` (histórico de pedidos, **92.444 linhas × 12 colunas**), com período de **01/10/2022 a 31/12/2022** (trimestre).

| Coluna | Descrição |
|---|---|
| `DTFAT` | Data do pedido |
| `CODCLI` | Código identificador do cliente |
| `CHAVE_SUB` | Concatenação de `CODSEC`, `CODCATEGORIA` e `CODSUBCATEGORIA` (chave do produto) |
| `CODSEC` / `NOME_SECAO` | Código e nome da seção |
| `CODCATEGORIA` / `NOME_CATEGORIA` | Código e nome da categoria |
| `CODSUBCATEGORIA` / `NOME_SUBCATEGORIA` | Código e nome da subcategoria |
| `PESOLIQ` | Peso referente à compra |

>  A base contém dados comerciais reais de clientes. Por isso, ela **não é versionada** neste repositório. Veja a seção [Como reproduzir](#-como-reproduzir).

>  Neste projeto, "produto" corresponde a uma **subcategoria** (`CHAVE_SUB`), e "pedido" corresponde a um par único **data + cliente**.

##  Visão geral da base

| Indicador | Valor |
|---|---|
| Clientes únicos | **2.059** |
| Pedidos | **16.903** |
| Seções | **12** |
| Categorias | **134** |
| Produtos (subcategorias) | **514** |
| Produtos distintos por cliente (mediana) | **9** |

A mediana de 9 subcategorias por cliente, contra 514 disponíveis, indica que a maior parte dos clientes compra uma fração pequena do portfólio. Isso é um sinal direto de potencial de ampliação de mix.

##  Metodologia

```text
Dados brutos → Limpeza/bases auxiliares → Cesta por cliente → Vetor binário
      → Definição de K (cotovelo + dendrograma) → PCA (2D) → K-means
      → Validação (silhueta) → Perfilamento dos clusters
```

### 1. Leitura e análise exploratória
Leitura do Excel com `pandas` e criação de bases auxiliares (clientes, pedidos, seções, categorias, produtos e relação cliente × produto) para calcular os indicadores da base.

### 2. Representação dos clientes
- Agrupamento das compras por cliente com `groupby` + `apply(list)`, gerando uma lista de subcategorias por cliente.
- Conversão com `MultiLabelBinarizer` (scikit-learn) para vetores binários: cada coluna é uma subcategoria e cada valor indica se o cliente **comprou (1)** ou **não comprou (0)**.

### 3. Escolha do número de clusters
- **Método do cotovelo:** inércia do K-means para K de 1 a 29. A curva perde inclinação a partir de aproximadamente 5 a 10 clusters.
- **Dendrograma hierárquico:** matriz de distâncias com **índice de Jaccard** (adequado a dados binários de presença/ausência) e ligação de Ward.
- Decisão: **K = 13**, buscando equilíbrio entre detalhamento dos perfis e interpretabilidade.

### 4. Clusterização
- Redução de dimensionalidade com **PCA (2 componentes)**.
- Treino do **K-means com 13 clusters**.
- Visualização interativa dos clientes com **Plotly**, com *hover* mostrando código do cliente e categorias compradas.

### 5. Validação
- **Coeficiente de silhueta: 0,38**, indicando estrutura de grupos moderada, com sobreposição entre alguns clusters (esperado em dados de compra B2B com itens básicos comuns a todos).

### 6. Perfilamento
- Tamanho de cada cluster (nº e % de clientes).
- Subcategorias mais recorrentes por cluster, cruzando `CLUSTER` com a base transacional.

##  Resultados

### Distribuição dos clientes por cluster

| Cluster | Clientes | % da base |
|---:|---:|---:|
| 10 | 602 | 29,2% |
| 1 | 344 | 16,7% |
| 12 | 219 | 10,6% |
| 7 | 189 | 9,2% |
| 2 | 129 | 6,3% |
| 9 | 94 | 4,6% |
| 11 | 94 | 4,6% |
| 4 | 87 | 4,2% |
| 3 | 84 | 4,1% |
| 0 | 71 | 3,4% |
| 5 | 60 | 2,9% |
| 8 | 57 | 2,8% |
| 6 | 29 | 1,4% |

**Leitura comercial:**

- Os **3 maiores clusters (10, 1 e 12) concentram ~56% dos clientes**, e o cluster 10 sozinho reúne quase 30%. São a base natural para campanhas de maior escala.
- Há uma "cauda" de clusters pequenos (cada um com menos de 5% da base), que representa nichos de compra mais específicos.

### Exemplos de perfis identificados

- **Cluster 2 (129 clientes):** cesta dominada por insumos básicos de cozinha e panificação, como creme culinário (linha food 1 kg), óleo de soja, gordura de palma, margarina (linha food 15 kg), açúcar cristal, sal refinado, trigo e filé de peito.
- **Cluster 6 (29 clientes):** cesta ampla e diversificada, com alta recorrência de molhos (mostarda, molhos orientais, ketchup), arroz, feijão (preto e carioca), farinha panko, leite de coco, vinagre, além de sal, açúcar e óleo. É o menor grupo e tem cara de compra mais abrangente.

> Os demais perfis podem ser explorados com a mesma consulta de ranking por cluster presente no notebook.

##  Aplicações comerciais

| Ação | Como a segmentação ajuda |
|---|---|
| **Campanhas direcionadas** | Comunicação e ofertas alinhadas à cesta típica de cada cluster |
| **Venda cruzada** | Oferecer a clientes de um cluster os itens que os clientes "mais completos" já compram |
| **Ampliação de mix** | Priorizar clientes que compram poucas subcategorias (mediana de 9 em 514) |
| **Priorização da carteira** | Direcionar esforço da equipe comercial por tamanho e perfil de segmento |
| **Gestão de estoque e compras** | Antecipar demanda dos itens-âncora dos clusters maiores |
| **Retenção** | Monitorar migração de clientes entre clusters e identificar queda de mix |

##  Limitações e próximos passos

**Limitações atuais**

- O período analisado é de apenas **3 meses**, o que pode não capturar sazonalidade.
- A segmentação usa **presença/ausência** de subcategorias. Ela não considera valor (R$), volume (`PESOLIQ`) ou frequência de compra.
- O K-means e o método do cotovelo foram aplicados sobre os dados reduzidos pelo **PCA de 2 componentes**, o que perde parte da informação original. A silhueta de 0,38 também foi calculada nesse espaço reduzido.
- O dendrograma foi construído com distância de Jaccard e ligação de Ward. O Ward pressupõe distâncias euclidianas, o que gera um aviso do SciPy. Ligação `average` ou `complete` é mais coerente com Jaccard.
- A escolha de K = 13 combina leitura visual e julgamento de negócio, não um critério único.

**Próximos passos**

- [ ] Incluir variáveis **RFM** (recência, frequência e valor monetário) e `PESOLIQ`.
- [ ] Rodar o K-means (ou K-medoids) nos vetores originais ou em mais componentes do PCA.
- [ ] Comparar K via silhueta, Davies-Bouldin e estabilidade entre execuções (`random_state`).
- [ ] Testar ligação `average` com Jaccard e outros algoritmos (DBSCAN, GMM).
- [ ] Nomear cada cluster com uma persona de negócio e estimar o faturamento por segmento.
- [ ] Construir um dashboard (Power BI) com os segmentos para uso do time comercial.
- [ ] Estender o histórico para 12 meses e acompanhar a migração entre clusters.

##  Como reproduzir

1. Clone o repositório:
   ```bash
   git clone https://github.com/<seu-usuario>/<nome-do-repositorio>.git
   cd <nome-do-repositorio>
   ```
2. Instale as dependências:
   ```bash
   pip install -r requirements.txt
   ```
3. Coloque o arquivo `BASE_FINAL.xlsx` na pasta `data/` (ou ajuste o caminho no notebook; no original ele é lido do Google Drive, em `/content/drive/MyDrive/`).
4. Abra o notebook `Seg_Clientes_BigData.ipynb` no Jupyter ou no Google Colab e execute as células em ordem.

**Dependências principais**

```text
pandas
openpyxl
scikit-learn
scipy
matplotlib
plotly
```

##  Estrutura do repositório

```text
.
├── README.md
├── requirements.txt
├── notebooks/
│   └── Seg_Clientes_BigData.ipynb
├── data/                  # base não versionada (.gitignore)
└── docs/
    └── Seg_Clientes_BigData.pdf
```

# 👤 Sobre mim 
**Samyra Bentemmuller**

Analista de Dados / BI Specialist

LinkedIn: https://www.linkedin.com/in/samyra-bentemmuller

GitHub: https://github.com/samyraLB
