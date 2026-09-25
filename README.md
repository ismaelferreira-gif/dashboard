# Projeto Abastecimento SP — Dashboard Power BI

Painel de acompanhamento do abastecimento de itens (SUPRI) nas unidades de saúde: estoque, cobertura em dias, entradas, saídas, perdas, recebimentos e entregas.

O projeto está salvo no formato **Power BI Project (.pbip)**, que guarda o relatório e o modelo como arquivos de texto. Assim o GitHub mostra exatamente o que mudou a cada versão.

## Como abrir

1. Baixe/clone este repositório.
2. Abra `PROJETO ABASTECIMENTO SP.pbip` no Power BI Desktop.
3. Em **Transformar dados → Gerenciar parâmetros**, ajuste o parâmetro **`CaminhoBase`** para a pasta onde estão as planilhas no seu computador.
4. Clique em **Atualizar**.

> Os dados em si **não** ficam no repositório (o `.gitignore` exclui o `cache.abf`). Cada pessoa precisa ter as pastas de dados localmente.

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
PROJETO ABASTECIMENTO SP.pbip              ← abra este arquivo
PROJETO ABASTECIMENTO SP.Report/           ← páginas, visuais, imagens e tema
PROJETO ABASTECIMENTO SP.SemanticModel/
  └─ definition/
       ├─ expressions.tmdl                  ← parâmetro CaminhoBase e funções de pasta
       ├─ relationships.tmdl
       └─ tables/*.tmdl                     ← uma tabela por arquivo (consultas M + medidas DAX)
```
