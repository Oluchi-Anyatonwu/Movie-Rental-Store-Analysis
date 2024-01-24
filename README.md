# Movie-Rental-Store-Analysis

## Table of Contents
- [Project Overview](#project-overview)
- [Tools Used](#tools-used)
- [Expected Deliverables](#expected-deliverables)
- [Benefits](benefits#)
- [Data Cleaning/Preparation](#data-cleaning/preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](data-analysis#)
- [Results/Findings](#results/findings)
- [Recommendations](#recommendations)

### Project Overview:
This portfolio project aims to analyze a movie rental database through SQL queries and present key insights through a comprehensive Power BI report. The analysis covers various business questions to aid decision-making and improve overall business performance.

![movie rental 1](https://github.com/Oluchi-Anyatonwu/Movie-Rental-Store-Analysis/assets/61971994/52d55d14-9339-447c-821c-b604b7295c76)
![movie rental 2](https://github.com/Oluchi-Anyatonwu/Movie-Rental-Store-Analysis/assets/61971994/5474aa59-a350-4c40-9887-fdf10bd7b39a)
![movie rental 3](https://github.com/Oluchi-Anyatonwu/Movie-Rental-Store-Analysis/assets/61971994/ed1733a0-1c29-49c7-95d5-0d79e1f255d3)

https://github.com/Oluchi-Anyatonwu/Movie-Rental-Store-Analysis/assets/61971994/e8f32ec1-b0cb-4c6c-ab0d-90ec745e78f9



### Tools Used:
- SQL for data extraction, transformation, and analysis.
- Power BI for creating an interactive and visually appealing report.

### Expected Deliverables
- SQL queries and scripts for data analysis.
- Power BI report showcasing key insights and visualizations.
- Documentation summarizing findings and recommendations based on the analysis.

### Benefits
This project will provide actionable insights into customer behavior, rental trends, and revenue generation, enabling strategic decision-making for the business. The Power BI report will serve as a valuable tool for quick and intuitive access to key performance indicators.

### Data Cleaning/Preparation
#### 1. Data Inspection:
The first step in preparing the data involved a thorough inspection of the dataset to identify missing values, outliers, and inconsistencies. Understanding the structure and content of each table was crucial for subsequent cleaning steps.

#### 2. Handling Missing Values:
Missing values were addressed by either removing rows with missing data, imputing values based on relevant metrics, or using default values where appropriate. This ensured that the data used for analysis was complete and reliable.

#### 3. Dealing with Duplicates:
Duplicate records were identified and removed to maintain data integrity. This step was critical, especially in customer-related queries, to prevent skewing results due to redundant entries.

#### 4. Data Type Standardization:
Data types across different columns were standardized to ensure consistency. This involved converting date/time fields to a uniform format, ensuring numerical values were of the correct type, and aligning data types across related tables for accurate joins.

#### 5. Handling Outliers:
Outliers that could distort analysis results were identified and either corrected or flagged for special consideration. This step was particularly important in scenarios where extreme values could impact metrics like revenue or rental frequency.

#### 6. Addressing Inconsistencies:
Inconsistencies in naming conventions, such as variations in movie titles or customer names, were standardized. This involved cleaning up text data, correcting spelling errors, and aligning categories to a common naming convention.

#### 7. Merging and Joining Tables:
To answer complex business questions, multiple tables were often involved. The relevant tables were joined or merged using appropriate keys to create a unified dataset, facilitating seamless analysis.

#### 8. Feature Engineering:
New features were created to enhance analysis. For example, calculating revenue per movie or identifying active users required the creation of derived metrics to provide deeper insights into customer behavior and business performance.

### Exploratory Data Analysis
- EDA involved exploring the data to answer key questions such as:
- Identify and retrieve the First Name, Last Name, and Email Address of customers from Store 2.
- Identify movies with a rental rate of $0.99.
- Explore the distribution of rental rates and determine the count of movies in each rate category.
- Identify the movie rating with the highest number of films.
- Determine the prevalent movie rating in each store.
- List films by Film Name, Category, and Language.
- Calculate the number of times each movie has been rented out.
- Analyze revenue per movie.
- Identify the most spending customer for potential rewards or loyalty points.
- Determine the store that historically brought the most revenue.
- Analyze revenue per month.
- Determine the number of distinct renters per month.
- Identify the number of distinct films rented each month.
- Analyze the number of rentals in the Comedy, Sports, and Family genres.
- Identify active users (where active = 1).
- List all reward users with their phone numbers.

### Data Analysis
-- First, Last name and Email address of customers from Store 2
```
SELECT first_name, last_name, email
FROM customer
WHERE store_id = 2;
```

-- movie with rental rate of 0.99$
```
SELECT *
FROM film
WHERE rental_rate = 0.99;
```

-- rental rate and how many movies are in each rental rate categories
```
SELECT rental_rate, COUNT(*) as movie_count
FROM film
GROUP BY rental_rate;
```

-- Which rating do we have the most films in?
```
SELECT rating, COUNT(*) as movie_count
FROM film
GROUP BY rating
ORDER BY movie_count DESC
LIMIT 1;
```

-- Which rating is most prevalant in each store?
```
WITH RankedFilms AS (
    SELECT
        s.store_id,
        f.rating,
        COUNT(*) as movie_count,
        ROW_NUMBER() OVER (PARTITION BY s.store_id ORDER BY COUNT(*) DESC) as rn
    FROM
        film f
        JOIN inventory i ON f.film_id = i.film_id
        JOIN rental r ON i.inventory_id = r.inventory_id
        JOIN store s ON s.store_id = s.store_id
    GROUP BY s.store_id, f.rating
)
SELECT store_id, rating, movie_count
FROM RankedFilms
WHERE rn = 1;
```

-- List of films by Film Name, Category, Language
```
SELECT
    f.title AS FilmName,
    c.name AS Category,
    l.name AS Language
FROM
    film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    JOIN language l ON f.language_id = l.language_id;
```    
    
-- How many times each movie has been rented out?
```
SELECT
    film.film_id,
    film.title AS FilmName,
    COUNT(rental.rental_id) AS RentalCount
FROM
    film
    LEFT JOIN inventory ON film.film_id = inventory.film_id
    LEFT JOIN rental ON inventory.inventory_id = rental.inventory_id
GROUP BY
    film.film_id, film.title
ORDER BY
    RentalCount DESC;
```    
    
-- Revenue per Movie
```
SELECT
    f.film_id,
    f.title AS FilmName,
    COUNT(r.rental_id) AS RentalCount,
    f.rental_rate,
    COUNT(r.rental_id) * f.rental_rate AS Revenue
FROM
    film f
    LEFT JOIN inventory i ON f.film_id = i.film_id
    LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY
    f.film_id, f.title, f.rental_rate
ORDER BY
    Revenue DESC;
```

-- Most Spending Customer so that we can send him/her rewards or debate points
```
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS CustomerName,
    SUM(p.amount) AS TotalSpent
FROM
    customer c
    JOIN payment p ON c.customer_id = p.customer_id
GROUP BY
    c.customer_id, CustomerName
ORDER BY
    TotalSpent DESC
LIMIT 1;
```

-- What Store has historically brought the most revenue
```
SELECT
    s.store_id,
    SUM(p.amount) AS TotalRevenue
FROM
    store s
    JOIN staff st ON s.store_id = st.store_id
    JOIN payment p ON st.staff_id = p.staff_id
GROUP BY
    s.store_id
ORDER BY
    TotalRevenue DESC
LIMIT 1;
```

-- How many rentals we have for each month
```
SELECT
    YEAR(rental_date) AS rental_year,
    MONTH(rental_date) AS rental_month,
    COUNT(rental_id) AS rental_count
FROM
    rental
GROUP BY
    rental_year, rental_month
ORDER BY
    rental_year, rental_month;
```    
    
-- Rentals per Month (such Jan => How much, etc)
```
SELECT
    MONTHNAME(rental_date) AS rental_month,
    COUNT(rental_id) AS rental_count
FROM
    rental
GROUP BY
    MONTHNAME(rental_date)
ORDER BY
    MIN(rental_date);
```    
    
-- Which date first movie was rented out ?
```
SELECT MIN(rental_date) AS first_rental_date
FROM rental;
```

-- Which date last movie was rented out
```
SELECT MAX(rental_date) AS last_rental_date
FROM rental;
```

-- For each movie, when was the first time and last time it was rented out?
```
SELECT
    f.title AS FilmName,
    MIN(r.rental_date) AS FirstRentalDate,
    MAX(r.rental_date) AS LastRentalDate
FROM
    film f
    JOIN inventory i ON f.film_id = i.film_id
    JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY
    f.title
ORDER BY
    FilmName;
```
   
-- Last Rental Date of every customer0
```
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS CustomerName,
    MAX(r.rental_date) AS LastRentalDate
FROM
    customer c
    JOIN rental r ON c.customer_id = r.customer_id
GROUP BY
    c.customer_id, CustomerName
ORDER BY
    CustomerName;
```    
    
-- Revenue Per Month
```
SELECT
    DATE_FORMAT(payment_date, '%Y-%m') AS payment_month,
    SUM(amount) AS monthly_revenue
FROM
    payment
GROUP BY
    payment_month
ORDER BY
    payment_month;
```

-- How many distint Renters per month
```
SELECT
    DATE_FORMAT(rental_date, '%Y-%m') AS rental_month,
    COUNT(DISTINCT customer_id) AS distinct_renters
FROM
    rental
GROUP BY
    rental_month
ORDER BY
    rental_month;
```    
    
-- Number of Distinct Film Rented Each Month
```
SELECT
    DATE_FORMAT(rental.rental_date, '%Y-%m') AS rental_month,
    COUNT(DISTINCT inventory.film_id) AS distinct_rented_films
FROM
    rental
    JOIN inventory ON rental.inventory_id = inventory.inventory_id
GROUP BY
    rental_month
ORDER BY
    rental_month;
```    
    
-- Number of Rentals in Comedy , Sports and Family
```
SELECT
    c.name AS category,
    COUNT(r.rental_id) AS rental_count
FROM
    rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film_category fc ON i.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
WHERE
    c.name IN ('Comedy', 'Sports', 'Family')
GROUP BY
    category;
```
    
-- Users who have been rented at least 3 times
```
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(r.rental_id) AS rental_count
FROM
    customer c
    JOIN rental r ON c.customer_id = r.customer_id
GROUP BY
    c.customer_id, customer_name
HAVING
    rental_count >= 3
ORDER BY
    rental_count DESC;
```    
    
-- How much revenue has one single store (store 1) made over PG13 and R rated films
```
SELECT
    s.store_id,
    SUM(p.amount) AS total_revenue
FROM
    store s
    JOIN staff st ON s.store_id = st.store_id
    JOIN payment p ON st.staff_id = p.staff_id
    JOIN rental r ON p.rental_id = r.rental_id
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
WHERE
    f.rating IN ('PG-13', 'R')
    AND s.store_id = 1 
GROUP BY
    s.store_id;
```    
    
-- Active User  where active = 1
```
SELECT
    customer_id,
    CONCAT(first_name, ' ', last_name) AS customer_name,
    active
FROM
    customer
WHERE
    active = 1;
```    
    
-- Reward Users : who has rented at least 30 times
```
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(r.rental_id) AS rental_count
FROM
    customer c
    JOIN rental r ON c.customer_id = r.customer_id
GROUP BY
    c.customer_id, customer_name
HAVING
    rental_count >= 30;
```    
    
-- Reward Users who are also active
```
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    COUNT(r.rental_id) AS rental_count
FROM
    customer c
    JOIN rental r ON c.customer_id = r.customer_id
WHERE
    c.active = 1
    GROUP BY
    c.customer_id, customer_name;
```    
    
-- Users with Phone
```
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    a.phone
FROM
    customer c
    JOIN address a ON c.address_id = a.address_id
WHERE
    a.phone IS NOT NULL AND a.phone <> '';
```

### Results/Findings
#### Insights:
- Eleanor is the top movie renter, closely followed by Karl Seal, Clara Shaw, Marcia Dean, and Tammy Sanders, contributing to a total of 16,044 movie rentals.
- Average revenue surged from May to July 2005, followed by a significant 89.34% decline from August 2005 to February 2006, indicating potential opportunities for targeted promotions.

#### Category Analysis:
- Sports led with 1,179 rentals, surpassing others by 42.05%. Despite its popularity, Sports accounted for 7.35% of the total rental count.
- Across 16 film categories, rental counts ranged from 830 to 1,179, reflecting diverse customer preferences.

#### Staff Performance:
- Staff 1 recorded 8,040 rentals, while Staff 2 recorded 8,004. Staff 1 generated higher revenue at $33,487.47, compared to Staff 2's $33,927.94.

### Recommendations:
#### Targeted Promotions:
Implement targeted promotions during historically low-revenue months for increased customer engagement.

#### Diversify Category Promotions:
Promote films from underexplored categories to diversify rental patterns and potentially boost revenue.

#### Staff Training and Recognition:
Share best practices between staff members and consider training or incentives to enhance overall performance.
