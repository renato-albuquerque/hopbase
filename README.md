# Projeto HOPBASE
Solução de Business Intelligence (BI) 100% open source, utilizando soluções gratuitas e robustas. Processo de ETL com [Apache Hop](https://hop.apache.org/) e Dataviz com [Metabase](https://www.metabase.com/). <br>

Time envolvido: <br>
[Renato Albuquerque](https://www.linkedin.com/in/renato-malbuquerque/) <br>
[Rafael Arruda](https://www.linkedin.com/in/rafael-arruda-39145738/) <br>
[Arruda Data Consulting](https://arrudaconsulting.com.br/) <br>

## 01. Arquitetura do Projeto
![arquitetura_hopbase](images/hopbase_mysql.JPG)

## 02. Preparação do Ambiente
Passo-a-passo detalhado: [Projeto Hopbase](https://hotmart.com/pt-BR/club/formacao-pentarruda/products/5599808/content/r488xZ2n4R?track=Pb4K0mLbeX).

### 📌 Banco de Dados
Uniserver: Banco de Dados MySQL. <br>
Unicontroller: Software para ativar o banco de dados. <br>
HeidiSQL: IDE para desenvolvimento. <br>

### 📌 Java
Criar variável de ambiente (JAVA_HOME).

### 📌 Apache Hop (Hop Orchestration Platform)
É uma plataforma de código aberto voltada para a engenharia, integração e orquestração de dados e metadados. <br>
Instalar o software [Apache Hop](https://hop.apache.org/download/). <br>
Realizar conexão do Apache Hop com o Banco de Dados (Drive de conexão do MySQL).

### 📌 Metabase
Ferramenta de Business Intelligence (BI) e análise de dados de código aberto (open source). Permite conectar com bancos de dados e criar gráficos, métricas e painéis visuais (dashboards). <br>
Instalar o software [Metabase](https://www.metabase.com/docs/latest/installation-and-operation/running-the-metabase-jar-file
). <br>
Seguir orientações da documentação do Metabase.

### 📌 SQL Power Architect
Ferramenta gráfica e de código aberto voltada para a **modelagem de bancos de dados e design de Data Warehouses**. <br>
Instalar o software SQL Power Architect.

### 📌 Resumo Tecnologias
![tecnologias_do_projeto](images/project_softwares.JPG)

## 03. Banco de Dados do Projeto: "north"
📝 Criar Banco de Dados (north) no HeidiSQL ("Rodar" arquivo DDL.sql). <br>
📝 Também foi criado os BDs: dev, qa, prod. <br>
📝 Ilustração BD north abaixo (12 tabelas): <br>
![bd_north](images/bd_north.JPG)

## 04. Staging Area (Criação e Carga de dados)
📝 Através do SQL Power Architect, foram criadas as tabelas da camada STAGING, no banco de dados "dev". <br>
📝 Documentação das tabelas da camada STAGING, gerado através do SQL Power Architect:
![sql_power_architect_documentacao_staging](images/sql_power_architect_documentacao_staging.JPG)
📝 Apache Hop: Desenvolvimento dos PIPELINES para carga de dados de cada tabela na camada STAGING. <br>
📝 Apache Hop: Desenvolvimento de WORKFLOW para carga completa e simultânea dos dados na camada STAGING. <br>
PIPELINES
![apache_hop_pipelines_staging](images/apache_hop_staging.JPG) <br> 
WORKFLOW
![apache_hop_workflow_staging](images/apache_hop_workflow_staging.JPG) <br>

## 05. Data Warehouse (DW)
📝 MySQL / HeidiSQL: Desenvolvimento da tabela FATO ft_orders (INNER JOIN entre as tabelas dev.st_orders & dev.st_order_details) <br>

```
SELECT
	o.order_id,
	o.customer_id AS sk_customer,
	o.employee_id AS sk_employee,
	od.product_id AS sk_product,
	o.order_date,
	o.required_date,
	o.shipped_date,
	od.unit_price,
	od.quantity,
	od.discount,
	ROUND(od.unit_price * od.quantity, 2) AS valor_bruto,
	ROUND((od.unit_price * od.quantity) * od.discount, 2) AS valor_desconto,
	ROUND((od.unit_price * od.quantity) - ((od.unit_price * od.quantity) * od.discount), 2) AS valor_liquido
FROM dev.st_orders o
INNER JOIN dev.st_order_details od
ON o.order_id = od.order_id;
```

📝 Através do SQL Power Architect, foram criadas as tabelas DIMENSÕES e FATO da camada DW, no banco de dados "dev". <br>
![sql_power_architect_dw](images/sql_power_architect_dw.JPG)

📝 Documentação das tabelas da camada DW, gerado através do SQL Power Architect:
![sql_power_architect_documentacao_dw](images/sql_power_architect_documentacao_dw.JPG)

🌟 Dimensão dim_customer <br>
Para a dim_customer, será apresentado 02 abordagens para carga dos dados: <br>
Carga Full x Carga SCD2 (Slowly Changing Dimension Tipo 2) <br>

✅ **Carga Full (Carga Completa):** Apaga e substitui todos os dados antigos a cada atualização. <br>

**Pipeline para carga full (Apache Hop)** <br>
![apache_hop_dim_customers_carga_full](images/apache_hop_dim_customers_carga_full.JPG)

✅ **Carga SCD2:** Método de modelagem de dados usado para guardar o histórico de alterações em tabelas dimensionais. <br>

1ª Etapa: Simulando atualização no bd produção (north), tabela customers. <br>
![heidi_sql_update_table_customers](images/heidi_sql_update_table_customers.JPG) <br>

2ª Etapa: Executar o workflow no apache hop st_area (Workflow com a carga de todas as stages). <br>
![apache_hop_workflow_staging](images/apache_hop_workflow_staging.JPG) <br>

3ª Etapa: **Pipeline para carga scd2 (Apache Hop)** <br>
![apache_hop_dim_customers_carga_scd2](images/apache_hop_dim_customers_carga_scd2.JPG) <br>

🌟 Dimensão dim_employee <br>
**Pipeline para carga full (Apache Hop)** <br>
![apache_hop_dim_employee_carga_full](images/apache_hop_dim_employee_carga_full.JPG) <br>

🌟 Dimensão dim_products <br>
**Pipeline para carga full (Apache Hop)** <br>
![apache_hop_dim_products_carga_full](images/apache_hop_dim_products_carga_full.JPG) <br>

🌟 Dimensão dim_calendario <br>
Tabela desenvolvida no Apache Hop. <br>
**Pipeline para carga full (Apache Hop)** <br>
![apache_hop_dim_calendario_carga_full](images/apache_hop_dim_calendario_carga_full.JPG) <br>

📝 Obs. Staging Area: Implementar a Staging da Tabela FATO. <br>
Através do SQL Power Architect, foi criada a tabela FATO na camada STAGING, st_fato_orders, no banco de dados "dev". <br>
Para suportar a tabela na camada DW, ft_orders. <br>
![sql_power_architect_staging_fato_orders](images/sql_power_architect_staging_fato_orders.JPG)

📝 Apache Hop: Desenvolvimento do PIPELINE para carga de dados da tabela st_fato_orders na camada STAGING. <br>
![apache_hop_st_fato_orders](images/apache_hop_st_fato_orders.JPG)

🌟 Tabela fato_orders <br>
Tabela desenvolvida no Apache Hop. <br>
(Lookups tabela st_fato_orders com: dim_customer_scd2, dim_employee, dim_products, dim_calendario. Para trazer as colunas sk's). <br>
**Pipeline para carga full (Apache Hop)** <br>
![apache_hop_fato_orders](images/apache_hop_fato_orders.JPG) <br>

Modelagem DW atualizada: <br>
![sql_power_architect_dw_updated](images/sql_power_architect_dw_updated.JPG) <br>

### CARGA INCREMENTAL

🌟 Dimensão dim_products <br>
![apache_hop_dim_products_carga_incremental](images/apache_hop_dim_products_carga_inc.JPG)

📝 Simulação alteração de dado. Alteração no BD de produção, tabela products, coluna units_in_stock (north.products). <br> 
![heidisql_products_alteracao_dado](images/heidisql_products_alteracao_dado.JPG)

📝 Apache Hop: Rodar o WORKFLOW staging area (st_area). <br>
📝 Apache Hop: Rodar o PIPELINE dim_products (carga_incremental). <br>
![apache_hop_products_alteracao_dado](images/apache_hop_products_alteracao_dado.JPG)

🌟 Dimensão dim_employee <br>
Outro método para carga incremental, utilizando apenas o step Synchronize after merge. <br>
![apache_hop_dim_employee_carga_inc](images/apache_hop_dim_employee_carga_inc.JPG)

🌟 Dimensão dim_calendario_dinamica <br>
Outra forma para desenvolvimento da dim_calendario. <br>
![apache_hop_dim_calendario_dinamica](images/apache_hop_dim_calendario_dinamica.JPG)

🌟 Dimensão dim_customer_scd2 <br>
Concluída na seção anterior.

### AJUSTANDO DOCUMENTAÇÃO DO DATA WAREHOUSE (DW) 

📝 Atualizando informações das tabelas DIMENSÃO na ferramenta SQL Power Architect.
![sql_power_architect_dw_latest_version](images/sql_power_architect_dw_latest_version.JPG)

📝 Criar WORKFLOW das tabelas dimensões. 
![apache_hop_workflow_dimensoes_dw](images/apache_hop_workflow_dimensoes_dw.JPG)

📝 Criar WORKFLOW da tabela fato.
![apache_hop_workflow_fato_dw](images/apache_hop_workflow_fato_dw.JPG)

📝 WORKFLOW geral para o DW.
![apache_hop_workflow_geral_dw](images/apache_hop_workflow_geral_dw.JPG)
