# Criação_Banco_de_Dados_Instagram.ipynb

Este script Python implementa um gerador de dados sintéticos para métricas de marketing digital do Instagram, simulando campanhas com dados realísticos de conversão e engajamento. 
O código cria um dataset abrangente com 20.000 registros, mantendo taxas de conversão entre visitas ao perfil e ganho de seguidores dentro de parâmetros realísticos (10-15%).

## ⚙️ Funcionalidades

- **Geração de Campanhas Sequenciais**: Cria campanhas não sobrepostas com duração variável (60-180 dias).

- **Métricas Realísticas de Conversão**: Implementa algoritmo para manter taxa de conversão entre 10-15% (visitas → novos seguidores), com correlação forte (0.65-0.85) entre as variáveis.

- **Crescimento Simulado**: Modela crescimento médio de 164 novos seguidores/dia, com variações naturais.

- **Métricas Abrangentes de Marketing**: Gera dados completos incluindo:
  - Impressões, alcance e frequência
  - CTR, CPC, CPM e investimento
  - Conversões, CPA e ROAS
  - Engajamento (curtidas, comentários, compartilhamentos)
  - Métricas específicas do Instagram (salvamentos, cliques no perfil, mensagens)

- **Controle de Qualidade**: Implementa validações para garantir:
  - Ausência de valores nulos
  - Taxas de conversão dentro do range esperado
  - Correlação adequada entre variáveis-chave
  - Ajuste automático para atingir targets específicos

- **Variabilidade Realística**: Inclui flutuações naturais como:
  - Dias com perda de seguidores (5% dos dias)
  - Variação sazonal nas métricas
  - Distribuição não uniforme entre anúncios

- **Saída Estruturada**: Gera arquivo CSV com dados organizados cronologicamente, incluindo:
  - Identificação de campanha e anúncio
  - Métricas de performance diárias
  - Seguidores acumulados
  - Dados de engajamento e conversão

## 📊 Estrutura dos Dados

O dataset final contém as seguintes colunas principais:

- **Identificação**: Id, Data, Campanha, Conjunto de Anúncios, Anúncio, Plataforma
- **Métricas de Alcance**: Impressões, Alcance, Frequência
- **Performance**: CTR, CPC, CPM, Investimento, Conversões, CPA, ROAS, ROI
- **Engajamento**: Curtidas, Comentários, Compartilhamentos, Salvamentos
- **Conversão**: Visitas ao perfil, Nº Seguidores Dia, Nº Seguidores (acumulado)
- **Específicas**: Cliques no perfil, Mensagens iniciadas, Visualizações da página de destino

O código utiliza métodos estatísticos avançados para garantir que os dados sintéticos mantenham características realísticas de campanhas de marketing digital, incluindo variabilidade natural, sazonalidade e correlações esperadas entre diferentes métricas de performance.

## ⚠️ Disclaimer

Este script foi desenvolvido com o auxílio de Inteligência Artificial (IA).
