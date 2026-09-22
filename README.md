# 🎵 Digital Music Store Analysis - SQL Project

## 📌 Project Overview
This project explores a digital music store's relational database across multiple interconnected tables (Customers, Invoices, Tracks, Albums, Artists). The objective is to identify top revenue-generating markets, analyze customer lifetime spending, and evaluate popular musical genres.

---

## 🛠️ Tools & Technologies Used
- **Database Management System:** PostgreSQL
- **Query Tool/IDE:** pgAdmin 4
- **Language:** SQL (Multi-table JOINs, CTEs, Subqueries, Window Functions, Aggregations)

---

## 📊 Business Questions Solved
- **Organizational Structure:** Identified senior leadership and employee reporting hierarchies.
- **Geographic Revenue:** Found the top countries and cities generating the highest billing totals.
- **High-Value Customers:** Identified top spending customers to guide loyalty and retention campaigns.
- **Genre & Track Popularity:** Filtered and analyzed customer listening preferences by genre (e.g., Rock).
- **Artist Rankings:** Ranked top artists per customer and country using CTEs and DENSE_RANK().

---
## SQL Queries & solutions 
### Q1: Which city has the best customers?
```sql
SELECT sum(total) AS invoice_total,
       billing_city
FROM invoice
GROUP BY billing_city
ORDER BY invoice_total DESC;
```
### Q2: Write query to return the email, first name, last name, & Genre of all Rock Music listeners. Return your list ordered alphabetically by email starting with A.
```sql
SELECT DISTINCT email,
                first_name,
                last_name
FROM customer
JOIN invoice ON customer.customer_id=invoice.customer_id
JOIN invoice_line ON invoice.invoice_id=invoice_line.invoice_id
WHERE track_id IN
    (SELECT track_id
     FROM track
     JOIN genre ON track.genre_id=genre.genre_id
     WHERE genre.name like 'Rock')
ORDER BY email;
```
### Q3: Find how much amount spent by each customer on artists? Write a query to return customer name, artist name and total spent?
```sql
WITH best_selling_artist AS
  (SELECT at.artist_id artist_id,
          at.name AS artist_name,
          sum(il.unit_price*il.quantity) AS total_spent
   FROM invoice_line il
   JOIN track t ON t.track_id=il.track_id
   JOIN album al ON al.album_id=t.album_id
   JOIN artist AT ON at.artist_id=al.artist_id
   GROUP BY 1
   ORDER BY 3 DESC
   LIMIT 1 )
 SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, 
 				SUM(il.unit_price*il.quantity) AS amount_spent
				FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1,2,3,4
ORDER BY 5 DESC;
```
### Q4: We want to find out the most popular music Genre for each country. We determine the most popular genre as the genre with the highest amount of purchases. Write a query that returns each country along with the top Genre. For countries where the maximum number of purchases is shared return all Genres.
```sql
WITH popular_genre AS 
(
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id, 
	ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo 
    FROM invoice_line 
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2,3,4
	ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1
```
### Q5: Write a query that determines the customer that has spent the most on music for each country. Write a query that returns the country along with the top customer and how much they spent. For countries where the top amount spent is shared, provide all customers who spent this amount.
```sql
WITH Customter_with_country AS (
		SELECT customer.customer_id,first_name,last_name,billing_country,SUM(total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo 
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1,2,3,4
		ORDER BY 4 ASC,5 DESC)
SELECT * FROM Customter_with_country WHERE RowNo <= 1
```
---
## 🔍 Key Insights
1. The USA and European regions represent the largest share of total customer revenue.
2. Rock is the leading genre in terms of track volume and overall sales.
3. High-value customers demonstrate strong brand loyalty with repeat purchases within targeted genres.
