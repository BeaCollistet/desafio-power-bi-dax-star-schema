# Desafio DIO: Modelagem e Transformação de Dados com DAX no Power BI

Projeto prático do desafio da DIO. A partir da tabela única **Financial Sample** (`financials`), criei as tabelas **dimensão** e **fato** de um modelo em **estrela (star schema)** no Power BI, usando Power Query e DAX.

## 🗺️ Esquema em estrela

![Esquema em estrela](esquema-estrela.png)

A tabela fato `F_Vendas` fica no centro e se liga às dimensões. A `financials_origem` fica oculta e sem relacionamentos (é só o backup dos dados originais).

## 🧱 Tabelas criadas

| Tabela | Tipo | O que guarda |
|---|---|---|
| `financials_origem` | Backup (oculta) | Tabela original, com os índices `SK_ID` e `ID_Produto` |
| `F_Vendas` | Fato | `SK_ID`, `ID_Produto`, `Date`, `Product`, `Units Sold`, `Sale Price`, `Discount Band`, `Segment`, `Country`, `Sales`, `Profit` |
| `D_Produtos` | Dimensão | 1 linha por produto, com média de unidades vendidas, média, mediana, máximo e mínimo do valor de venda |
| `D_Produtos_Detalhes` | Dimensão | `ID_Produto`, `Discount Band`, `Sale Price`, `Units Sold`, `Manufacturing Price` |
| `D_Descontos` | Dimensão | `ID_Produto`, `Discounts`, `Discount Band` |
| `D_Detalhes` | Dimensão | O que sobrou das outras tabelas: `SK_ID`, `Gross Sales`, `COGS`, `Month Number`, `Month Name`, `Year` |
| `D_Calendario` | Dimensão (DAX) | Uma linha por dia, com ano, mês e trimestre |

## 🔧 Processo de construção

### 1. Power Query (transformação)

1. **Importei** a tabela `financials` e a renomeei para `financials_origem` (backup).
2. **Coluna de índice:** criei `SK_ID` (de 0 a 699), um número único para cada venda.
3. **Coluna condicional:** criei `ID_Produto`, que dá um número a cada produto (Carretera = 0, Montana = 1, Paseo = 2, Velo = 3, VTT = 4, Amarilla = 5).
4. **Duplicar e agrupar:** duplicando a origem, criei `D_Produtos` com **Agrupar por** produto, calculando média, mediana, máximo e mínimo.
5. **Escolher colunas:** a partir de cópias da origem, criei `D_Produtos_Detalhes`, `D_Descontos`, `D_Detalhes` e `F_Vendas`, mantendo só as colunas de cada uma.
6. **Reorganizei as colunas**, deixando as chaves (`ID_Produto`, `SK_ID`) no começo.

### 2. DAX (calendário)

Criei a tabela `D_Calendario` com **Nova tabela**:

```
D_Calendario = CALENDAR(
    DATE(YEAR(MIN(F_Vendas[Date])), 1, 1),
    DATE(YEAR(MAX(F_Vendas[Date])), 12, 31)
)
```

E adicionei colunas com **Nova coluna**:

```
Ano         = YEAR(D_Calendario[Date])
Mes_Numero  = MONTH(D_Calendario[Date])
Mes_Nome    = FORMAT(D_Calendario[Date], "mmmm")
Trimestre   = "T" & QUARTER(D_Calendario[Date])
```

### 3. Modelo (relacionamentos)

| De | Para | Cardinalidade |
|---|---|---|
| `D_Produtos (ID_Produto)` | `F_Vendas (ID_Produto)` | 1 : muitos |
| `D_Produtos (ID_Produto)` | `D_Produtos_Detalhes (ID_Produto)` | 1 : muitos |
| `D_Produtos (ID_Produto)` | `D_Descontos (ID_Produto)` | 1 : muitos |
| `D_Calendario (Date)` | `F_Vendas (Date)` | 1 : muitos |
| `D_Detalhes (SK_ID)` | `F_Vendas (SK_ID)` | 1 : 1 |

A `financials_origem` ficou **sem relacionamentos** e **oculta**. O Power BI criou relacionamentos automáticos com ela, e eu os excluí, porque ela é só o backup e ligações extras deixavam a relação principal da fato inativa.

## 🧠 Funções DAX utilizadas

| Função | Para que usei |
|---|---|
| `CALENDAR` | Gerar uma linha para cada data entre dois limites |
| `DATE` | Montar 1º de janeiro e 31 de dezembro dos anos de início e fim |
| `MIN` / `MAX` | Achar a primeira e a última data de venda |
| `YEAR` | Extrair o ano |
| `MONTH` | Extrair o número do mês |
| `FORMAT` | Escrever o nome do mês |
| `QUARTER` | Descobrir o trimestre |

## 📝 Observações

- A **estrela tem um pequeno "galho"**: `D_Produtos_Detalhes` e `D_Descontos` se ligam à `D_Produtos` (pelo `ID_Produto`), e não direto à fato. Mantive assim porque é como o enunciado define as tabelas.
- `D_Detalhes` guarda `Month Number`, `Month Name` e `Year`, que também existem no `D_Calendario`. Mantive porque o enunciado pede para guardar as informações que não couberam nas outras tabelas. Em uma versão futura, essas colunas poderiam sair.

## 📁 Conteúdo do repositório

- `desafio-5-power-bi.pbix`: projeto do Power BI
- `esquema-estrela.png`: imagem do modelo em estrela
- `README.md`: este arquivo

## ▶️ Como abrir

1. Instale o **Power BI Desktop** (gratuito, só para Windows).
2. Baixe o arquivo `desafio-5-power-bi.pbix`.
3. Abra no Power BI Desktop. As transformações estão em **Transformar dados**, e o modelo em **Exibição de modelo**.

## 🧠 O que aprendi

[Escreva 2 ou 3 frases com suas palavras. Por exemplo: como separar uma tabela única em fato e dimensões, como criar um calendário com DAX e como conferir os relacionamentos.]

## 🔗 Referências

- Repositório do curso: https://github.com/julianazanelatto/power_bi_analyst
