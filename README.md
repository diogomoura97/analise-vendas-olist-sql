Análise de E-commerce com SQL Server (Olist)



 Sobre o Projeto

Este projeto faz parte do meu portfólio de desenvolvimento em **SQL**, utilizando o **SQL Server Management Studio (SSMS) 2022**. O objetivo foi simular um cenário real de mercado utilizando a base de dados pública da **Olist** (maior loja unificada do e-commerce brasileiro no Kaggle). 



O projeto aborda desde a engenharia de dados inicial (criação de tabelas e limpeza de arquivos CSV) até a extração de inteligência de negócios através de consultas SQL.



Arquitetura e Modelagem do Banco de Dados

A base de dados é composta por 9 tabelas relacionais que mapeiam o processo completo de um pedido de e-commerce: clientes, produtos, pedidos, itens, pagamentos, avaliações, vendedores, geolocalização e tradução de categorias.









Principais Análises de Negócio (Queries)

### 1. Top Categorias por Receita
Identifica quais categorias de produtos trazem o maior faturamento bruto para a plataforma.
* **Conceitos aplicados:** `JOIN` múltiplo, Funções de Agregação (`SUM`, `COUNT`), Ordenação (`ORDER BY`) e Limitação de Resultados (`TOP`).


SELECT TOP 10
    t.product_category_name_english AS Categoria_Produto,
    COUNT(DISTINCT oi.order_id) AS Total_Pedidos,
    CAST(SUM(oi.price) AS DECIMAL(10,2)) AS Receita_Total_R$
FROM olist_order_items_dataset oi
JOIN olist_products_dataset p 
    ON oi.product_id = p.product_id
JOIN product_category_name_translation t 
    ON p.product_category_name = t.product_category_name
GROUP BY 
    t.product_category_name_english
ORDER BY 
    Receita_Total_R$ DESC;




    


    
2. Eficiência Logística por Estado
Mede o tempo médio que os pedidos levam para serem entregues aos clientes em cada estado brasileiro.

Conceitos aplicados: Manipulação de datas (DATEDIFF), filtros condicionais avançados e agregação por estado.

SQL
SELECT 
    c.customer_state AS Estado,
    COUNT(o.order_id) AS Total_Pedidos_Entregues,
    AVG(DATEDIFF(DAY, o.order_purchase_timestamp, o.order_delivered_customer_date)) AS Tempo_Medio_Entrega_Dias
FROM olist_orders_dataset o
JOIN olist_customers_dataset c 
    ON o.customer_id = c.customer_id
WHERE 
    o.order_status = 'delivered' 
    AND o.order_delivered_customer_date IS NOT NULL
GROUP BY 
    c.customer_state
ORDER BY 
    Tempo_Medio_Entrega_Dias ASC;







    
3. Taxa de Atrasos nas Entregas
Calcula a porcentagem geral de pedidos entregues após a data estimada de entrega prometida ao cliente.

Conceitos aplicados: Expressões de Tabela Comuns (CTEs), Lógica Condicional (CASE WHEN), e cálculos com conversão de ponto flutuante.

SQL
WITH Status_Entregas AS (
    SELECT 
        order_id,
        CASE 
            WHEN order_delivered_customer_date > order_estimated_delivery_date THEN 1 
            ELSE 0 
        END AS Entregue_Atrasado
    FROM olist_orders_dataset
    WHERE order_status = 'delivered' 
      AND order_delivered_customer_date IS NOT NULL
)
SELECT 
    COUNT(order_id) AS Total_Pedidos_Avaliados,
    SUM(Entregue_Atrasado) AS Total_Atrasados,
    CAST((SUM(Entregue_Atrasado) * 100.0) / COUNT(order_id) AS DECIMAL(5,2)) AS Porcentagem_Atraso
FROM Status_Entregas;
