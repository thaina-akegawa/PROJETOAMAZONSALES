# PROJETO4 - DataLab Amazon Sales

**Introdução:**
Este projeto foi realizado em dupla com o objetivo de aplicar os conhecimentos adquiridos no bootcamp para analisar dados de vendas da Amazon. Optamos por focar na análise de vendas da Amazon devido à sua relevância global e à disponibilidade de um conjunto de dados robusto.

**Descrição do Projeto:**
Escolhemos analisar dados de vendas da Amazon, utilizando a base fornecida para explorar padrões de desempenho de produtos e comportamento do consumidor. O objetivo é extrair insights que possam otimizar estratégias de marketing e vendas da empresa.

**Planejamento e Organização:**
- Reuniões semanais foram agendadas às segundas, terças e quartas-feiras, com sessões adicionais às sextas-feiras à tarde para discutir progresso e estratégias.
- A abordagem foi flexível, permitindo ajustes conforme avançávamos na análise e na limpeza de dados.

**Processo de Limpeza de Dados:**

1. **União de Tabelas:**
   ```sql
   CREATE OR REPLACE TABLE projeto4-tb.projeto4.amazon_product_reviews AS
   SELECT
       p.*,
       r.user_id,
       r.review_title,
       r.review_content,
       r.rating,
       r.rating_count
   FROM
       `projeto4-tb.projeto4.amazon_product` AS p
   LEFT JOIN
       `projeto4-tb.projeto4.amazon_review` AS r
   ON
       p.product_id = r.product_id;
   ```
   - **Explicação:** Esta consulta SQL combina duas tabelas relacionadas (`amazon_product` e `amazon_review`) com base no `product_id`, permitindo a análise conjunta de informações de produtos e avaliações dos clientes.

2. **Identificação de Duplicados:**
   ```sql
   SELECT
       user_id,
       product_id,
       COUNT(*) AS num_duplicates
   FROM projeto4-tb.projeto4.amazon_product_reviews
   GROUP BY user_id, product_id
   HAVING COUNT(*) > 1;
   ```
   - **Explicação:** Esta consulta identifica registros duplicados na tabela `amazon_product_reviews` com base nos campos `user_id` e `product_id`, permitindo que esses duplicados sejam tratados posteriormente para manter a integridade dos dados.

3. **Remoção de Duplicados:**
   ```sql
   WITH CTE AS (
       SELECT
           user_id,
           product_id,
           ROW_NUMBER() OVER(PARTITION BY user_id, product_id ORDER BY (SELECT NULL)) AS RowNumber
       FROM projeto4-tb.projeto4.amazon_product_reviews
   )
   DELETE
   FROM CTE WHERE RowNumber > 1;
   ```
   - **Explicação:** Utilizando uma Common Table Expression (CTE), esta query remove duplicatas da tabela `amazon_product_reviews`, mantendo apenas uma entrada para cada `user_id` e `product_id` com base em critérios de ordenação definidos.

4. **Criação de Variáveis Novas:**

   a) **Criar a variável `discount_percentage_percent`:**
   ```sql
   ALTER TABLE projeto4-tb.projeto4.amazon_product_reviews ADD COLUMN discount_percentage_percent STRING;
   UPDATE projeto4-tb.projeto4.amazon_product_reviews
   SET discount_percentage_percent = CONCAT(CAST(ROUND(discount_percentage * 100, 1) AS STRING), '%')
   WHERE 1:1;
   ```
   - **Explicação:** Esta série de comandos SQL adiciona uma nova coluna `discount_percentage_percent` à tabela `amazon_product_reviews` e calcula o percentual de desconto de cada produto, convertendo-o para uma string formatada com um único decimal seguido de `%`.

   b) **Categorização da `product_popularity`:**
   ```sql
   CREATE OR REPLACE TABLE projeto4-tb.projeto4.amazon_product_reviews AS
   SELECT
       *,
       CASE
           WHEN SAFE_CAST(rating AS FLOAT64) BETWEEN 4.0 AND 5.0 THEN 'ALTA'
           WHEN SAFE_CAST(rating AS FLOAT64) BETWEEN 3.0 AND 4.0 THEN 'MÉDIA'
           WHEN SAFE_CAST(rating AS FLOAT64) >= 0.0 AND SAFE_CAST(rating AS FLOAT64) < 3.0 THEN 'BAIXA'
       END AS product_popularity
   FROM projeto4-tb.projeto4.amazon_product_reviews;
   ```
   - **Explicação:** Esta consulta SQL adiciona a coluna `product_popularity` à tabela `amazon_product_reviews`, categorizando a popularidade do produto com base nas avaliações dos clientes. Classifica os produtos como 'ALTA', 'MÉDIA' ou 'BAIXA' com base na pontuação de avaliação.

5. **Análise de Quartis para `reliability_avaliation`:**
   ```sql
   CREATE OR REPLACE TABLE projeto4-tb.projeto4.amazon_product_reviews AS
   SELECT
       *,
       CASE
           WHEN NTILE(4) OVER (ORDER BY rating_count) = 1 THEN 'BAIXA'
           WHEN NTILE(4) OVER (ORDER BY rating_count) = 2 THEN 'MÉDIA-BAIXA'
           WHEN NTILE(4) OVER (ORDER BY rating_count) = 3 THEN 'MÉDIA-ALTA'
           WHEN NTILE(4) OVER (ORDER BY rating_count) = 4 THEN 'ALTA'
           ELSE 'Outro'
       END AS popularity_avaliation
   FROM projeto4-tb.projeto4.amazon_product_reviews_temp;
   ```
   - **Explicação:** Esta consulta SQL calcula os quartis da coluna `rating_count` na tabela `amazon_product_reviews_temp` e categoriza a `popularity_avaliation` com base nesses quartis. Classifica os produtos em quatro categorias de popularidade: 'BAIXA', 'MÉDIA-BAIXA', 'MÉDIA-ALTA' e 'ALTA'.

**Implementação no Power BI:**
- Utilizamos o Power BI para criar visualizações detalhadas e interativas.
- Exploramos padrões de vendas por categorias principais como Home&Kitchen, Eletronics e Computer.
- Desenvolvemos gráficos dinâmicos para apresentar insights sobre desempenho de vendas e eficácia de descontos.

**Apresentação no Google Slides:**
- Preparamos uma apresentação detalhada utilizando o Google Slides para comunicar os resultados e insights obtidos.
- Incluímos gráficos e visualizações do Power BI para ilustrar de forma clara e impactante os principais achados do projeto.

**Conclusão:**
O projeto proporcionou uma análise abrangente e detalhada dos dados de vendas da Amazon, utilizando ferramentas avançadas como Power BI e Google Slides para apresentar insights valiosos. As conclusões obtidas serão fundamentais para orientar estratégias futuras da empresa, visando melhorar o desempenho de produtos e otimizar a experiência do consumidor.

---
