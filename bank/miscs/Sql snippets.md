
### Filter stored procedures Per execution count

SELECT 
    p.name, 
    c.execution_count, 
    p.object_id
FROM sys.dm_exec_procedure_stats c
INNER JOIN sys.procedures p 
    ON c.object_id = p.object_id
WHERE p.name LIKE '%pharm%' 
   OR p.name LIKE '%dose%' 
   OR p.name LIKE '%drug%' 
   OR p.name LIKE '%phm%' 
   OR p.name LIKE '%medication%' 
   OR p.name LIKE '%disp%' 
   OR p.name LIKE '%order%'
order by execution_count ASC

### Get the Size of a table containing a certain column

SELECT
	c.TABLE_NAME,
	t.TABLE_TYPE,
	COALESCE(SUM(ps.reserved_page_count) * 8 / 1024, 0) AS table_size_mb
FROM INFORMATION_SCHEMA.COLUMNS c
JOIN INFORMATION_SCHEMA.TABLES t
	ON c.TABLE_NAME = t.TABLE_NAME
	AND c.TABLE_SCHEMA = t.TABLE_SCHEMA
LEFT JOIN sys.tables st ON c.TABLE_NAME = st.name
LEFT JOIN sys.views v ON c.TABLE_NAME = v.name
LEFT JOIN sys.dm_db_partition_stats ps
	ON (st.object_id = ps.object_id OR v.object_id = ps.object_id)
 WHERE
	(c.COLUMN_NAME LIKE '%trade_code%'
	    OR c.COLUMN_NAME LIKE '%drug_code%'
	    OR c.COLUMN_NAME LIKE '%dose%'
	    OR c.COLUMN_NAME LIKE '%pharm%'
	    OR c.COLUMN_NAME LIKE '%disp%'
	    OR c.COLUMN_NAME LIKE '%item%'
	    OR c.COLUMN_NAME LIKE '%diagno%')
GROUP BY c.TABLE_NAME, t.TABLE_TYPE
HAVING COALESCE(SUM(ps.reserved_page_count) * 8 / 1024, 0) > 0  -- Moved filter to HAVING
ORDER BY table_size_mb DESC;

### Identify drugs with no route of administration in DB

SELECT * from trade_drug
WHERE inactive = 1 AND drugcode NOT IN ( SELECT drugcode from DrugstoRoutes)




