```

--Part 4: Starting With Data

--Question 1: Percentage of people who look at the site who actually make a purchase:

WITH TransactionCounts AS (
    SELECT
        SUM(CASE WHEN "transactions" > 0 THEN 1 ELSE 0 END) AS Sessions_With_Transaction,
        SUM(CASE WHEN "transactions" = 0 THEN 1 ELSE 0 END) AS Sessions_Without_Transaction,
        COUNT(*) AS Total_Sessions
    FROM all_sessions
)
SELECT
    Sessions_With_Transaction,
    Sessions_Without_Transaction,
    Total_Sessions,
    ROUND((Sessions_With_Transaction::NUMERIC / Total_Sessions) * 100, 2) AS Percent_With_Transaction,
    ROUND((Sessions_Without_Transaction::NUMERIC / Total_Sessions) * 100, 2) AS Percent_Without_Transaction
FROM TransactionCounts;

--Question 2: How does this vary by whether or not someone is a Californian, a non-Cal American, or
--from a different country?

WITH VisitorCategories AS (
    SELECT
        CASE
            WHEN country = 'United States' AND "usa_iscal" = 'Yes' THEN 'California'
            WHEN country = 'United States' AND ("usa_iscal" = 'No' OR "usa_iscal" IS NULL) THEN 'Rest of USA'
            ELSE 'Other Countries'
        END AS Visitor_Category,
        (CASE WHEN "transactions" > 0 THEN 1 ELSE 0 END) AS Made_Purchase
    FROM all_sessions
),
PurchaseRates AS (
    SELECT
        Visitor_Category,
        COUNT(*) AS Total_Visitors,
        SUM(Made_Purchase) AS Purchasers,
        ROUND((SUM(Made_Purchase)::NUMERIC / COUNT(*) * 100), 2) AS Purchase_Percentage
    FROM VisitorCategories
    GROUP BY Visitor_Category
)
SELECT * FROM PurchaseRates
ORDER BY Purchase_Percentage DESC;

--Question 3: What colors of T-shirts are in stock? How many unique products are available in each
--color? And what is the stock level of each color?

CREATE TEMP TABLE products_tees AS
SELECT 
    *,
    CASE
        WHEN "name" ILIKE '%white%' THEN 'White'
        WHEN "name" ILIKE '%black%' THEN 'Black'
        WHEN "name" ILIKE '%charcoal%' THEN 'Charcoal'
        WHEN "name" ILIKE '%grey%' OR "name" ILIKE '%pewter%' THEN 'Grey'
        WHEN "name" ILIKE '%green%' OR "name" ILIKE '%olive%' OR "name" ILIKE '%sage%' THEN 'Green'
        WHEN "name" ILIKE '%red%' THEN 'Red'
        WHEN "name" ILIKE '%pink%' THEN 'Pink'
        WHEN "name" ILIKE '%blue%' OR "name" ILIKE '%navy%' OR "name" ILIKE '%aqua%' THEN 'Blue'
        WHEN "name" ILIKE '%yellow%' THEN 'Yellow'
        WHEN "name" ILIKE '%purple%' OR "name" ILIKE '%lavender%' THEN 'Purple'
        ELSE 'Other'
    END AS color
FROM products
WHERE ("name" ILIKE '%tee shirt%' OR
       "name" ILIKE '%tee-shirt%' OR
       "name" ILIKE '%t-shirt%' OR
       "name" ILIKE '% t shirt%' OR
       "name" ILIKE '% t-shirt%' OR
       "name" ILIKE '%tee%')
  AND "name" NOT ILIKE '%thermostat%'
  AND "name" NOT ILIKE '%bottle%'
  AND "name" NOT ILIKE '%stainless steel%';

UPDATE products_tees
SET color = 'Other'
WHERE color IS NULL;

SELECT * FROM products_tees

WITH ColorCounts AS (
    SELECT color, COUNT(*) AS number_of_products
    FROM products_tees
    GROUP BY color
),
TotalCount AS (
    SELECT COUNT(*) AS total_count
    FROM products_tees
)
SELECT 
    cc.color,
    cc.number_of_products,
    ROUND((cc.number_of_products::NUMERIC / tc.total_count) * 100, 2) AS color_percentage
FROM ColorCounts cc, TotalCount tc
ORDER BY cc.number_of_products DESC;

--Question 4: How many products are marketed to men versus women?

WITH Counts AS (
    SELECT
        SUM(CASE WHEN "name" ILIKE '%women%' THEN 1 ELSE 0 END) AS women_count,
        SUM(CASE WHEN "name" ILIKE '%men%' AND "name" NOT ILIKE '%women%' THEN 1 ELSE 0 END) AS men_count,
        SUM(CASE WHEN "name" ILIKE '%boy%' THEN 1 ELSE 0 END) AS boy_count,
        SUM(CASE WHEN "name" ILIKE '%girl%' THEN 1 ELSE 0 END) AS girl_count,
        SUM(CASE
            WHEN "name" NOT ILIKE '%women%' AND
                 "name" NOT ILIKE '%men%' AND
                 "name" NOT ILIKE '%boy%' AND
                 "name" NOT ILIKE '%girl%' THEN 1
            ELSE 0
        END) AS non_gendered_count,
        COUNT(*) AS total_count
    FROM products
)
SELECT
    women_count,
    men_count,
    boy_count,
    girl_count,
    non_gendered_count,
    ROUND((women_count::NUMERIC / total_count) * 100, 2) AS women_percentage,
    ROUND((men_count::NUMERIC / total_count) * 100, 2) AS men_percentage,
    ROUND((boy_count::NUMERIC / total_count) * 100, 2) AS boy_percentage,
    ROUND((girl_count::NUMERIC / total_count) * 100, 2) AS girl_percentage,
    ROUND((non_gendered_count::NUMERIC / total_count) * 100, 2) AS non_gendered_percentage
FROM Counts;


	--sanity inspection

SELECT * FROM products
WHERE name ILIKE '%girl%'

SELECT * FROM products
WHERE name ILIKE '%boy%'

SELECT * FROM products
WHERE name ILIKE '%women%'

SELECT * FROM products
WHERE name ILIKE '%men%' AND "name" NOT ILIKE '%women%'  -- inspection passed

```


