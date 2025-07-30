# Meta Ads Analytics Dashboard 📊

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)

Este projeto foi desenvolvido para analisar campanhas de Intagram com objetivos de ganho de novos seguidores. Os dados do projeto são fictícios, com fins demonstrativos apenas.

Em edição...
<!--
## 🎯 Objetivos do Projeto

- **Análise de Performance**: Monitoramento de campanhas publicitárias focadas em crescimento de seguidores
- **Eficiência de Investimento**: Cálculo de métricas como CPV (Custo Por Visita) e CPNS (Custo Por Novo Seguidor)
- **Correlações Estatísticas**: Análise da relação entre visitas ao perfil e novos seguidores
- **Análise Temporal**: Identificação de padrões e tendências ao longo do tempo

## 🏗️ Arquitetura do Modelo de Dados

### Visão Geral do Modelo
```
┌─────────────────┐     ┌──────────────────┐
│   Calendario    │────▶│  MetaAds         │
│   (Dimensão)    │     │  Analytics       │
└─────────────────┘     │  (Fato)          │
                        └──────────────────┘
                                │
                        ┌───────┴──────────┐
                        │                  │
                   ┌────▼────┐      ┌─────▼──────┐
                   │ Slicer  │      │   Slicer   │
                   │Medidas  │      │ Seguidores │
                   └─────────┘      └────────────┘
```

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

#### 👥 Slicer Seguidores
Tabela auxiliar para análise de seguidores.

| Campo | Valor | Ordem |
|-------|-------|-------|
| N° Seguidores | N° Seguidores Dia | 0 |
| Visitas Perfil | Visitas Perfil | 1 |

## 📊 Medidas DAX Principais

### 💰 Métricas de Investimento

#### Total Investimento
```dax
Total Investimento = SUM('MetaAds Analytics'[Investimento (R$)])/35
```
> **Nota**: Divisor aplicado para normalização dos dados fictícios

#### CPV (Custo Por Visita)
```dax
CPV = DIVIDE([Total Investimento],[Visitas Perfil],0)*3
```
Calcula o custo por visita ao perfil, métrica fundamental para avaliar eficiência.

#### CPNS (Custo Por Novo Seguidor)
```dax
CPNS = DIVIDE([Total Investimento],[Total Novos Seguidores],0)*5
```
Métrica crucial para campanhas focadas em crescimento de audiência.

### 📈 Métricas de Performance

#### CTR (Click-Through Rate)
```dax
CTR = AVERAGE('MetaAds Analytics'[CTR])/75
```
Taxa de cliques normalizada para análise comparativa.

#### Taxa de Conversão
```dax
Taxa Conversão = DIVIDE([N° Seguidores Dia], [Visitas Perfil],0)
```
Percentual de visitas que se convertem em novos seguidores.

#### Total Novos Seguidores
```dax
Total Novos Seguidores = [Visitas Perfil] * [Taxa Conversão]
```
Cálculo derivado combinando visitas e taxa de conversão.

### 📊 Análises Estatísticas Avançadas

#### Correlação de Pearson
Analisa a correlação linear entre visitas ao perfil e novos seguidores:

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

#### Correlação de Spearman
Análise de correlação não-paramétrica baseada em rankings:

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

#### Teste de Normalidade
Avalia normalidade das distribuições via Skewness e Kurtosis:

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

## 🔄 Processo ETL - Power Query

### Conexão com Dados
O dashboard conecta-se a um banco PostgreSQL hospedado no Supabase:

```powerquery
// Conecta-se ao banco de dados PostgreSQL
Fonte = PostgreSQL.Database("aws-0-sa-east-1.pooler.supabase.com", "postgres")

// Seleciona a tabela 'meta_ads_instagram' do esquema 'public'
TabelaMetaAdsInstagram = Fonte{[Schema="public",Item="meta_ads_instagram"]}[Data]
```

### Transformações Principais

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

## 🎨 Funcionalidades do Dashboard

### 📊 Visuais Implementados
- **Cards de KPI**: Métricas principais (CTR, CPV, CPNS, Taxa de Conversão)
- **Gráficos de Linha**: Evolução temporal das métricas
- **Tabelas Detalhadas**: Breakdown por campanha, conjunto e anúncio
- **Análises Estatísticas**: Correlações e testes de normalidade

### 🔧 Recursos Interativos
- **Slicers Dinâmicos**: Seleção de métricas via parâmetros
- **Filtros Temporais**: Análise granular por período
- **Drill-down Hierárquico**: Navegação Campanha → Conjunto → Anúncio
- **Cross-filtering**: Filtros cruzados entre visuais

### 📈 Insights Gerados
- **Eficiência de Investimento**: ROI por campanha e período
- **Correlações Estatísticas**: Relação entre visitas e conversões
- **Padrões Temporais**: Identificação de sazonalidades
- **Performance Comparativa**: Benchmarking entre campanhas

## 🛠️ Configuração e Uso

### Pré-requisitos
- Power BI Desktop (versão mais recente)
- Acesso ao arquivo `.pbix`
- Conhecimento básico em Power BI

### Estrutura de Arquivos
```
meta-ads-analytics/
├── README.md
├── MetaAds Analytics.pbix
├── meta_ads_readme.md
├── assets/
│   └── images/
│       ├── dashboard-overview.png
│       └── data-model.png
└── docs/
    └── technical-documentation.md
```

### Parâmetros Configuráveis
| Parâmetro | Tipo | Descrição | Valor Padrão |
|-----------|------|-----------|--------------|
| `RangeStart` | DateTime | Data de início da análise | (dinâmico) |
| `RangeEnd` | DateTime | Data de fim da análise | (dinâmico) |

## 📊 Interpretação das Métricas

### KPIs Principais

| Métrica | Fórmula | Interpretação | Meta Sugerida |
|---------|---------|---------------|---------------|
| **CTR** | Cliques ÷ Impressões | Taxa de engajamento | > 1% |
| **CPV** | Investimento ÷ Visitas | Custo por visita | < R$ 2,00 |
| **CPNS** | Investimento ÷ Novos Seguidores | Custo por seguidor | < R$ 5,00 |
| **Taxa de Conversão** | Seguidores ÷ Visitas | Eficácia da conversão | > 15% |

### Correlações Estatísticas

| Valor | Interpretação |
|-------|---------------|
| **0.8 a 1.0** | Correlação muito forte |
| **0.6 a 0.8** | Correlação forte |
| **0.4 a 0.6** | Correlação moderada |
| **0.2 a 0.4** | Correlação fraca |
| **0.0 a 0.2** | Correlação muito fraca |

## 🔍 Metodologia e Limitações

### Metodologia Estatística
- **Correlação de Pearson**: Relações lineares
- **Correlação de Spearman**: Relações monotônicas
- **Teste de Normalidade**: Skewness < 0.5 e Kurtosis < 2

### Limitações do Projeto
- Dados fictícios/anonimizados para demonstração
- Algumas métricas normalizadas com divisores
- Testes estatísticos simplificados (sem intervalos de confiança)
- Sem análise de significância estatística

### Considerações Técnicas
- Performance otimizada para datasets de médio porte
- Relacionamentos configurados para máxima eficiência
- Medidas DAX otimizadas para cálculos complexos

## 🚀 Roadmap e Melhorias Futuras

### Próximas Funcionalidades
- [ ] **Previsões Temporais**: Implementação de forecasting
- [ ] **Análise de Cohort**: Acompanhamento de coortes de seguidores
- [ ] **Alertas Automáticos**: Notificações para KPIs fora do padrão
- [ ] **Benchmark de Mercado**: Comparação com dados de referência

### Integrações Planejadas
- [ ] **Google Ads**: Análise cross-platform
- [ ] **TikTok Ads**: Expansão para outras redes
- [ ] **API em Tempo Real**: Atualizações automáticas
- [ ] **Machine Learning**: Modelos preditivos avançados

### Melhorias Técnicas
- [ ] **Testes A/B**: Framework para experimentos
- [ ] **Segmentação Avançada**: Análise por audiência
- [ ] **ROI Detalhado**: Análise financeira aprofundada
- [ ] **Mobile First**: Otimização para dispositivos móveis

## 📚 Recursos Adicionais

### Documentação Técnica
- [Power BI Best Practices](https://docs.microsoft.com/en-us/power-bi/)
- [DAX Guide](https://dax.guide/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

### Artigos Relacionados
- Análise de Performance em Marketing Digital
- Estatística Aplicada ao Marketing
- Data Modeling em Power BI

## 👥 Contribuição

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto é desenvolvido para fins educacionais e de demonstração. Os dados utilizados são fictícios.

## 📞 Contato

Para dúvidas, sugestões ou colaborações, abra uma issue neste repositório ou entre em contato através do LinkedIn.

---

**Desenvolvido com** ❤️ **usando Power BI, DAX e PostgreSQL**

*Dashboard criado para demonstrar capacidades analíticas avançadas em Business Intelligence*
