# db-brazilianEcommerce.ipynb

Este script Python implementa um pipeline ETL para criar e popular o banco de dados PostgreSQL `brazilian_ecommerce` com dados do dataset *Brazilian E-commerce* (Kaggle). Ele automatiza a criação de tabelas, a extração e transformação de dados de arquivos CSV e a carga com validação de integridade, preparando o ambiente para análises, como no Power BI.

## ⚙️ Funcionalidades

- **Criação do Banco de Dados**: Inicializa automaticamente o banco `brazilian_ecommerce` no PostgreSQL
- **Definição do Schema**: Cria tabelas (`orders`, `customers`, `products`, `order_items`, etc.) com chaves primárias e estrangeiras, incluindo a tabela `zip_code_master` para consolidar CEPs únicos.
- **Extração e Transformação**: Lê e processa arquivos CSV, tratando tipos de dados, removendo duplicatas e filtrando registros inválidos com base em chaves estrangeiras.
- **Carga de Dados**: Carrega os dados em tabelas, respeitando dependências (ex.: `product_category_name_translation` antes de `products`, `customers` antes de `orders`).
- **Validação de Integridade**: Verifica e corrige inconsistências, como CEPs ausentes ou categorias de produtos faltantes, inserindo traduções padrão quando necessário.
- **Modularidade**: Organizado em funções reutilizáveis para conexão, configuração de tabelas e carregamento de dados.
- **Execução Controlada**: Orquestra o ETL em sequência lógica, garantindo a ordem correta e a integridade referencial.
