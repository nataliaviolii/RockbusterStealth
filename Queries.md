## **Some Queries Used Throughout the Project**

### Join Table
#### Find the top 20 countries with highest total payment:
```
SELECT country,
       SUM(amount) AS total_payment,
       COUNT(customer.customer_id) AS customer_count,
       SUM(amount)/COUNT(customer.customer_id) AS average_payment_per_customer
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
INNER JOIN payment ON customer.customer_id = payment.customer_id
GROUP BY country
ORDER BY total_payment DESC
LIMIT 20;
```

#### Find the top 10 countries for Rockbuster in terms of customer numbers:
```
SELECT country,
       COUNT(customer_id) AS customer_count
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
GROUP BY country
ORDER BY customer_count DESC
LIMIT 10;
```

#### Find the top 10 cities within the top 10 countries identified in the previous step:
```
SELECT country,
       city,
       COUNT(customer_id) AS customer_count
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
WHERE country IN('India', 'China', 'United States', 'Japan', 'Mexico', 'Brazil', ' Russian Federation', ' Philippines', ' Turkey', 'Indonesia')
GROUP BY country, city
ORDER BY customer_count DESC
LIMIT 10; 
```


### Subqueries
#### Find the average amount paid by the top 5 customers (sum then average):
```
SELECT AVG(total_amount_paid) AS average
FROM
    (SELECT customer.customer_id, first_name, last_name,
            country,
            city,
            SUM(amount) AS total_amount_paid
    FROM payment
    INNER JOIN customer ON payment.customer_id = customer.customer_id
    INNER JOIN address ON customer.address_id = address.address_id
    INNER JOIN city ON address.city_id = city.city_id
    INNER JOIN country ON city.country_id = country.country_id
    WHERE city IN('Aurora', 'Acua', 'Citrus Heights', 'Iwaki', 'Ambattur', 'Shanwei', 'So Leopoldo', 'Tianjin', 'Hami', 'Cianjur')
    GROUP BY customer.customer_id, first_name, last_name, country, city
    LIMIT 5) AS sub
```

#### Find out how many of the top 5 customers are based within each country (sum then count):
```
SELECT country.country,
       COUNT(DISTINCT customer.customer_id) AS all_customer_count,
       COUNT(DISTINCT top_5_customers) AS top_customer_count
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
LEFT JOIN
        (SELECT customer.customer_id, first_name, last_name,
                country,
                city,
                SUM(amount) AS total_amount_paid
        FROM payment
        INNER JOIN customer ON payment.customer_id = customer.customer_id
        INNER JOIN address ON customer.address_id = address.address_id
        INNER JOIN city ON address.city_id = city.city_id
        INNER JOIN country ON city.country_id = country.country_id
        WHERE city IN('Aurora', 'Acua', 'Citrus Heights', 'Iwaki', 'Ambattur', 'Shanwei', 'So Leopoldo', 'Tianjin', 'Hami', 'Cianjur')
        GROUP BY customer.customer_id, first_name, last_name, country, city
        ORDER BY total_amount_paid DESC
        LIMIT 5) AS top_5_customers ON customer.customer_id = top_5_customers.customer_id
GROUP BY country.country
ORDER BY all_customer_count DESC, top_customer_count
LIMIT 5;
```
#### Notes:
> * In this case, I can't do two steps above without subqueries (or CTEs) as I am doing multiple levels of aggregations, where I sum first then proceed to average or I sum first then proceed to count
> * The sum here is calculated with the help of the aggregate function SUM() and is returned by the subquery. In other words, I can't replace the subqueries because I don’t have a table with the sum previously calculated


### Common Table Expressions (CTE)
#### Practice using CTE for the subqueries above:
```
WITH top_5_customers_cte (customer_id, first_name, last_name, country, city, total_amount_paid) AS
    (SELECT customer.customer_id, first_name, last_name,
            country,
            city,
            SUM(amount) AS total_amount_paid
    FROM payment
    INNER JOIN customer ON payment.customer_id = customer.customer_id
    INNER JOIN address ON customer.address_id = address.address_id
    INNER JOIN city ON address.city_id = city.city_id
    INNER JOIN country ON city.country_id = country.country_id
    WHERE city IN('Aurora', 'Acua', 'Citrus Heights', 'Iwaki', 'Ambattur', 'Shanwei', 'So Leopoldo', 'Tianjin', 'Hami', 'Cianjur')
    GROUP BY customer.customer_id, first_name, last_name, country, city
    ORDER BY total_amount_paid DESC
    LIMIT 5)

SELECT country.country,
       COUNT(DISTINCT customer.customer_id) AS all_customer_count,
       COUNT(DISTINCT top_5_customers_cte) AS top_customer_count
FROM customer
INNER JOIN address ON customer.address_id = address.address_id
INNER JOIN city ON address.city_id = city.city_id
INNER JOIN country ON city.country_id = country.country_id
LEFT JOIN top_5_customers_cte ON customer.customer_id = top_5_customers_cte.customer_id
GROUP BY country.country
ORDER BY all_customer_count DESC, top_customer_count
LIMIT 5;
```

#### Notes:
> * Some of the main things that make CTEs advantageous over subqueries are maintainability and multiple referencing
> * Later (in complex queries) I believe CTE will be more useful because I already store the value in the CTE, so whenever I need it, I can simply refer to it by the CTE name
> * If I want to do an update then it will be more efficient because I only need to update the CTE query (I only need to change it in one place) and call it back once it is fixed
> * Since things get complicated later, CTE will be the best option for making queries readable
