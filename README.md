# SQL

Foobar is a Python library for dealing with word pluralization.

## TEORIA

### MIN and MAX

Funcionalmente, MIN e MAX são semelhantes a COUNT , pois podem ser usados ​​em colunas não numéricas. Dependendo do tipo de coluna, MIN retornará o número mais baixo, a data mais antiga ou o valor não numérico o mais cedo possível no alfabeto. Como você pode suspeitar, MAX faz o oposto - retorna o número mais alto, a data mais recente ou o valor não numérico mais próximo alfabeticamente de “Z”.


### GOURP BY

1) GROUP BY pode ser usado para agregar dados dentro de subconjuntos de dados. Por exemplo, agrupamento para contas diferentes, regiões diferentes ou representantes de vendas diferentes.

2) Qualquer coluna na instrução SELECT que não esteja em um agregador deve estar na cláusula GROUP BY .
3) O GROUP BY sempre fica entre WHERE e ORDER BY .
4) ORDER BY funciona como SORT no software de planilha.
5) Você pode GROUP BY com várias colunas de uma vez, como mostramos aqui. Geralmente, isso é útil para agregar vários segmentos diferentes.
6) A ordem das colunas listadas na cláusula ORDER BY faz diferença. Você está ordenando as colunas da esquerda para a direita.
7) A ordem dos nomes das colunas em sua cláusula GROUP BY não importa - os resultados serão os mesmos independentemente. Se executarmos a mesma consulta e invertermos a ordem na cláusula GROUP BY , você verá que obtemos os mesmos resultados.
8) Assim como no ORDER BY , você pode substituir os nomes das colunas por números na cláusula GROUP BY . Geralmente, é recomendável fazer isso apenas quando você estiver agrupando muitas colunas ou se algo mais estiver fazendo com que o texto na cláusula GROUP BY seja excessivamente longo.
9) Um lembrete qualquer coluna que não esteja dentro de uma agregação deve aparecer em sua instrução GROUP BY. Se você esquecer, provavelmente obterá um erro.

### DISTINCT

Pode tornar a consulta mais lenta

### CASE

1) A instrução CASE sempre vai na cláusula SELECT.
2) CASE deve incluir os seguintes componentes: WHEN, THEN e END. ELSE é um componente opcional para capturar casos que não atendem a nenhuma das outras condições CASE anteriores.
3) Você pode fazer qualquer declaração condicional usando qualquer operador condicional (como WHERE ) entre WHEN e THEN. Isso inclui encadear várias instruções condicionais usando AND e OR.
4) Você pode incluir várias instruções WHEN, bem como uma instrução ELSE novamente, para lidar com quaisquer condições não resolvidas.

OBS.: Antes de nos aprofundarmos nas agregações usando instruções GROUP BY , é importante notar que o SQL avalia as agregações antes da cláusula LIMIT . Se você não agrupar por nenhuma coluna, obterá um resultado de 1 linha - sem problemas. Se você agrupar por uma coluna com valores únicos suficientes que exceda o número LIMIT , as agregações serão calculadas e, em seguida, algumas linhas serão simplesmente omitidas dos resultados.


## EXERCÍCIOS

### GROUP BY

1) Find the total sales in usd for each account. You should include two columns - the total sales for each company's orders in usd and the company name.

```sql
SELECT a.name, SUM(total_amt_usd) total_sales
FROM orders o
JOIN accounts a
ON a.id = o.account_id
GROUP BY a.name;
```

2) What was the smallest order placed by each account in terms of total usd. Provide only two columns - the account name and the total usd. Order from smallest dollar amounts to largest.

```sql
SELECT a.name, MIN(total_amt_usd) smallest_order
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY a.name
ORDER BY smallest_order;
```

3) Determine the number of times a particular channel was used in the web_events table for each sales rep. Your final table should have three columns - the name of the sales rep, the channel, and the number of occurrences. Order your table with the highest number of occurrences first.

```sql
SELECT s.name, w.channel, COUNT(*) num_events
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.name, w.channel
ORDER BY num_events DESC;
```

4) Determine the number of times a particular channel was used in the web_events table for each region. Your final table should have three columns - the region name, the channel, and the number of occurrences. Order your table with the highest number of occurrences first.

```sql
SELECT r.name, w.channel, COUNT(*) num_events
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
JOIN sales_reps s
ON s.id = a.sales_rep_id
JOIN region r
ON r.id = s.region_id
GROUP BY r.name, w.channel
ORDER BY num_events DESC;
```

### HAVING

1) Which accounts used facebook as a channel to contact customers more than 6 times?

```sql
SELECT a.id, a.name, w.channel, COUNT(*) use_of_channel
FROM accounts a
JOIN web_events w
ON a.id = w.account_id
GROUP BY a.id, a.name, w.channel
HAVING COUNT(*) > 6 AND w.channel = 'facebook'
ORDER BY use_of_channel;
```

2) How many accounts spent less than 1,000 usd total across all orders?

```sql
SELECT a.id, a.name, SUM(o.total_amt_usd) total_spent
FROM accounts a
JOIN orders o
ON a.id = o.account_id
GROUP BY a.id, a.name
HAVING SUM(o.total_amt_usd) < 1000
ORDER BY total_spent;
```


### DATE FUNCTION

1) In which month of which year did Walmart spend the most on gloss paper in terms of dollars?

```sql
SELECT DATE_TRUNC('month', o.occurred_at) ord_date, SUM(o.gloss_amt_usd) tot_spent
FROM orders o 
JOIN accounts a
ON a.id = o.account_id
WHERE a.name = 'Walmart'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```

2) Which month did Parch & Posey have the greatest sales in terms of total number of orders? Are all months evenly represented by the dataset?

```sql
SELECT DATE_PART('month', occurred_at) ord_month, COUNT(*) total_sales
FROM orders
WHERE occurred_at BETWEEN '2014-01-01' AND '2017-01-01'
GROUP BY 1
ORDER BY 2 DESC; 
```

### CASE

1) Write a query to display for each order, the account ID, total amount of the order, and the level of the order - ‘Large’ or ’Small’ - depending on if the order is $3000 or more, or less than $3000.

```sql
SELECT account_id, total_amt_usd,
CASE WHEN total_amt_usd > 3000 THEN 'Large'
ELSE 'Small' END AS order_level
FROM orders; 
```

2) Write a query to display the number of orders in each of three categories, based on the total number of items in each order. The three categories are: 'At Least 2000', 'Between 1000 and 2000' and 'Less than 1000'.

```sql
SELECT CASE WHEN total >= 2000 THEN 'At Least 2000'
   WHEN total >= 1000 AND total < 2000 THEN 'Between 1000 and 2000'
   ELSE 'Less than 1000' END AS order_category,
COUNT(*) AS order_count
FROM orders
GROUP BY 1; 
```

3) We would like to understand 3 different branches of customers based on the amount associated with their purchases. The top branch includes anyone with a Lifetime Value (total sales of all orders) greater than 200,000 usd. The second branch is between 200,000 and 100,000 usd. The lowest branch is anyone under 100,000 usd. Provide a table that includes the level associated with each account. You should provide the account name, the total sales of all orders for the customer, and the level. Order with the top spending customers listed first.

```sql
SELECT a.name, SUM(total_amt_usd) total_spent, 
     CASE WHEN SUM(total_amt_usd) > 200000 THEN 'top'
     WHEN  SUM(total_amt_usd) > 100000 THEN 'middle'
     ELSE 'low' END AS customer_level
FROM orders o
JOIN accounts a
ON o.account_id = a.id 
GROUP BY a.name
ORDER BY 2 DESC;
```

4) We would now like to perform a similar calculation to the first, but we want to obtain the total amount spent by customers only in 2016 and 2017. Keep the same levels as in the previous question. Order with the top spending customers listed first.

```sql
SELECT a.name, SUM(total_amt_usd) total_spent, 
     CASE WHEN SUM(total_amt_usd) > 200000 THEN 'top'
     WHEN  SUM(total_amt_usd) > 100000 THEN 'middle'
     ELSE 'low' END AS customer_level
FROM orders o
JOIN accounts a
ON o.account_id = a.id
WHERE occurred_at > '2015-12-31' 
GROUP BY 1
ORDER BY 2 DESC;
```

5) The previous didn't account for the middle, nor the dollar amount associated with the sales. Management decides they want to see these characteristics represented as well. We would like to identify top performing sales reps, which are sales reps associated with more than 200 orders or more than 750000 in total sales. The middle group has any rep with more than 150 orders or 500000 in sales. Create a table with the sales rep name, the total number of orders, total sales across all orders, and a column with top, middle, or low depending on this criteria. Place the top sales people based on dollar amount of sales first in your final table.

```sql
SELECT s.name, COUNT(*), SUM(o.total_amt_usd) total_spent, 
     CASE WHEN COUNT(*) > 200 OR SUM(o.total_amt_usd) > 750000 THEN 'top'
     WHEN COUNT(*) > 150 OR SUM(o.total_amt_usd) > 500000 THEN 'middle'
     ELSE 'low' END AS sales_rep_level
FROM orders o
JOIN accounts a
ON o.account_id = a.id 
JOIN sales_reps s
ON s.id = a.sales_rep_id
GROUP BY s.name
ORDER BY 3 DESC;
```

## Contributing
SQL for Data Analysis - [Udacity](http://udacity.com)

## License
[MIT](https://choosealicense.com/licenses/mit/)
