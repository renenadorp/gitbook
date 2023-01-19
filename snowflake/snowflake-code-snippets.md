---
description: Code Snippets
---

# Code Snippets

### -- TABLE ROWCOUNT

USE DATABASE DWH\_DEV; SELECT t.table\_schema || '.' || t.table\_name as "table\_name",t.row\_count FROM information\_schema.tables t WHERE t.table\_type = 'BASE TABLE' and t.table\_catalog = 'DWH\_DEV' AND T.TABLE\_SCHEMA ='BA' ORDER BY t.row\_count;

### -- DROP TABLE

SELECT 'DROP TABLE ' || TABLE\_SCHEMA ||'.'||TABLE\_NAME || ';' FROM INFORMATION\_SCHEMA.TABLES WHERE TABLE\_CATALOG = 'DWH\_DEV\_RNADORP' AND TABLE\_SCHEMA IN ('BA','BA\_TMP','DA','DA\_TMP', 'IA', 'MA');

### -- ALL DATABASE OBJECTS

USE DATABASE DWH\_DEV;

SELECT 'TABLE' AS OBJECT\_TYPE, TABLE\_CATALOG AS DATABASE, TABLE\_SCHEMA AS SCHEMA, TABLE\_NAME AS NAME , TABLE\_OWNER AS OWNER, ROW\_COUNT AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.TABLES WHERE TABLE\_SCHEMA IN ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI')

UNION ALL

SELECT 'VIEW' AS OBJECT\_TYPE, TABLE\_CATALOG AS DATABASE, TABLE\_SCHEMA AS SCHEMA, TABLE\_NAME AS NAME , TABLE\_OWNER AS OWNER, NULL AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.VIEWS WHERE TABLE\_SCHEMA IN ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI')

UNION ALL

SELECT 'FORMAT' AS OBJECT\_TYPE, FILE\_FORMAT\_CATALOG AS DATABASE, FILE\_FORMAT\_SCHEMA AS SCHEMA, FILE\_FORMAT\_NAME AS NAME , FILE\_FORMAT\_OWNER AS OWNER, NULL AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.FILE\_FORMATS

UNION ALL

SELECT 'STAGE' AS OBJECT\_TYPE, STAGE\_CATALOG AS DATABASE, STAGE\_SCHEMA AS SCHEMA, STAGE\_NAME AS NAME , STAGE\_OWNER AS OWNER, NULL AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.STAGES

UNION ALL

SELECT 'SEQ' AS OBJECT\_TYPE, SEQUENCE\_CATALOG AS DATABASE, SEQUENCE\_SCHEMA AS SCHEMA, SEQUENCE\_NAME AS NAME , SEQUENCE\_OWNER AS OWNER, NULL AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.SEQUENCES WHERE SEQUENCE\_SCHEMA IN ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI')

UNION ALL

SELECT 'PROC' AS OBJECT\_TYPE, PROCEDURE\_CATALOG AS DATABASE, PROCEDURE\_SCHEMA AS SCHEMA, PROCEDURE\_NAME AS NAME , PROCEDURE\_OWNER AS OWNER, NULL AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.PROCEDURES WHERE PROCEDURE\_SCHEMA IN ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI')

UNION ALL

SELECT 'PIPE' AS OBJECT\_TYPE, PIPE\_CATALOG AS DATABASE, PIPE\_SCHEMA AS SCHEMA, PIPE\_NAME AS NAME , PIPE\_OWNER AS OWNER, NULL AS ROW\_COUNT, LAST\_ALTERED AS CHANGE\_DATE FROM INFORMATION\_SCHEMA.PIPES WHERE PIPE\_SCHEMA IN ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI') ;

### -- COMPARE MELOG\_LIQUIBASE MD5SUM BETWEEN DATABASES

select case when dev\_lqb.md5sum <> tst\_lqb.md5sum then 'CHANGED' when tst\_lqb.md5sum is null then 'NEW' end as IND, dev\_lqb.\* from dwh\_dev\_rnadorp.ma.melog\_liquibase dev\_lqb left outer join dwh\_tst.ma.melog\_liquibase tst\_lqb on dev\_lqb.id = tst\_lqb.id where dev\_lqb.md5sum <> tst\_lqb.md5sum or tst\_lqb.id is null

;

### -- INSERT DATA INTO DA.RTDIV\_TIME

insert into da.rtdiv\_time select --dateadd(day, '-' || seq4(), current\_date()) as dte, to\_time(to\_varchar(floor(seq4()/ 60), '00') || ':' || to\_varchar(mod(seq4(),60), '00') || ':00') as time ,'DIV' AS TK\_SOURCESYSTEM ,2 AS TA\_STATUS\_CODE ,1 AS TA\_METADATA ,CURRENT\_DATE() AS TA\_INSERT\_DATETIME ,CURRENT\_DATE() AS TA\_UPDATE\_DATETIME ,1 AS TA\_HASH ,'MANUAL' TA\_RUNID ,MA.MSSEQ\_ROWID\_DWH.NEXTVAL AS TA\_ROWID from table (generator(rowcount => 60\*24)) t;

### -- INSERT DATA INTO DA.RTDIV\_DATE

INSERT INTO DA.RTDIV\_DATE ( SELECT DATEADD(DAY, SEQ4(), date('2000-01-01')) AS DATE, 'DIV' AS TK\_SOURCESYSTEM, 2 AS TA\_STATUS\_CODE, 1 AS TA\_METADATA, CURRENT\_DATE() AS TA\_INSERT\_DATETIME, CURRENT\_DATE() AS TA\_UPDATE\_DATETIME, 1 AS TA\_HASH, '1' AS TA\_RUNID, NULL FROM TABLE(GENERATOR(ROWCOUNT=>366\*500)) -- Number of days after reference date in previous line ) ;

### -- TABLES WITHOUT PK

WITH PK AS ( select c.constraint\_name, case when c.constraint\_name is null then FALSE ELSE TRUE end as HAS\_PK, t. _from information\_schema.tables t left outer join information\_schema.table\_constraints c on t.table\_catalog = c.table\_catalog and t.table\_schema = c.table\_schema and t.table\_name = c.table\_name and c.constraint\_type = 'PRIMARY KEY' where t.table\_schema in ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI') ) SELECT_ FROM PK WHERE NOT HAS\_PK ;

### -- TABLES WITH WRONG OWNER (NOT DDL\_)

with t as ( select _from information\_schema.TABLES T where t.table\_schema in ('DA', 'DA\_TMP', 'BA', 'BA\_TMP', 'MA', 'IA\_POWERBI')_\
_) select_ from t where table_owner NOT LIKE 'DDL_%'

### -- IA Views

select _from ia\_powerbi.tijd; select_ from ia\_powerbi.datum; select _from ia\_powerbi.regio; -- 0 select_ from ia\_powerbi.concessie; --0 select _from ia\_powerbi.subconcessie; --0 select_ from ia\_powerbi.trein; --0 select _from ia\_powerbi.treinactiviteit; select_ from ia\_powerbi."Trein volgorde"; --0 select \* from ia\_powerbi.plaats; --0

### -- COLUMN\_DEFAULT: SEQUENCE CANNOT BE FOUND

select \* from information\_schema.columns where column\_default is not null AND TABLE\_SCHEMA IN ('BA','BA\_TMP','DA','DA\_TMP', 'IA', 'MA') AND COLUMN\_DEFAULT LIKE '%sequence cannot be found%' ;

select 'ALTER TABLE ' || table\_schema || '.' || table\_name || ' ALTER COLUMN ' || COLUMN\_NAME || ' SET DEFAULT MA.MSSEQ\_ROWID\_DWH.NEXTVAL;' from information\_schema.columns where column\_default is not null AND TABLE\_SCHEMA IN ('BA','BA\_TMP','DA','DA\_TMP', 'IA', 'MA') AND COLUMN\_DEFAULT LIKE '%sequence cannot be found%' ;

### -- RETRIEVE PROCEDURE DDL

show procedures in database dwh\_dev\_rnadorp; select get\_ddl('PROCEDURE', 'DWH\_DEV\_RNADORP.PUBLIC.UDF\_CREATE\_UNIT\_TESTS(VARCHAR, VARCHAR, VARCHAR)');

### -- TABLE PK COLUMNS

SHOW PRIMARY KEYS IN DATABASE dwh\_dev; create OR REPLACE temp table tmp\_pk as ( select \* from table(result\_scan(last\_query\_id())) where "schema\_name" IN ('BA','BA\_TMP','DA','DA\_TMP', 'IA', 'MA') ) ;

### -- GET DDL

SELECT GET\_DDL('VIEW', 'DWH\_DEV\_RNADORP.PUBLIC.TEST');

\-------- CLEAR DATABASE set DB = 'DWH\_DEV\_RNADORP'; USE ROLE DEVELOPER; USE DATABASE DWH\_DEV\_RNADORP; call DWH\_DEV\_RNADORP.PUBLIC.UDF\_clear\_schema($DB, 'MA'); call DWH\_DEV\_RNADORP.PUBLIC.udf\_clear\_schema($DB, 'DA'); call DWH\_DEV\_RNADORP.PUBLIC.udf\_clear\_schema($DB, 'DA\_TMP'); call DWH\_DEV\_RNADORP.PUBLIC.udf\_clear\_schema($DB, 'BA');

call DWH\_DEV\_RNADORP.PUBLIC.udf\_clear\_schema($DB, 'BA\_TMP');

### -- COMPARE QUERY PERFORMANCE BETWEEN 2 IMPLEMENTATIONS

USE ROLE ACCOUNTADMIN; USE DATABASE DWH\_DEV; USE SCHEMA DA;

ALTER SESSION SET USE\_CACHED\_RESULT = FALSE; -- Do not use Query Result cache to ensure comparability SET WAIT\_QTY = 1; -- Number of units to wait after suspending a warehouse to ensure the local cache is flushed SET WAIT\_UNIT = 'MINUTES';

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME;

\--Query 1: Item Key uit ba\_inventorytransaction (inventtrans: 7.992.728 rijen, inventtable: 90.358 rijen) --Origineel: SELECT concat\_ws( '#', it.tk\_sourcesystem, it.partition, it.recid) AS INVENTORY\_TRANSACTION\_KEY , concat\_ws( '#', i.tk\_sourcesystem, i.partition, i.recid) AS ITEM\_KEY FROM da.hvaxe\_inventtrans\_ax2012r3 it LEFT JOIN da.hvaxe\_inventtable\_ax2012r3 i ON it.ea\_actual\_flag = 1 AND i.tk\_sourcesystem = it.tk\_sourcesystem AND i.partition = it.partition AND i.dataareaid = it.dataareaid AND i.itemid = it.itemid AND i.ea\_actual\_flag = 1 AND i.ta\_status\_code = 2 ; SET QID\_Q1\_A = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT);

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\--Aangepast (recid vervangen met itemid uit hoofdtabel): SELECT concat\_ws( '#', it.tk\_sourcesystem, it.partition, it.recid) AS INVENTORY\_TRANSACTION\_KEY , concat\_ws( '#', it.tk\_sourcesystem, it.partition, it.itemid) AS ITEM\_KEY FROM da.hvaxe\_inventtrans\_ax2012r3 it //LEFT JOIN da.hvaxe\_inventtable\_ax2012r3 i WHERE TRUE AND it.ea\_actual\_flag = 1 // AND i.tk\_sourcesystem = it.tk\_sourcesystem // AND i.partition = it.partition // AND i.dataareaid = it.dataareaid // AND i.itemid = it.itemid // AND i.ea\_actual\_flag = 1 // AND i.ta\_status\_code = 2 // ;

SET QID\_Q1\_B = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT);

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE; --Query 2: Sales\_Order\_Key uit ba\_inventorytransaction (join op join, inventtrans: 7.992.728 rijen, salesline: 1.126.069 rijen): --Origineel SELECT concat\_ws( '#', it.tk\_sourcesystem, it.partition, it.recid) AS INVENTORY\_TRANSACTION\_KEY , concat\_ws( '#', sl.tk\_sourcesystem, sl.partition, sl.recid) AS SALES\_ORDER\_KEY FROM da.hvaxe\_inventtrans\_ax2012r3 it LEFT JOIN da.hvaxe\_inventtransorigin\_ax2012r3 io ON io.recid = it.inventtransorigin AND io.dataareaid = it.dataareaid AND io.partition = it.partition AND io.ea\_actual\_flag = 1 LEFT JOIN da.hvaxe\_salesline\_ax2012r3 sl on sl.inventtransid = io.inventtransid AND sl.partition = io.partition AND sl.dataareaid = io.dataareaid AND sl.tk\_sourcesystem = io.tk\_sourcesystem AND sl.ea\_actual\_flag = 1 ; SET QID\_Q2\_A = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT);

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\--Aangepast (recid vervangen met inventtransid uit inventtransorigin) SELECT concat\_ws( '#', it.tk\_sourcesystem, it.partition, it.recid) AS INVENTORY\_TRANSACTION\_KEY , concat\_ws( '#', it.tk\_sourcesystem, it.partition, io.inventtransid) AS SALES\_ORDER\_KEY FROM da.hvaxe\_inventtrans\_ax2012r3 it LEFT JOIN da.hvaxe\_inventtransorigin\_ax2012r3 io on io.recid = it.inventtransorigin AND io.dataareaid = it.dataareaid AND io.partition = it.partition AND io.ea\_actual\_flag = 1 //LEFT JOIN da.hvaxe\_salesline\_ax2012r3 sl // ON sl.inventtransid = io.inventtransid // AND sl.partition = io.partition // AND sl.dataareaid = io.dataareaid // AND sl.tk\_sourcesystem = io.tk\_sourcesystem // AND sl.ea\_actual\_flag = 1 //

### ;

SET QID\_Q2\_B = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT);

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\--Query 3: Customer\_key uit ba\_salesorder (salestable: 705.904 rijen, custtable: 40.337 rijen) --Origineel SELECT concat\_ws( '#', cus.tk\_sourcesystem, cus.partition, cus.recid) AS CUSTOMER\_KEY FROM da.hvaxe\_salestable\_ax2012r3 s LEFT OUTER JOIN da.hvaxe\_custtable\_ax2012r3 cus ON cus.tk\_sourcesystem = s.tk\_sourcesystem AND cus.partition = s.partition AND cus.dataareaid = s.dataareaid AND cus.accountnum = s.custaccount AND cus.ea\_actual\_flag = 1 AND cus.ta\_status\_code = 2 ; SET QID\_Q3\_A = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT);

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\--Aangepast (recid vervangen door custaccount) SELECT concat\_ws( '#', s.tk\_sourcesystem, s.partition, s.custaccount) AS CUSTOMER\_KEY FROM da.hvaxe\_salestable\_ax2012r3 s //LEFT OUTER JOIN da.hvaxe\_custtable\_ax2012r3 cus // ON cus.tk\_sourcesystem = s.tk\_sourcesystem // AND cus.partition = s.partition // AND cus.dataareaid = s.dataareaid // AND cus.accountnum = s.custaccount // AND cus.ea\_actual\_flag = 1 // AND cus.ta\_status\_code = 2 ; SET QID\_Q3\_B = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT);

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\-- Query 4: Purchaseorder\_Key van ba\_purchaseorder (purchtable: 331.809 rijen, purchline: 417.837 rijen) -- Origineel SELECT concat\_ws( '#', pl.tk\_sourcesystem, pl.partition, pl.recid) AS PURCHASEORDER\_KEY FROM da.hvaxe\_purchtable\_ax2012r3 p LEFT OUTER JOIN da.hvaxe\_purchline\_ax2012r3 pl ON pl.tk\_sourcesystem = p.tk\_sourcesystem AND pl.partition = p.partition AND pl.dataareaid = p.dataareaid AND pl.purchid = p.purchid AND pl.ea\_actual\_flag = 1 ; SET QID\_Q4\_A = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT); ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\--Aangepast (recid vervangen door purchid) SELECT concat\_ws( '#', p.tk\_sourcesystem, p.partition, p.purchid) AS PURCHASEORDER\_KEY FROM da.hvaxe\_purchtable\_ax2012r3 p // LEFT OUTER JOIN da.hvaxe\_purchline\_ax2012r3 pl // ON pl.tk\_sourcesystem = p.tk\_sourcesystem // AND pl.partition = p.partition // AND pl.dataareaid = p.dataareaid // AND pl.purchid = p.purchid // AND pl.ea\_actual\_flag = 1 ; SET QID\_Q4\_B = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT); ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE;

\-- Query 5: Salesorder\_key van salesorderinvoice (Custinvoicetrans: 875.918 rijen, Salesline: 1.126.073 rijen) -- Origineel SELECT concat\_ws( '#', sl.tk\_sourcesystem, sl.partition, sl.recid) AS Salesorder\_Key FROM da.HVAXE\_CUSTINVOICETRANS\_AX2012R3 inv LEFT JOIN da.HVAXE\_SALESLINE\_AX2012R3 sl ON sl.inventtransid = inv.inventtransid AND sl.dataAreaID = inv.DataAreaID AND sl.partition = inv.partition AND sl.tk\_sourcesystem = inv.tk\_sourcesystem AND sl.ea\_actual\_flag = 1 AND sl.ta\_status\_code = 2 ; SET QID\_Q5\_A = last\_query\_id();

ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME IF SUSPENDED; ALTER WAREHOUSE WH\_COAPTTEAM\_M SUSPEND; call system$wait($WAIT\_QTY, $WAIT\_UNIT); ALTER WAREHOUSE WH\_COAPTTEAM\_M RESUME; ALTER ACCOUNT SET USE\_CACHED\_RESULT = FALSE; -- Aangepast (recid vervangen door inventtransid) SELECT concat\_ws( '#', inv.tk\_sourcesystem, inv.partition, inv.inventtransid) AS Salesorder\_Key FROM da.HVAXE\_CUSTINVOICETRANS\_AX2012R3 inv //LEFT JOIN da.HVAXE\_SALESLINE\_AX2012R3 sl // ON sl.inventtransid = inv.inventtransid // AND sl.dataAreaID = inv.DataAreaID // AND sl.partition = inv.partition // AND sl.tk\_sourcesystem = inv.tk\_sourcesystem // AND sl.ea\_actual\_flag = 1 // AND sl.ta\_status\_code = 2 ; SET QID\_Q5\_B = last\_query\_id();

call system$wait(4, 'MINUTES');

SELECT CASE WHEN QUERY\_ID = '' || $QID\_Q1\_A || '' THEN 'QID\_Q1\_A' WHEN QUERY\_ID = '' || $QID\_Q1\_B || '' THEN 'QID\_Q1\_B' WHEN QUERY\_ID = '' || $QID\_Q2\_A || '' THEN 'QID\_Q2\_A' WHEN QUERY\_ID = '' || $QID\_Q2\_B || '' THEN 'QID\_Q2\_B' WHEN QUERY\_ID = '' || $QID\_Q3\_A || '' THEN 'QID\_Q3\_A' WHEN QUERY\_ID = '' || $QID\_Q3\_B || '' THEN 'QID\_Q3\_B' WHEN QUERY\_ID = '' || $QID\_Q4\_A || '' THEN 'QID\_Q4\_A' WHEN QUERY\_ID = '' || $QID\_Q4\_B || '' THEN 'QID\_Q4\_B' WHEN QUERY\_ID = '' || $QID\_Q5\_A || '' THEN 'QID\_Q5\_A' WHEN QUERY\_ID = '' || $QID\_Q5\_B || '' THEN 'QID\_Q5\_B' END AS QID , QH.\*

FROM SNOWFLAKE.ACCOUNT\_USAGE.QUERY\_HISTORY QH WHERE QUERY\_ID IN (

'' || $QID\_Q1\_A || '' , '' || $QID\_Q1\_B || '' , '' || $QID\_Q2\_A || '' , '' || $QID\_Q2\_B || '' , '' || $QID\_Q3\_A || '' , '' || $QID\_Q3\_B || '' , '' || $QID\_Q4\_A || '' , '' || $QID\_Q4\_B || '' , '' || $QID\_Q5\_A || '' , '' || $QID\_Q5\_B || ''

) ;
