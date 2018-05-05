-- sqlite3实验差集
-- 1 集合差集
SELECT	* FROM	test1 
EXCEPT
SELECT	* FROM	test2;

-- 子查询 NOT EXISTS 比 NOT IN 好，不需要考虑 NULL 的影响
SELECT
	* 
FROM
	test1 T1 
WHERE
	NOT EXISTS ( SELECT * FROM test2 WHERE T1.col1 = col2 );
-- 左连接实现
SELECT
	T1.*,
	T2.* 
FROM
	test1 AS T1
	LEFT JOIN test2 AS T2 ON T1.col1 = T2.col2 
WHERE
	T2.col2 IS NULL;
