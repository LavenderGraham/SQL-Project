```

--Viewing tables
SELECT * FROM all_sessions
LIMIT 10

SELECT * FROM analytics
LIMIT 10

SELECT * FROM products

SELECT * FROM sales_by_sku

SELECT * FROM sales_report



----- This Function drops any column that is all nulls
DO $$
DECLARE
  r RECORD;
  total_count bigint;
  column_count bigint;
BEGIN
  -- Get the total count of rows in the table
  EXECUTE 'SELECT COUNT(*) FROM all_sessions' INTO total_count;

  FOR r IN SELECT column_name FROM information_schema.columns WHERE table_name = 'all_sessions'
  LOOP
    -- Count non-null entries for each column
    EXECUTE format('SELECT COUNT(%I) FROM all_sessions', r.column_name) INTO column_count;

    -- Compare with total_count
    IF column_count = 0 THEN
      RAISE NOTICE 'Column % has only NULL values', r.column_name;
    END IF;
  END LOOP;
END $$;
---

---This function figures out what percentage of columns are null
CREATE OR REPLACE FUNCTION get_null_percentage_all_columns(tablename TEXT, schemaname TEXT DEFAULT 'public')
RETURNS TABLE(column_name_out TEXT, null_percentage_out NUMERIC) AS $$
DECLARE
  col_name TEXT;
  total_count BIGINT;
  null_count BIGINT;
BEGIN
  -- Count total number of rows in the table
  EXECUTE format('SELECT COUNT(*) FROM %I.%I', schemaname, tablename) INTO total_count;

  FOR col_name IN SELECT column_name FROM information_schema.columns WHERE table_name = tablename AND table_schema = schemaname
  LOOP
    -- Count number of NULL values in the current column
    EXECUTE format('SELECT COUNT(*) FROM %I.%I WHERE %I IS NULL', schemaname, tablename, col_name) INTO null_count;

    -- Calculate the percentage of NULL values
    column_name_out := col_name; -- Set the output column name
    null_percentage_out := (null_count::NUMERIC / total_count) * 100; -- Set the output null percentage

    RETURN NEXT;
  END LOOP;
END;
$$ LANGUAGE plpgsql;
---

--This function figures out columns with only one value
CREATE OR REPLACE FUNCTION find_single_value_columns(tablename TEXT, schemaname TEXT DEFAULT 'public')
RETURNS TABLE(column_name_out TEXT, unique_value_out TEXT) AS $$
DECLARE
  col_name TEXT;
  value_count BIGINT;
  unique_value_text TEXT;
BEGIN
  -- Retrieve each column name from the specified table and schema
  FOR col_name IN SELECT column_name FROM information_schema.columns WHERE table_name = tablename AND table_schema = schemaname
  LOOP
    -- Execute a dynamic SQL to count the distinct values in the current column
    EXECUTE format('SELECT COUNT(DISTINCT %I) FROM %I.%I', col_name, schemaname, tablename) INTO value_count;
    
    -- If there is only one distinct value, retrieve that value
    IF value_count = 1 THEN
      EXECUTE format('SELECT %I FROM %I.%I LIMIT 1', col_name, schemaname, tablename) INTO unique_value_text;
      
      -- Return the column name and its unique value
      column_name_out := col_name; -- Use modified output variable name
      unique_value_out := unique_value_text; -- Use modified output variable name
      RETURN NEXT;
    END IF;
  END LOOP;
END;
$$ LANGUAGE plpgsql;
--
--Examining columns for all NULLS

SELECT * FROM get_null_percentage_all_columns('all_sessions');


SELECT * FROM get_null_percentage_all_columns('analytics');

SELECT * FROM get_null_percentage_all_columns('products');

SELECT * FROM get_null_percentage_all_columns('sales_by_sku');

SELECT * FROM get_null_percentage_all_columns('sales_report');

--Dropping columns cuz they're all NULL or have only one value

SELECT * FROM all_sessions

ALTER TABLE all_sessions -- All Null
  DROP COLUMN "productRefundAmount",
  DROP COLUMN "itemQuantity",
  DROP COLUMN "itemRevenue",
  DROP COLUMN "searchKeyword";

ALTER TABLE all_sessions -- Only One
	DROP COLUMN "currencyCode"

ALTER TABLE analytics -- All Null
	DROP COLUMN "bounces"

ALTER TABLE analytics -- Only One
	DROP COLUMN "socialEngagementType"
  
--

--Examining columns for only one value
SELECT * FROM find_single_value_columns('all_sessions')
	--based on the output here i will delete currencyCode as its only value is USD

SELECT * FROM find_single_value_columns('analytics')
	--based on the output here i will delete socialEngagementType as its only value is Not Socially Engaged (also bounces is all null)

SELECT * FROM find_single_value_columns('products')
	--not going to exclude anything based on output here

SELECT * FROM find_single_value_columns('sales_by_sku')
	--not going to exclude anything based on output here

SELECT * FROM find_single_value_columns('sales_report')
	--not going to exclude anything based on output here

--type cast to numeric
ALTER TABLE analytics
ALTER COLUMN unit_price TYPE NUMERIC USING unit_price::NUMERIC;

ALTER TABLE analytics
ALTER COLUMN unit_price_original TYPE NUMERIC USING unit_price_original::NUMERIC;

ALTER TABLE all_sessions
ALTER COLUMN "productPrice" TYPE NUMERIC USING "productPrice"::NUMERIC;

ALTER TABLE all_sessions
ALTER COLUMN "totalTransactionRevenue" TYPE NUMERIC USING "totalTransactionRevenue"::NUMERIC;

ALTER TABLE all_sessions
ALTER COLUMN "transactions" TYPE NUMERIC USING "transactions"::NUMERIC;

ALTER TABLE all_sessions
ALTER COLUMN "productQuantity" TYPE NUMERIC USING "productQuantity"::NUMERIC;

--Getting rid of excess zeros in columns like unit_price

UPDATE analytics
SET analytics = unit_price / 1000000;

UPDATE analytics
SET unit_price_original = unit_price * 1000000;

UPDATE all_sessions
SET "productPrice" = "productPrice" / 1000000

UPDATE all_sessions
SET "totalTransactionRevenue" = "totalTransactionRevenue" / 1000000

--Okay lets look for columns that are mostly null or set to mostly zero or mostly set to one

SELECT "totalTransactionRevenue", "transactions", "productQuantity" FROM all_sessions
WHERE "totalTransactionRevenue" is NOT NULL or "transactions" is NOT NULL or "productQuantity" is NOT NULL

SELECT "totalTransactionRevenue", "transactions", "productQuantity" FROM all_sessions
WHERE "totalTransactionRevenue" > 0 or "transactions" > 0

SELECT "eCommerceAction_type" FROM all_sessions -- turns out eCommerceAction_type has 349 values not set to zero
WHERE "eCommerceAction_type" != '0'

SELECT "eCommerceAction_step" FROM all_sessions -- turns out eCommerceAction_step has 18 values not set to zero
WHERE "eCommerceAction_step" != '1'

SELECT "eCommerceAction_option" FROM all_sessions -- turns out 31 eCommerceAction_option has 31 values not set to zero
WHERE "eCommerceAction_option" IS NOT NULL

SELECT "pagePathLevel1" FROM all_sessions -- turns out there are 816 other rows in this column
WHERE "pagePathLevel1" != '/google+redesign/'

SELECT COUNT(DISTINCT "pagePathLevel1") FROM all_sessions -- with ten different options
WHERE "pagePathLevel1" != '/google+redesign/'

SELECT "transactionId" FROM all_sessions -- nine rows have entries
WHERE "transactionId" IS NOT NULL

SELECT "transactionRevenue" FROM all_sessions -- four rows have entries
WHERE "transactionRevenue" IS NOT NULL

SELECT "productVariant" FROM all_sessions -- 40 rows have entries
WHERE "productVariant" != '(not set)'

SELECT "productRevenue" FROM all_sessions -- four rows have entries
WHERE "productRevenue" IS NOT NULL

SELECT "productQuantity" FROM all_sessions -- 53 rows have entries
WHERE "productQuantity" IS NOT NULL

SELECT "type" FROM all_sessions -- 192 rows have entries
WHERE "type" != 'PAGE'

SELECT COUNT(DISTINCT "type") FROM all_sessions -- with two different options. PAGE and EVENT

SELECT "sessionQualityDim" FROM all_sessions --  1228 rows have entries
WHERE "sessionQualityDim" IS NOT NULL

SELECT COUNT(DISTINCT "sessionQualityDim") FROM all_sessions -- with 44 different options

--add missing zeros to totalTransactionRevenue and transactions

BEGIN;
UPDATE all_sessions
SET 
  "totalTransactionRevenue" = COALESCE("totalTransactionRevenue", '0'),
  transactions = COALESCE(transactions, '0');
COMMIT;
ROLLBACK;

--Removing or updating rows where Country and City are incompatible or incomplete
SELECT *
FROM all_sessions
WHERE country = 'Canada' AND city = 'New York';

DELETE FROM all_sessions
WHERE country = 'Canada' AND city = 'New York';

UPDATE all_sessions
SET "city" = 'Singapore'
WHERE "city" = '(not set)'
AND "country" = 'Singapore'

--Examine Not Set
SELECT "v2ProductCategory", "v2ProductName", "city", "country", "usa_iscal", "totalTransactionRevenue", "transactions"
FROM all_sessions
WHERE "v2ProductCategory" = '(not set)' AND "transactions" > 0

--Filling in Missing Product Categories where it's obvious what they are

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Bags/Backpacks/'
WHERE "v2ProductCategory" = '(not set)'
AND "v2ProductName" ILIKE '%Backpack%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Bags/'
WHERE "v2ProductCategory" = '(not set)'
AND "v2ProductName" ILIKE '% Bag%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/Men''s/Men''s-T-Shirts/'
WHERE "v2ProductCategory" = '(not set)'
AND "v2ProductName" ILIKE '%Men''s Vintage Tee%' OR "v2ProductName" ILIKE '%Men''s Short Sleeve Badge Tee Charcoal'

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/Men''s/Men''s-Outerwear'
WHERE "v2ProductCategory" = '(not set)'
AND ("v2ProductName" ILIKE '%Men''s Zip Hoodie%' OR "v2ProductName" ILIKE '%Men''s  Zip Hoodie%')

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/Men''s/Men''s-Outerwear'
WHERE ("v2ProductCategory" = '(not set)' OR "v2ProductCategory" = 'Home/Apparel/Men''s/')
AND ("v2ProductName" ILIKE '%Men''s Zip Hoodie%' OR "v2ProductName" ILIKE '%Men''s  Zip Hoodie%')

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/Headgear'
WHERE ("v2ProductCategory" = 'Headgear' OR "v2ProductCategory" = '${escCatTitle}')
AND "v2ProductName" ILIKE '%Heather Cap%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/'
WHERE "v2ProductCategory" = '${escCatTitle}'
AND "v2ProductName" ILIKE '%Socks%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/'
WHERE "v2ProductCategory" = '${escCatTitle}'
AND "v2ProductName" ILIKE '%Sunglasses%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Office'
WHERE "v2ProductCategory" = '${escCatTitle}'
AND "v2ProductName" ILIKE '%Stickers%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Office'
WHERE "v2ProductCategory" = '${escCatTitle}'
AND "v2ProductName" ILIKE '%Pen%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Office'
WHERE "v2ProductCategory" = '(not set)'
AND "v2ProductName" ILIKE '%Notebook%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Nest'
WHERE ("v2ProductCategory" = '(not set)' OR "v2ProductCategory" = '${escCatTitle}')
AND "v2ProductName" ILIKE '%Nest%';

UPDATE all_sessions
SET "v2ProductCategory" = 'Home/Apparel/Women''s/Women''s-Outerwear/'
WHERE "v2ProductCategory" = '${escCatTitle}'
AND "v2ProductName" ILIKE '%Women''s Convertible Vest-Jacket%'

		--quick inspect
SELECT "v2ProductCategory", "Big Brands", "v2ProductName" FROM all_sessions
WHERE "productQuantity" > 0

--INSPECTION: Filtering all_sesssions for transactions (prints 80 rows)
SELECT "totalTransactionRevenue" FROM all_sessions
WHERE "totalTransactionRevenue" > 0

SELECT "transactions" FROM all_sessions
WHERE "transactions" > 0

SELECT * FROM all_sessions
WHERE "transactions" > 0

SELECT "totalTransactionRevenue", "transactions", "productQuantity" FROM all_sessions
WHERE "totalTransactionRevenue" > 0

SELECT "productQuantity" FROM all_sessions
WHERE "productQuantity" IS NOT NULL

--Inspecting product names and catgories for mismatch

SELECT "v2ProductCategory", "v2ProductName" FROM all_sessions
WHERE "v2ProductName" ILIKE '%backpack%' or "v2ProductName" ILIKE '%bag%'

SELECT "v2ProductCategory", "v2ProductName" FROM all_sessions
WHERE "productQuantity" > 0

SELECT "v2ProductCategory", "v2ProductName" FROM all_sessions
WHERE "productQuantity" > 0
AND "v2ProductName" ILIKE '%Notebook%' -- malleable line

--Examining product categories

SELECT COUNT(DISTINCT "v2ProductCategory") 
FROM all_sessions

SELECT "v2ProductCategory", "city", "country", "usa_iscal", "totalTransactionRevenue", "transactions"
FROM all_sessions
WHERE "v2ProductCategory" IS NOT NULL AND "transactions" > 0

--Counting Transactions per country

SELECT country, SUM("transactions")
FROM all_sessions
WHERE "transactions" = 1
GROUP BY country;

--Counting Number of cities per country

SELECT COUNT(DISTINCT city), country, SUM("transactions")
FROM all_sessions
GROUP BY country
ORDER BY SUM("transactions") DESC

--Making a column to specify whether or not it's in California

ALTER TABLE all_sessions
ADD COLUMN usa_iscal TEXT;

UPDATE all_sessions
SET usa_iscal = CASE
    WHEN country != 'United States' THEN NULL
    WHEN country = 'United States' AND city IN ('Los Angeles', 'San Francisco', 'San Jose', 'Palo Alto', 'San Bruno', 'Mountain View', 'Sunnyvale') THEN 'Yes'
    WHEN country = 'United States' AND city = 'not available in demo dataset' THEN 'Unknown'
    WHEN country = 'United States' THEN 'No'
    END;
	
--Adding my Big Brands column
ALTER TABLE all_sessions
ADD COLUMN "Big Brands" TEXT;

UPDATE all_sessions
SET "Big Brands" = CASE
    WHEN "v2ProductName" ILIKE '%Netflix%' THEN 'Netflix'
    WHEN "v2ProductName" ILIKE '%Nest%' OR "v2ProductName" ILIKE '%Google%' THEN 'Google'
    ELSE NULL
END;

--Remove excess zeros

UPDATE all_sessions
SET 
    "totalTransactionRevenue" = ROUND(CAST("totalTransactionRevenue" AS NUMERIC), 2),
    "productPrice" = ROUND(CAST("productPrice" AS NUMERIC), 2);

--Changing Text Columns to Numeric or Date when needed

ALTER TABLE all_sessions
  ALTER COLUMN "timeOnSite" TYPE NUMERIC USING "timeOnSite"::NUMERIC,
  ALTER COLUMN "pageviews" TYPE NUMERIC USING "pageviews"::NUMERIC,
  ALTER COLUMN "productRevenue" TYPE NUMERIC USING "productRevenue"::NUMERIC,
  ALTER COLUMN "transactionRevenue" TYPE NUMERIC USING "transactionRevenue"::NUMERIC;

ALTER TABLE all_sessions
ALTER COLUMN "date" TYPE DATE USING "date"::DATE;

ALTER TABLE analytics
ALTER COLUMN "date" TYPE DATE USING "date"::DATE;

ALTER TABLE analytics
  ALTER COLUMN "units_sold" TYPE NUMERIC USING "units_sold"::NUMERIC,
  ALTER COLUMN "pageviews" TYPE NUMERIC USING "pageviews"::NUMERIC,
  ALTER COLUMN "timeonsite" TYPE NUMERIC USING "timeonsite"::NUMERIC,
  ALTER COLUMN "revenue" TYPE NUMERIC USING "revenue"::NUMERIC;

ALTER TABLE products
  ALTER COLUMN "orderedQuantity" TYPE NUMERIC USING "orderedQuantity"::NUMERIC,
  ALTER COLUMN "stockLevel" TYPE NUMERIC USING "stockLevel"::NUMERIC,
  ALTER COLUMN "restockingLeadTime" TYPE NUMERIC USING "restockingLeadTime"::NUMERIC,
  ALTER COLUMN "sentimentScore" TYPE NUMERIC USING "sentimentScore"::NUMERIC,
  ALTER COLUMN "sentimentMagnitude" TYPE NUMERIC USING "sentimentMagnitude"::NUMERIC;

ALTER TABLE sales_by_sku
  ALTER COLUMN "total_ordered" TYPE NUMERIC USING "total_ordered"::NUMERIC;

ALTER TABLE sales_report
  ALTER COLUMN "total_ordered" TYPE NUMERIC USING "total_ordered"::NUMERIC,
  ALTER COLUMN "stockLevel" TYPE NUMERIC USING "stockLevel"::NUMERIC,
  ALTER COLUMN "restockingLeadTime" TYPE NUMERIC USING "restockingLeadTime"::NUMERIC,
  ALTER COLUMN "sentimentScore" TYPE NUMERIC USING "sentimentScore"::NUMERIC,
  ALTER COLUMN "sentimentMagnitude" TYPE NUMERIC USING "sentimentMagnitude"::NUMERIC,
  ALTER COLUMN "ratio" TYPE NUMERIC USING "ratio"::NUMERIC;
```

