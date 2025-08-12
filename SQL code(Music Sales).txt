CREATE DATABASE MusicDB

USE MusicDB

SELECT * FROM invoice

--Q1: Find the most senior employee based on job title.
SELECT CONCAT(first_name,' ', last_name) AS EName, hire_date, title FROM Employee
ORDER BY levels 

--Q2: Determine which countries have the most invoices.
SELECT COUNT(invoice_id) Total_Invoice, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY Total_Invoice DESC

--Q3: Identify the top 3 invoice totals.
SELECT TOP 3 * FROM invoice
ORDER BY total DESC

--Q4: Find the city with the highest total invoice amount to determine the best location for
--a promotional event.
SELECT TOP 1 SUM(total) Total_Invoice, billing_city 
FROM invoice
GROUP BY billing_city
ORDER BY Total_Invoice DESC


--Q5: Identify the customer who has spent the most money.
SELECT TOP 1 
    C.first_name + ' ' + C.last_name AS Full_Name, 
    SUM(I.total) AS Total_Spent
FROM Invoice I
INNER JOIN Customer C ON C.customer_id = I.customer_id
GROUP BY C.first_name, C.last_name
ORDER BY Total_Spent DESC;


--Find the email, first name, and last name of customers who listen to Rock music.
SELECT DISTINCT CONCAT(C.first_name,' ', C.last_name) AS Full_Name, C.email, G.name AS Genre_Type
FROM customer C
INNER JOIN invoice I ON C.customer_id = I.customer_id
INNER JOIN invoice_line IL ON I.invoice_id = IL.invoice_id
INNER JOIN track T ON IL.track_id = T.track_id
INNER JOIN genre G ON T.genre_id = G.genre_id
WHERE G.genre_id = 1

SELECT * FROM invoice
SELECT * FROM track
SELECT * FROM invoice_line
SELECT * FROM artist
SELECT * FROM album
SELECT * FROM album2

--Identify the top 10 rock artists based on track count.
WITH CombinedAlbums AS (
    SELECT album_id, artist_id FROM Album
    UNION ALL
    SELECT album_id, artist_id FROM Album2
)
SELECT TOP 10 A.name AS Artist_Name, 
              COUNT(*) AS Total_Listens
FROM Invoice_Line IL
INNER JOIN Track T ON IL.track_id = T.track_id
INNER JOIN Genre G ON T.genre_id = G.genre_id
INNER JOIN CombinedAlbums CA ON T.album_id = CA.album_id
INNER JOIN Artist A ON CA.artist_id = A.artist_id
WHERE G.name = 'Rock'
GROUP BY A.name
ORDER BY Total_Listens DESC;


--Q3: Find all track names that are longer than the average track length.
SELECT DISTINCT name AS Song_name, milliseconds FROM track
WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
ORDER BY milliseconds


--Q1: Calculate how much each customer has spent on each artist.
WITH CombinedAlbums AS (
    SELECT album_id, artist_id FROM Album
    UNION ALL
    SELECT album_id, artist_id FROM Album2
)
SELECT C.customer_id, C.first_name + ' ' +C.last_name AS Full_Name, AR.name,
SUM(IL.unit_price * IL.quantity) AS Total_Spent
FROM customer C  
INNER JOIN invoice I ON C.customer_id = I.customer_id
INNER JOIN invoice_line IL  ON I.invoice_id = IL.invoice_id
INNER JOIN track TR ON IL.track_id = TR.track_id
INNER JOIN CombinedAlbums CA ON TR.album_id = CA.album_id
INNER JOIN artist AR ON CA.artist_id = AR.artist_id
GROUP BY 
    C.customer_id, 
    C.first_name, 
    C.last_name,
    AR.name
ORDER BY Total_Spent DESC
    
--Determine the most popular music genre for each country based on purchases.
WITH GenrePopularity AS (
    SELECT 
        C.country,
        G.name AS Genre,
        COUNT(*) AS Total_Purchases,
        RANK() OVER (
            PARTITION BY C.country 
            ORDER BY COUNT(*) DESC
        ) AS GenreRank
    FROM Invoice_Line IL
    INNER JOIN Invoice I ON IL.invoice_id = I.invoice_id
    INNER JOIN Customer C ON I.customer_id = C.customer_id
    INNER JOIN Track T ON IL.track_id = T.track_id
    INNER JOIN Genre G ON T.genre_id = G.genre_id
    GROUP BY C.country, G.name
)
SELECT country, Genre, Total_Purchases
FROM GenrePopularity
WHERE GenreRank = 1
ORDER BY Total_purchases DESC

--Identify the top-spending customer for each country.
-- Step 1: Build spending per customer per country with ranking
WITH CustomerSpending AS (
    SELECT 
        C.customer_id,
        C.first_name + ' ' + C.last_name AS Full_Name,
        C.country,
        SUM(IL.unit_price * IL.quantity) AS Total_Spent,
        ROW_NUMBER() OVER (
            PARTITION BY C.country 
            ORDER BY SUM(IL.unit_price * IL.quantity) DESC
        ) AS SpendRank
    FROM Customer C
    INNER JOIN Invoice I ON C.customer_id = I.customer_id
    INNER JOIN Invoice_Line IL ON I.invoice_id = IL.invoice_id
    GROUP BY C.customer_id, C.first_name, C.last_name, C.country
)

-- Step 2: Select only the top spender in each country
SELECT 
    customer_id,
    Full_Name,
    country,
    Total_Spent
FROM CustomerSpending
WHERE SpendRank = 1
ORDER BY country;
