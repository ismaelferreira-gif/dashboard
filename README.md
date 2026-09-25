# Projeto Abastecimento SP — Dashboard Power BI

## Objetivo

Mostrar **tudo o que entra e sai das unidades de saúde de São Paulo**, ou seja, todo o abastecimento distribuído dentro da rede como **receita interna**:

- **Materiais hospitalares**
- **Medicamentos**
- **Itens de laboratório**
- Demais materiais distribuídos às unidades

O painel acompanha o caminho completo desses itens: pedido, programação e **faturamento** das entregas, recebimento de notas, movimentação (entradas, saídas e perdas), estoque por unidade e cobertura em dias. Assim dá para ver onde há falta, excesso ou risco de desabastecimento.

Painel de acompanhamento do abastecimento de itens (SUPRI) nas unidades de saúde: estoque, cobertura em dias, entradas, saídas, perdas, recebimentos e entregas.

O projeto está salvo no formato **Power BI Project (.pbip)**, que guarda o relatório e o modelo como arquivos de texto. Assim o GitHub mostra exatamente o que mudou a cada versão.

## Como baixar e abrir

O repositório tem duas pastas e uma Release:

| Onde | Conteúdo | Como baixar |
|---|---|---|
| [`dashboard/`](dashboard) | `PROJETO ABASTECIMENTO SP.zip`, com o dashboard completo (Power BI Project) | Clique no arquivo e depois no botão de download (↓) |
| [`dados/`](dados) | Todos os arquivos que o Power Query lê, menos a CONSOLIDADOS BI | Abra cada arquivo e clique em download (↓), ou baixe tudo pelo botão verde **Code → Download ZIP** |
| [Release **dados-v1**](../../releases) | Os 6 arquivos da **CONSOLIDADOS BI** (2026-01 a 2026-06). Cada um tem mais de 25 MB, acima do limite de envio pelo site | Abra a Release e baixe os arquivos em **Assets** |

Passo a passo:

1. Baixe tudo pelo botão verde **Code → Download ZIP** e extraia no computador.
2. Baixe os 6 arquivos da Release e coloque-os em `dados/CONSOLIDADOS BI/`. Crie essa pasta.
3. Em `dados/GSS DO DIA/`, extraia o `GSS DO DIA.zip` ali mesmo e **apague o .zip**, porque a consulta lê todos os arquivos dessa pasta.
4. Extraia `dashboard/PROJETO ABASTECIMENTO SP.zip` e abra `PROJETO ABASTECIMENTO SP.pbip` no Power BI Desktop.
5. Em **Transformar dados → Gerenciar parâmetros**, troque **`CaminhoBase`** pelo caminho da pasta `dados`. Exemplo: `C:\Users\SeuNome\Downloads\dashboard-main\dados`.
6. Clique em **Atualizar**.

## Pasta `dados/`

Contém os arquivos lidos pelo Power Query, na mesma estrutura de pastas que as consultas esperam:

```
dados/
├── CATEGORIA/CATEGORIA SUPRIS.xlsx
├── CONSOLIDADOS BI/          ← baixar os 6 arquivos da Release dados-v1
├── ENTREGAS/20260731101347550.xlsx
├── ERROS/ERROS.xlsx
├── ESTOQUE POR LOCAL/BASICA/01.xlsx
├── ESTOQUE RECEBIMENTO/2020.xlsx ... 2026.xlsx
└── GSS DO DIA/GSS DO DIA.zip   ← CSV de 114 MB compactado (o limite do GitHub é 100 MB)
```

---

## De onde vêm os dados

Todas as fontes são arquivos locais (Excel e CSV), lidos a partir de uma pasta raiz definida pelo parâmetro:

```
CaminhoBase = "C:\Users\kauan\OneDrive\Desktop"
```

Para rodar em outro computador, basta alterar **só** esse parâmetro.

### Fontes externas (arquivos)

| Tabela no modelo | Caminho (a partir de `CaminhoBase`) | Tipo | Aba / formato | Observações |
|---|---|---|---|---|
| **CONSOLIDADOS BI** | `\CONSOLIDADOS BI\` (pasta) | Excel | aba `01` | Combina apenas os arquivos `2026-01.xlsx` a `2026-06.xlsx`. Ao adicionar um mês novo, inclua o nome na lista do passo `Fonte`. |
| **GSS DO DIA** | `\GSS DO DIA\` (pasta) | CSV (`;`, Windows-1252) | 12 colunas | Combina todos os arquivos da pasta. Exclui SMS, Material de Escritório, Remédio em Casa e Terminais SPTrans. Classifica em *Rede Hospitalar* × *Atenção Básica*. |
| **PAINEL GSS** | `\GSS DO DIA\` (pasta) | CSV | — | Mesma origem do GSS DO DIA, cruzada com ESTOQUE BASICA RESUMO. Calcula **cobertura em dias** = Estoque ÷ CMM × 30, **quantidade necessária** e **status KPI** (🔴 Zerado, 🟡 Sem CMM, 🔵 < 15 dias, 🟣 > 90 dias, 🟢 Adequado). |
| **ESTOQUE RECEBIMENTO** | `\ESTOQUE RECEBIMENTO\` (pasta) | Excel | aba `Relatorio` | Combina todos os arquivos da pasta. Exclui fornecedores *REDE BASICA* e *REDE HOSPITALAR*. |
| **ESTOQUE BASICA** | `\ESTOQUE POR LOCAL\BASICA\01.xlsx` | Excel | aba `Relatorio` | Estoque por local, lote e validade. |
| **ESTOQUE BASICA RESUMO** | `\ESTOQUE POR LOCAL\BASICA\01.xlsx` | Excel | aba `Relatorio` | Mesmo arquivo, agrupado por código SUPRI (soma de QT DISP). |
| **ENTREGAS** | `\ENTREGAS\20260731101347550.xlsx` | Excel | aba `Plan1` | Pedidos, programação e faturamento por unidade de saúde. |
| **ENTREGAS_ULTIMO_PEDIDO** | `\ENTREGAS\20260731101347550.xlsx` | Excel | aba `Plan1` | Mesmo arquivo, mantendo só o pedido mais recente por unidade + item. |
| **ERROS** | `\ERROS\ERROS.xlsx` | Excel | aba `Planilha1` | Movimentos de erro por código SUPRI e mês. |
| **CATEGORIA** | `\CATEGORIA\CATEGORIA SUPRIS.xlsx` | Excel | aba `Export` | Categoria de cada código SUPRI. |

> ⚠️ **ENTREGAS** aponta para um arquivo com nome fixo (`20260731101347550.xlsx`). Se chegar um arquivo novo com outro nome, é preciso atualizar o caminho na consulta.

### Tabelas derivadas (calculadas dentro do Power BI)

| Tabela | Origem | O que faz |
|---|---|---|
| **PRECO_VIGENTE** | ESTOQUE RECEBIMENTO | Menor preço unitário por item e mês. |
| **SUPRI_MES** | PRECO_VIGENTE + CALENDARIO_MENSAL | Grade item × mês com o preço vigente. |
| **CALENDARIO_MENSAL** | Gerada em Power Query | Um registro por mês, de jan/2023 até o mês atual. |
| **Calendario** | DAX, a partir de CONSOLIDADOS BI | Calendário diário com ano, mês, trimestre, semestre e dia da semana. |
| **DIM_SUPRI** | DAX: códigos distintos de CONSOLIDADOS BI | Dimensão de itens (tabela central do modelo). |
| **DIM_UNIDADES** | DAX: estabelecimentos distintos de GSS DO DIA | Dimensão de unidades. |
| **Medidas** / **html** | — | Tabelas vazias que só guardam medidas. |

### Fluxo dos dados

```
Planilhas / CSV (OneDrive\Desktop)
        │
        ▼
 Power Query  ──►  CONSOLIDADOS BI, GSS DO DIA, ESTOQUE RECEBIMENTO,
                   ESTOQUE BASICA, ENTREGAS, ERROS, CATEGORIA
        │
        ├──►  PAINEL GSS  (GSS DO DIA + ESTOQUE BASICA RESUMO)
        ├──►  PRECO_VIGENTE ──► SUPRI_MES
        │
        ▼
 Modelo (relacionamentos via DIM_SUPRI e Calendario)
        │
        ▼
 Relatório (páginas abaixo)
```

### Principais relacionamentos

- **DIM_SUPRI[Cod. SUPRI]** liga CONSOLIDADOS BI, GSS DO DIA, PAINEL GSS, ESTOQUE RECEBIMENTO, ESTOQUE BASICA, ENTREGAS, ENTREGAS_ULTIMO_PEDIDO, PRECO_VIGENTE, SUPRI_MES e CATEGORIA.
- **Calendario[Date]** liga CONSOLIDADOS BI[Anomes], SUPRI_MES[ANOMES], ESTOQUE RECEBIMENTO[DT EMISSAO] e as datas de pedido, programação e faturamento de ENTREGAS.
- **GSS DO DIA[ESTABELECIMENTO]** → **CONSOLIDADOS BI[Estabelecimento]**.

---

## Páginas do relatório

`PAGINA INICIAL` (menu de navegação) → `SAIDAS`, `SAIDAS DETALHADA`, `ENTRADAS`, `ENTRADAS DETALHADA`, `DESUSO`, `DESUSO DETALHADA`, `PAINEL`, `PAINEL 01` a `PAINEL 05`, `ESTOQUE`, `RECEBIMENTOS`, `ENTREGAS`.

## Principais medidas

- **Movimentação:** Entradas Líquidas, Saídas Líquidas, Perdas Líquidas (quantidade, valor, % por tipo e médias mensais), % Variação Entradas Líquidas
- **Recebimento:** Valor Total Recebido, Média Mensal Recebida, Preço Unitário
- **Estoque e cobertura:** CMM, Estoque BASICA, Cobertura Atenção Básica, Validades Próximas 90 Dias BASICA
- **KPIs do painel:** Itens Cobertura Crítica 15, Itens Cobertura Excesso 90, Itens Estoque Adequado, Itens Sem CMM
- **Visuais HTML:** cards e tabelas de KPIs por página (`Page_*_KPIs_HTML`, `Tabela_Sugestao_Inteligente_90Dias_HTML`)

## Estrutura do repositório

```
dados/                                  ← arquivos de origem do Power Query
dashboard/
  └─ PROJETO ABASTECIMENTO SP.zip       ← dashboard completo; dentro dele:
       ├─ PROJETO ABASTECIMENTO SP.pbip           ← abra este arquivo
       ├─ PROJETO ABASTECIMENTO SP.Report/        ← páginas, visuais, imagens e tema
       └─ PROJETO ABASTECIMENTO SP.SemanticModel/
            └─ definition/
                 ├─ expressions.tmdl               ← parâmetro CaminhoBase
                 ├─ relationships.tmdl
                 └─ tables/*.tmdl                  ← consultas M e medidas DAX
```
