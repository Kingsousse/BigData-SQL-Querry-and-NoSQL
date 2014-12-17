# Requêtes analytiques avec TPC H

### Contexte
Dans le cadre du cours Bases de Données Large Echelle [BDLE] UE (5I852), on est ramené à faire des requetes sql analytiques sur la base [TPC-H Benchmark] et à répondre aux question du [sujet].

### Environnement technologique 
- Oracle 10G
- Emacs

##### Exercice 1

####### A1

 Le top 10 des clients ayant dépensé le plus

```sql
with T as (
		select 
			C_CUSTKEY, 
			sum(O_TOTALPRICE) AS TOTAL,  
			RANK() OVER (ORDER BY SUM(O_TOTALPRICE) DESC NULLS LAST ) AS CLASSMENT
		FROM 
			CUSTOMER, 
			ORDERS
		WHERE 
			C_CUSTKEY=O_CUSTKEY
		GROUP BY C_CUSTKEY
		)
select * 
from T 
where rownum <= 10;
```

####### A3
Le top 5 des pays avec le plus grand nombre de clients
```sql
with T as(
		SELECT 
			N_NATIONKEY,
			N_NAME, 
			COUNT(*) AS NOMBRE_DE_CLIENT, 
			DENSE_RANK() OVER (ORDER BY count(*) DESC NULLS LAST ) AS CLASSEMENT
		FROM 
			NATION,
			CUSTOMER
		WHERE 
			C_NATIONKEY=N_NATIONKEY
		GROUP BY 
			N_NAME,
			N_NATIONKEY
		)
select * 
from T 
where CLASSEMENT <= 5;
```

####### A4
Le top 20% des pays avec le plus grand nombre de clients.

```sql
column RANG_RELATIF format 9.99
with T as (
			SELECT 
				N_NATIONKEY,
				N_NAME, 
				COUNT(*) AS NOMBRE_DE_CLIENT, 
				DENSE_RANK() OVER (ORDER BY count(*) DESC NULLS LAST ) AS CLASSEMENT, 
				PERCENT_RANK () OVER (ORDER BY count(*) DESC NULLS LAST ) as RANG_RELATIF
			FROM 
				NATION,
				CUSTOMER
			WHERE 
				C_NATIONKEY=N_NATIONKEY
			GROUP BY 
				N_NAME,
				N_NATIONKEY
			)
select * 
from T
WHERE RANG_RELATIF <=0.2;
```

####### A5
Classement national des produits vendus en plus grande quantité.

```sql
WITH T AS(
			select 
				N_NAME,
				P_PARTKEY,SUM(L_QUANTITY) AS QUANTITY, RANK() OVER (PARTITION BY N_NAME ORDER BY SUM(L_QUANTITY) desc) AS RANG
			FROM 
				PART,
				CUSTOMER,
				NATION,
				LINEITEM,
				ORDERS
			WHERE 
				N_NATIONKEY=C_NATIONKEY
				AND C_CUSTKEY=O_CUSTKEY
				AND O_ORDERKEY=L_ORDERKEY
				AND L_PARTKEY=P_PARTKEY
			GROUP BY 
				N_NAME,
				P_PARTKEY
		)
SELECT * 
FROM T 
WHERE QUANTITY >=150;
```






[sujet]:http://www-bd.lip6.fr/ens/bdmd2013/index.php/Ap
[BDLE]:http://dac.lip6.fr/master/ues-2014-2015/bdle-2014-2015/
[TPC-H Benchmark]:http://www-master.ufr-info-p6.jussieu.fr/2005/IMG/naacke/bdwa/bdwa2006/extra/tme/tpch-spec-2.3.0.pdf


