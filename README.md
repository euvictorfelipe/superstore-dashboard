# Dashboard de Análise de Vendas - Superstore

Dashboard interativo desenvolvido em Power BI para análise de vendas, identificação de produtos e regiões mais lucrativos, e monitoramento de performance mês a mês.

## 📊 Visão Geral

Este projeto analisa dados de vendas de uma loja fictícia (Superstore) com foco em:
- Identificação de produtos e categorias de maior receita
- Análise de lucratividade por região geográfica
- Monitoramento de performance temporal (crescimento MoM)
- Ranqueamento de top performers (estados, clientes, produtos)

## 🛠️ Stack Técnica

- **Power BI Desktop** - Desenvolvimento do dashboard e modelagem de dados
- **DAX** - Criação de medidas calculadas e time intelligence
- **Power Query (M)** - ETL e transformação de dados
- **Azure Maps** - Visualização geográfica de vendas

## 📁 Estrutura do Projeto

```
superstore-dashboard/
├── analise_vendas_superstore.pbix    # Arquivo Power BI
├── Sample - Superstore.csv           # Dataset original
└── README.md                         # Documentação
```

## 🔄 Processo de ETL

### Extração
Importação de arquivo CSV com configuração específica de encoding:

```m
Csv.Document(
    File.Contents("Sample - Superstore.csv"),
    [Delimiter=",", Columns=21, Encoding=1252, QuoteStyle=QuoteStyle.None]
)
```

**Problema resolvido:** O encoding padrão (UTF-8) não conseguia interpretar corretamente os dados. A solução foi especificar `Encoding=1252` (Windows-1252) durante a importação, garantindo leitura correta de todos os caracteres.

### Transformação
- **Correção de tipos de dados** - Ajuste de colunas numéricas e de data
- **Criação de tabela dCalendario** - Tabela de dimensão de datas para suportar funções de time intelligence

### Modelagem
- Tabela fato: `Sample - Superstore` (dados transacionais)
- Tabela dimensão: `dCalendario` (datas)
- Relacionamento: `Sample - Superstore[Order Date]` → `dCalendario[Data]`

## 📐 Medidas DAX Criadas

### Medidas Base
```dax
Receita Total = SUM('Sample - Superstore'[Sales])

Lucro Total = SUM('Sample - Superstore'[Profit])

Margem de Lucro = ([Lucro Total] / [Receita Total])

Qtd Pedidos = DISTINCTCOUNT('Sample - Superstore'[Order ID])

Ticket Medio = [Receita Total] / [Qtd Pedidos]
```

### Time Intelligence - Mês Anterior
```dax
Faturamento Mes Anterior = 
VAR _valor =
    CALCULATE(
        [Receita Total],
        DATEADD('dCalendario'[Data], -1, MONTH)
    )
RETURN
IF(ISBLANK(_valor), BLANK(), _valor)

Lucro total Mes Anterior = 
VAR _valor =
    CALCULATE(
        [Lucro Total],
        DATEADD('dCalendario'[Data], -1, MONTH)
    )
RETURN
IF(ISBLANK(_valor), BLANK(), _valor)

Qtd Pedidos Mes Anterior = 
CALCULATE(
    [Qtd Pedidos],
    DATEADD(dCalendario[Data], -1, MONTH)
)

Ticket Medio Mes Anterior = 
CALCULATE(
    [Ticket Medio],
    DATEADD(dCalendario[Data], -1, MONTH)
)

Qtd Clientes Mes Anterior = 
CALCULATE(
    DISTINCTCOUNT('Sample - Superstore'[Customer ID]),
    DATEADD(dCalendario[Data], -1, MONTH)
)
```

### Variação MoM (Month-over-Month)
```dax
Faturamento MoM % = 
DIVIDE(
    [Receita Total] - [Faturamento Mes Anterior],
    [Faturamento Mes Anterior]
)

Lucro MoM % = 
DIVIDE(
    [Lucro Total] - [Lucro total Mes Anterior],
    [Lucro total Mes Anterior]
)
```

## 📊 Funcionalidades do Dashboard

### KPIs Principais
- **Receita Total** - $2.30M
- **Lucro Total** - $286.40M
- **Qtd Pedidos** - 5009
- **Margem de Lucro** - 12.47%
- **Qtd Clientes** - 793
- **Ticket Médio** - $458.61

### Visualizações Implementadas

1. **Gráfico de Pizza** - Distribuição de receita por categoria
2. **Gráfico de Linha Dupla** - Evolução mensal de vendas vs quantidade de pedidos (2014-2017)
3. **Gráfico de Barras Horizontais** (3x):
   - Top 10 Estados por Receita
   - Top 10 Clientes por Receita
   - Top 10 Produtos por Receita
4. **Mapa Azure** - Distribuição geográfica de vendas
5. **Cartões MoM** - Indicadores de variação mensal (Receita, Lucro, Pedidos, Ticket)

### Filtros Interativos (Slicers)
- Data (período)
- Cliente
- Produto
- Estado
- Categoria
- Segmento

## 🎯 Principais Insights

1. **Categoria Technology domina receita** - Representa a maior fatia do faturamento total
2. **Crescimento consistente 2014-2017** - Tendência positiva em receita e lucro
3. **Top Cliente: Sean Miller** - $25.04k em receita
4. **Top Produto: Canon imageClass** - $61.60k em receita
5. **Margem de lucro estável** - 12.47% de rentabilidade média

## 🧠 Aprendizados Técnicos

### 1. Time Intelligence com DATEADD
Implementação de comparações mês-a-mês usando tabela de calendário customizada, essencial para análises temporais precisas.

### 2. Tratamento de Valores BLANK()
Uso de `IF(ISBLANK())` nas medidas de mês anterior para evitar erros quando não há dados do período comparativo.

### 3. Pattern VAR-RETURN no DAX
Organização de medidas complexas usando variáveis para melhor legibilidade e performance.

### 4. Encoding em Importação CSV
Resolução de problemas de caracteres especiais usando encoding Windows-1252 (CP-1252) ao invés do UTF-8 padrão.

### 5. DISTINCTCOUNT vs COUNT
Escolha de `DISTINCTCOUNT` para contagem de pedidos, evitando duplicatas em casos de múltiplas linhas por pedido.

## 🚀 Como Usar

1. Baixar o arquivo `analise_vendas_superstore.pbix`
2. Abrir no Power BI Desktop
3. Atualizar caminho do arquivo CSV em Power Query, se necessário:
   - Ir em **Transformar Dados** > **Configurações de Fonte de Dados**
   - Atualizar o caminho para o arquivo `Sample - Superstore.csv`
4. Interagir com filtros e visuais para explorar os dados

## 📈 Possíveis Extensões

- [ ] Análise de sazonalidade por categoria
- [ ] Previsão de vendas usando forecasting
- [ ] Análise de churn de clientes
- [ ] Drill-through para detalhamento de pedidos específicos
- [ ] Segmentação RFM (Recency, Frequency, Monetary)

## 👤 Autor

**Victor**  
Análise de Dados | Python · SQL · Power BI  
[LinkedIn](https://www.linkedin.com/in/victor-martins1/) · [GitHub](https://github.com/euvictorfelipe)

---

**Tecnologias:** `Power BI` `DAX` `Power Query` `ETL` `Data Visualization` `Business Intelligence`
