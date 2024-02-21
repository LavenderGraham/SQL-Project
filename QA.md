```
--Checking my fullVisitorID and visitorID columns in all_sessions and analytics to check for values that
--do not have an appropriate number of digits

SELECT "fullVisitorId",  -- Check that all rows have an Id digit count between 16 and 19
       CHAR_LENGTH("fullVisitorId") AS Id_digit_count
FROM all_sessions
WHERE CHAR_LENGTH("fullVisitorId") NOT BETWEEN 15 AND 19
ORDER BY Id_digit_count;

SELECT "visitId",   -- Check that all rows have an Id digit count of 10
       CHAR_LENGTH("visitId") AS Id_digit_count
FROM all_sessions
WHERE CHAR_LENGTH("visitId") != 10
ORDER BY Id_digit_count;

SELECT "fullvisitorId",  -- Check that all rows have an Id digit count between 15 and 19
       CHAR_LENGTH("fullvisitorId") AS Id_digit_count
FROM analytics
WHERE CHAR_LENGTH("fullvisitorId") NOT BETWEEN 15 AND 19
ORDER BY Id_digit_count DESC

--Check for rows in my date columns that are before the year where data first started being collected

SELECT *  -- check for rows in all_sessions which are not between august 2016 and today
FROM all_sessions
WHERE "date" NOT BETWEEN '2016-08-01' AND CURRENT_DATE
ORDER BY "date";

SELECT *  -- check for rows in analytics which are not between may 2017 and today
FROM analytics
WHERE "date" NOT BETWEEN '2017-05-01' AND CURRENT_DATE
ORDER BY "date";

--Check that the SKUs have between 7 and 16 digits

SELECT "SKU"
FROM products
WHERE CHAR_LENGTH("SKU") < 7 OR CHAR_LENGTH("SKU") > 16;

SELECT "productSKU"
FROM sales_by_sku
WHERE CHAR_LENGTH("productSKU") < 7 OR CHAR_LENGTH("productSKU") > 16;

SELECT "productSKU"
FROM sales_report
WHERE CHAR_LENGTH("productSKU") < 7 OR CHAR_LENGTH("productSKU") > 16;


--We sell mostly inexpensive items so if an item costs more than $1000 or a transaction
--is more than $1000 it is likely to be a mistake. Let's check for these kinds of mistakes.
--A transaction or a product over 1000 is not necessarily a mistake but it could be.

SELECT "totalTransactionRevenue" -- Check for transactions over $1000
FROM all_sessions
WHERE "totalTransactionRevenue" > 1000;

SELECT "productPrice" -- Check for product price in all_sessions over $1000
FROM all_sessions
WHERE "productPrice" > 1000;

SELECT "unit_price" -- Check for product price in analytics over $1000
FROM analytics
WHERE "unit_price" > 1000;

--Make sure transactions in the all_sessions table is 0 or 1

SELECT "transactions"
FROM all_sessions
WHERE "transactions" NOT IN (0, 1);

--Check to see if anyone was on the site for longer than 12 hours. If so that could be
--a mistake. (However it is not necessarily a mistake. Check with the web development team
--to see how the number of second someone on the site was calculated. If just leaving a tab
--on in the background is registered as on the site then people could regularly be recorded
--as being on the site for longer than 12 hours)

SELECT * -- Checking all_sessions to see if people are on the site for longer than 12 hours
FROM all_sessions
WHERE "timeOnSite" >= 43200;

SELECT * -- Checking analytics to see if people are on the site for longer than 12 hours
FROM analytics
WHERE "timeonsite" >= 43200;

--Check that the number of page views in all_sessions is at most 50. If someone is recorded as 
--having more
--than 50 page views in one session that could possibly be a mistake.

SELECT * -- Checking all_sessions to find the number of page views
FROM all_sessions
WHERE "pageviews" >= 50;

--Check that numbers of products ordered or stocked are between 0 and 10,000
--Negative numbers are impossible and definitely mistakes. Numbers over 10,000
--are probably too high and therefore mistakes.

SELECT "SKU", "name", "stockLevel" -- Checking the products table
FROM products
WHERE "orderedQuantity" IS NOT NULL AND ("orderedQuantity" < 0 OR "orderedQuantity" > 10000);

		--Flagged. A Kick Ball and a 22 oz Water Bottle were recorded as having Order
		--quantities over 10,000. Check with the sales team to make sure this is correct.

SELECT "SKU", "name", "stockLevel" -- Checking the products table
FROM products
WHERE "stockLevel" IS NOT NULL AND ("stockLevel" < 0 OR "stockLevel" > 10000);

		--Flagged. Two different 22 oz Water Bottles were recorded as having stockLevels
		--over 15,000. Check with the sales team to make sure this is correct.

SELECT "productSKU", "total_ordered" -- Checking the sales_by_sku table
FROM sales_by_sku
WHERE "total_ordered" IS NOT NULL AND ("total_ordered" < 0 OR "total_ordered" > 10000);

SELECT "productSKU", "total_ordered" -- Checking the sales_report table
FROM sales_report
WHERE "total_ordered" IS NOT NULL AND ("total_ordered" < 0 OR "total_ordered" > 10000);

SELECT "productSKU", "stockLevel" -- Checking the sales_report table
FROM sales_report
WHERE "stockLevel" IS NOT NULL AND ("stockLevel" < 0 OR "stockLevel" > 10000);

--Check restocking lead time to make sure it is an integer between zero and fifty.
--Restocking lead time is by definition positive. But it should not take longer than
--fifty days.

SELECT "SKU", "restockingLeadTime"
FROM products
WHERE "restockingLeadTime" IS NULL OR "restockingLeadTime" < 0 OR "restockingLeadTime" > 50;

SELECT "productSKU", "restockingLeadTime"
FROM sales_report
WHERE "restockingLeadTime" IS NULL OR "restockingLeadTime" < 0 OR "restockingLeadTime" > 50;


--Check sentiment score in products and sales_report and make sure all values are between -1 and 1
--and have one decimal place.

SELECT "SKU", "sentimentScore"
FROM products
WHERE "sentimentScore" < -1 OR "sentimentScore" > 1
OR CAST("sentimentScore" AS NUMERIC(10,1)) != "sentimentScore";

SELECT "productSKU", "sentimentScore"
FROM sales_report
WHERE "sentimentScore" < -1 OR "sentimentScore" > 1
OR CAST("sentimentScore" AS NUMERIC(10,1)) != "sentimentScore";

--Check sentiment magnitude in products and sales_report and make sure all values are between 0 and 2.

SELECT "SKU", "sentimentMagnitude"
FROM products
WHERE "sentimentMagnitude" < 0 OR "sentimentMagnitude" > 2
OR CAST("sentimentMagnitude" AS NUMERIC(10,1)) != "sentimentMagnitude";

SELECT "productSKU", "sentimentMagnitude"
FROM sales_report
WHERE "sentimentMagnitude" < 0 OR "sentimentMagnitude" > 2
OR CAST("sentimentMagnitude" AS NUMERIC(10,1)) != "sentimentMagnitude";
```
