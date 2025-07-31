# Meta Ads Performance | Ganho de Seguidores  📊

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)

## Visão Geral

![Dashboard Meta Analytics](https://github.com/LRLeite/Dashboards/blob/main/Meta%20Ads%20Analytics/assets/imagens/dashboard.png)

Este projeto apresenta uma solução de análise de dados desenvolvida em Power BI, focada em monitorar e otimizar campanhas de tráfego pago com o objetivo principal de aumentar a base de seguidores em plataformas como o Instagram. A solução visa transformar dados brutos em insights acionáveis, permitindo que as decisões de marketing sejam mais precisas e alinhadas aos resultados de negócio.

## Problema Abordado
No cenário do marketing digital, campanhas de aquisição de seguidores frequentemente geram grandes volumes de dados. No entanto, a análise fragmentada ou o foco em métricas isoladas podem mascarar ineficiências e comprometer a escalabilidade. A dificuldade reside em conectar o investimento realizado com o impacto real no crescimento da base de seguidores, compreendendo o "porquê" por trás dos resultados, e não apenas o "o quê".

## Solução Proposta: Dashboard de Performance de Campanhas
Para superar esses desafios, foi desenvolvido um dashboard interativo em Power BI que atua como um sistema coeso de métricas. Este painel de controle intuitivo consolida informações de diversas fontes, proporcionando uma visão abrangente e integrada do desempenho da campanha.

### Principais Características e Funcionalidades:
* Análise Orientada a Objetivos: O dashboard utiliza métricas intrinsicamente alinhadas aos objetivos da campanha.

* Visão Sistêmica de KPIs: Conecta métricas de diferentes estágios do funil de aquisição (investimento, alcance, cliques, CTR, CPV, taxa de conversão, CPNS, entre outros), permitindo uma leitura integrada que transforma números soltos em decisões estratégicas.

* Granularidade Temporal e Hierárquica: Permite analisar o desempenho em diversas granularidades de tempo (do ano ao dia) e por diferentes níveis hierárquicos da campanha (campanha, conjunto de anúncios, anúncio individual).

* Visualizações Dinâmicas: Utiliza recursos interativos (filtros, slicers) para explorar dados e identificar picos de desempenho, tendências e oportunidades de otimização.

* Insights: Vai além da descrição dos resultados, revelando o "porquê" por trás dos números e guiando a otimização contínua dos investimentos para maximizar o retorno.

## Arquitetura do Modelo de Dados
![Visão geral do modelo de dados](https://github.com/LRLeite/Dashboards/blob/main/Meta%20Ads%20Analytics/assets/imagens/modelo-dados.png)

<br>

## 🔄 Processo ETL - Power Query

### Conexão com Dados
O dashboard conecta-se a um banco PostgreSQL hospedado no Supabase:

```powerquery
// Conecta-se ao banco de dados PostgreSQL
Fonte = PostgreSQL.Database("aws-0-sa-east-1.pooler.supabase.com", "postgres")

// Seleciona a tabela 'meta_ads_instagram' do esquema 'public'
TabelaMetaAdsInstagram = Fonte{[Schema="public",Item="meta_ads_instagram"]}[Data]
```

### Transformações

#### 1. Limpeza de Dados
```powerquery
// Remove colunas desnecessárias para a análise
#"Colunas Removidas" = Table.RemoveColumns(TabelaMetaAdsInstagram,{
    "Frequência", "CPC (R$)", "CPM (R$)", "Conversões", "CPA (R$)", 
    "ROAS", "Curtidas", "Comentários", "Compartilhamentos", "Salvamentos", 
    "Cliques no perfil", "Mensagens iniciadas", "Visualizações da página de destino", 
    "Tempo médio visualização vídeo (s)", "ROI"
})
```

#### 2. Filtros Dinâmicos
```powerquery
// Implementa filtro dinâmico por período usando parâmetros
RangeStart_dt = RangeStart,
RangeEnd_Ajustado_dt = DateTime.From(Date.From(RangeEnd)) + #duration(0, 23, 59, 59),

#"Linhas Filtradas por Data" = Table.SelectRows(#"Data Como DateTime", 
    each [Data] >= RangeStart_dt and [Data] <= RangeEnd_Ajustado_dt)
```

#### 3. Filtros de Negócio
```powerquery
// Filtra campanhas específicas conforme regras de negócio
#"Linhas Filtradas de Campanhas" = Table.SelectRows(#"Linhas Filtradas por Data", 
    each ([Campanha] <> "Campanha_19" and [Campanha] <> "Campanha_27"))
```

#### 4. Otimização de Tipos
```powerquery
// Otimiza tipos de dados para melhor performance
#"Tipos Alterados Numericos e Inteiros" = Table.TransformColumnTypes(#"Colunas Renomeadas", {
    {"CTR", type number}, 
    {"Investimento (R$)", type number},
    {"Cliques no link", Int64.Type}, 
    {"Impressões", Int64.Type},
    {"Alcance", Int64.Type}, 
    {"Nº Seguidores", Int64.Type},
    {"Nº Seguidores Dia", Int64.Type}, 
    {"Visitas ao perfil", Int64.Type}
}, "en-US")
```

### Criação da Dimensão Temporal
```dax
Calendario = 
ADDCOLUMNS(
    CALENDAR(MIN('MetaAds Analytics'[Data]), MAX('MetaAds Analytics'[Data])),
    "Ano", YEAR([Date]),
    "Mês", MONTH([Date]),
    "Mês (MMM)", FORMAT([Date], "MMM"),
    "Semana do Mês", WEEKNUM([Date]) - WEEKNUM(DATE(YEAR([Date]), MONTH([Date]), 1)) + 1,
    "Semana do Ano", WEEKNUM([Date]),
    "Dia", DAY([Date])
)
```

<br>

### Tabelas do Modelo

#### 📈 MetaAds Analytics (Tabela Fato)
Tabela principal contendo todas as métricas de performance das campanhas.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `Id` | Text | Identificador único do registro |
| `Data` | Date | Data da métrica |
| `Campanha` | Text | Nome da campanha publicitária |
| `Conjunto de Anúncios` | Text | Agrupamento de anúncios |
| `Anúncio` | Text | Anúncio específico |
| `Plataforma` | Text | Plataforma de veiculação |
| `Impressões` | Integer | Número de impressões |
| `Alcance` | Integer | Alcance único |
| `Cliques no link` | Integer | Cliques em links |
| `Investimento (R$)` | Decimal | Valor investido |
| `Visitas ao perfil` | Integer | Visitas ao perfil |
| `Nº Seguidores` | Integer | Total de seguidores |
| `Nº Seguidores Dia` | Integer | Novos seguidores no dia |
| `CTR` | Decimal | Taxa de cliques |

<br>

#### 📅 Calendario (Dimensão Temporal)
Tabela de dimensão criada via DAX para análise temporal granular.

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `Date` | Date | Data principal |
| `Ano` | Integer | Ano |
| `Mês` | Integer | Mês numérico |
| `Mês (MMM)` | Text | Mês abreviado |
| `Semana do Mês` | Integer | Semana dentro do mês |
| `Semana do Ano` | Integer | Semana do ano |
| `Dia` | Integer | Dia do mês |

<br>

#### 🎛️ Slicer Medidas
Tabela auxiliar para seleção dinâmica de métricas.

| Campo | Valor | Ordem |
|-------|-------|-------|
| Investimento | Total Investimento | 0 |
| Alcance | Alcance | 1 |
| Impressões | Impressões | 2 |
| Cliques no link | Cliques no link | 3 |
| CTR | CTR | 4 |
| Visitas Perfil | Visitas Perfil | 5 |
| CPV | CPV | 6 |
| Novos Seguidores | Total Novos Seguidores | 7 |
| CPNS | CPNS | 8 |

<br>

#### 👥 Slicer Seguidores
Tabela auxiliar para análise de seguidores.

| Campo | Valor | Ordem |
|-------|-------|-------|
| N° Seguidores | N° Seguidores Dia | 0 |
| Visitas Perfil | Visitas Perfil | 1 |

<br>

## 📊 Medidas DAX

<br>

### 💰 Métricas de Investimento

#### **`Total Investimento`**
> Custo total da campanha.

```dax
Total Investimento = SUM('MetaAds Analytics'[Investimento (R$)])
```


#### **`CPV (Custo Por Visita ao Perfil)`**
> Calcula o custo por visita ao perfil, métrica fundamental para avaliar eficiência.

```dax
CPV = DIVIDE([Total Investimento],[Visitas Perfil],0)
```


#### **`CPNS (Custo Por Novo Seguidor)`**
> Eficiência de custo na aquisição de cada novo seguidor, indicador de ROI direto para o objetivo.

```dax
CPNS = DIVIDE([Total Investimento],[Total Novos Seguidores],0)
```

<br>

### 📈 Métricas de Performance

#### **`Alcance`**
> Mede o número total de contas únicas que visualizaram a publicação.

```dax
Alcance = 
SUM('MetaAds Analytics'[Alcance])
```

#### **`Impressões`**
> Mede o número total de vezes que o seu conteúdo foi exibido, independentemente de ter sido clicado ou visualizado.

```dax
Impressões = 
SUM('MetaAds Analytics'[Impressões])
```

#### **`Cliques no link`**
> Monitoram o engajamento inicial com o anúncio.

```dax
Cliques no link = 
SUM('MetaAds Analytics'[Cliques no link])
```

#### **`CTR`**
> Avalia a atratividade e relevância dos anúncios.

```dax
CTR = AVERAGE('MetaAds Analytics'[CTR])
```

#### **`Taxa de Conversão (Visita -> Seguidor)`**
> Percentual de visitas ao perfil convertidas em novos seguidores.

```dax
Taxa Conversão = DIVIDE([N° Seguidores Dia], [Visitas Perfil],0)
```

#### **`Total de Novos Seguidores`**
> O KPI final, medindo diretamente o resultado do objetivo da campanha.

```dax
Total Novos Seguidores = [Visitas Perfil] * [Taxa Conversão]
```

<br>

### 📊 Estatísticas Inferenciais

#### **`Correlação de Pearson`**
> Analisa a correlação linear entre visitas ao perfil e novos seguidores.

```dax
Correlação Pearson = 
VAR _DailyTable = 
    SUMMARIZE(
        'MetaAds Analytics',
        'MetaAds Analytics'[Data],
        "TotalVisitas", SUM('MetaAds Analytics'[Visitas ao perfil]),
        "TotalSeguidores", SUM('MetaAds Analytics'[Nº Seguidores Dia])
    )

VAR _Count = COUNTROWS(_DailyTable)
VAR _SumX = SUMX(_DailyTable, [TotalVisitas])
VAR _SumY = SUMX(_DailyTable, [TotalSeguidores])
VAR _SumXY = SUMX(_DailyTable, [TotalVisitas] * [TotalSeguidores])  
VAR _SumX2 = SUMX(_DailyTable, [TotalVisitas] * [TotalVisitas])
VAR _SumY2 = SUMX(_DailyTable, [TotalSeguidores] * [TotalSeguidores])

VAR _MeanX = DIVIDE(_SumX, _Count)
VAR _MeanY = DIVIDE(_SumY, _Count)

VAR _Numerator = _SumXY - (_Count * _MeanX * _MeanY)
VAR _DenominatorX = _SumX2 - (_Count * _MeanX * _MeanX)
VAR _DenominatorY = _SumY2 - (_Count * _MeanY * _MeanY)
VAR _Denominator = SQRT(_DenominatorX * _DenominatorY)

RETURN DIVIDE(_Numerator, _Denominator)
```

#### Interpretação das Correlações

| Valor de r | Interpretação |
|------------|---------------|
| 0,7 a 1,0 | Correlação forte |
| 0,3 a 0,7 | Correlação moderada |
| 0,1 a 0,3 | Correlação fraca |
| -0,1 a 0,1 | Correlação negligível |
| -0,3 a -0,1 | Correlação fraca negativa |
| -0,7 a -0,3 | Correlação moderada negativa |
| -1,0 a -0,7 | Correlação forte negativa |

**⚠️ IMPORTANTE: CORRELAÇÃO NÃO IMPLICA CAUSALIDADE**

<br>

#### **`Correlação de Spearman`**
> Análise de correlação não-paramétrica baseada em rankings.

```dax
Correlação Spearman = 
VAR _DailyTable = 
    ADDCOLUMNS(
        SUMMARIZE(
            'MetaAds Analytics',
            'MetaAds Analytics'[Data],
            "TotalVisitas", SUM('MetaAds Analytics'[Visitas ao perfil]),
            "TotalSeguidores", SUM('MetaAds Analytics'[Nº Seguidores Dia])
        ),
        "RankX", RANKX(
            SUMMARIZE('MetaAds Analytics', 'MetaAds Analytics'[Data], 
                "TotalVisitas", SUM('MetaAds Analytics'[Visitas ao perfil])),
            [TotalVisitas]
        ),
        "RankY", RANKX(
            SUMMARIZE('MetaAds Analytics', 'MetaAds Analytics'[Data], 
                "TotalSeguidores", SUM('MetaAds Analytics'[Nº Seguidores Dia])),
            [TotalSeguidores]
        )
    )

VAR _Count = COUNTROWS(_DailyTable)
VAR _SumX = SUMX(_DailyTable, [RankX])
VAR _SumY = SUMX(_DailyTable, [RankY])
VAR _SumXY = SUMX(_DailyTable, [RankX] * [RankY])
VAR _SumX2 = SUMX(_DailyTable, [RankX] * [RankX])
VAR _SumY2 = SUMX(_DailyTable, [RankY] * [RankY])

VAR _MeanX = DIVIDE(_SumX, _Count)
VAR _MeanY = DIVIDE(_SumY, _Count)

VAR _Numerator = _SumXY - (_Count * _MeanX * _MeanY)
VAR _DenominatorX = _SumX2 - (_Count * _MeanX * _MeanX)
VAR _DenominatorY = _SumY2 - (_Count * _MeanY * _MeanY)
VAR _Denominator = SQRT(_DenominatorX * _DenominatorY)

RETURN DIVIDE(_Numerator, _Denominator)
```

#### Interpretação das Correlações

| Valor de r | Interpretação |
|------------|---------------|
| 0,7 a 1,0 | Correlação forte |
| 0,3 a 0,7 | Correlação moderada |
| 0,1 a 0,3 | Correlação fraca |
| -0,1 a 0,1 | Correlação negligível |
| -0,3 a -0,1 | Correlação fraca negativa |
| -0,7 a -0,3 | Correlação moderada negativa |
| -1,0 a -0,7 | Correlação forte negativa |

**⚠️ IMPORTANTE: CORRELAÇÃO NÃO IMPLICA CAUSALIDADE**

<br>

#### **`Teste de Normalidade`**
> Avalia normalidade das distribuições via Skewness e Kurtosis:

```dax
Teste Normalidade = 
VAR _DailyTable = 
    SUMMARIZE(
        'MetaAds Analytics',
        'MetaAds Analytics'[Data],
        "TotalVisitas", SUM('MetaAds Analytics'[Visitas ao perfil]),
        "TotalSeguidores", SUM('MetaAds Analytics'[Nº Seguidores Dia])
    )

VAR _Count = COUNTROWS(_DailyTable)

-- Cálculos para Visitas
VAR _MeanX = AVERAGEX(_DailyTable, [TotalVisitas])
VAR _StdDevX = SQRT(SUMX(_DailyTable, POWER([TotalVisitas] - _MeanX, 2)) / _Count)
VAR _SkewX = SUMX(_DailyTable, POWER(([TotalVisitas] - _MeanX) / _StdDevX, 3)) / _Count
VAR _KurtX = (SUMX(_DailyTable, POWER(([TotalVisitas] - _MeanX) / _StdDevX, 4)) / _Count) - 3

-- Cálculos para Seguidores  
VAR _MeanY = AVERAGEX(_DailyTable, [TotalSeguidores])
VAR _StdDevY = SQRT(SUMX(_DailyTable, POWER([TotalSeguidores] - _MeanY, 2)) / _Count)
VAR _SkewY = SUMX(_DailyTable, POWER(([TotalSeguidores] - _MeanY) / _StdDevY, 3)) / _Count
VAR _KurtY = (SUMX(_DailyTable, POWER(([TotalSeguidores] - _MeanY) / _StdDevY, 4)) / _Count) - 3

-- Interpretação simplificada
VAR _NormalX = IF(ABS(_SkewX) < 0.5 && ABS(_KurtX) < 2, "Normal", "Não-Normal")
VAR _NormalY = IF(ABS(_SkewY) < 0.5 && ABS(_KurtY) < 2, "Normal", "Não-Normal")

RETURN 
    "VISITAS: " & _NormalX & " (Skew: " & FORMAT(_SkewX, "0.00") & ", Kurt: " & FORMAT(_KurtX, "0.00") & ")" &
    " | SEGUIDORES: " & _NormalY & " (Skew: " & FORMAT(_SkewY, "0.00") & ", Kurt: " & FORMAT(_KurtY, "0.00") & ")"
```

- **Skewness (Assimetria)**:
  - `|Skew| < 0.5`: Distribuição aproximadamente simétrica
  - `0.5 ≤ |Skew| < 1`: Moderadamente assimétrica
  - `|Skew| ≥ 1`: Altamente assimétrica

- **Kurtosis (Curtose)**:
  - `|Kurt| < 2`: Distribuição aproximadamente normal
  - `|Kurt| ≥ 2`: Distribuição não-normal

<br>

## Aprendizados e Insights
Este projeto reforça a importância de uma abordagem data-driven no marketing digital. A construção de um dashboard personalizado e a análise integrada de KPIs permitem:

* Identificar gargalos no funil de aquisição.

* Otimizar o orçamento de campanha ao direcioná-lo para as estratégias mais eficientes.

* Validar a qualidade do tráfego e a capacidade de conversão do perfil.

* Tomar decisões proativas baseadas em dados, e não em suposições.

<!-- -->

<br>

## 🔧 Recursos Interativos
- **Slicers Dinâmicos**: Seleção de métricas via parâmetros
- **Filtros Temporais**: Análise granular por período
- **Drill-down Hierárquico**: Navegação Campanha → Conjunto → Anúncio
- **Cross-filtering**: Filtros cruzados entre visuais

<br>

## 📚 Documentação Técnica

- [Power BI Best Practices](https://docs.microsoft.com/en-us/power-bi/)
- [DAX Guide](https://dax.guide/)





