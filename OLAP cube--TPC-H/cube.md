# Cube : manipulation de données multi-dimensionnellles OLAP

![alt text](http://upload.wikimedia.org/wikipedia/commons/9/95/Cube.png "screen 1")


### Contexte
Dans le cadre du cours Bases de Données Large Echelle [BDLE] UE (5I852), on est ramené à  calculer, transformer et interroger un cube en SQL sur la base [TPC-H Benchmark] et à répondre aux question du [sujet].

### Environnement technologique 
- Oracle 10G
- Emacs

##### Présentation du Cube C1
C1 représente la somme des ventes d'articles commandés, selon 3 dimensions :

+ dimension x : les produits
+ dimension y : les fournisseurs
+ dimension z : la date de commande

Les niveaux de la hiérarchie des produits sont : nom et type (plusieurs produits par type). 
Les niveaux de la hiérarchie des fournisseurs sont : nom, pays, continent 
Les niveaux de la hiérarchie de la date sont : jour, mois, année

```sql
select *
from (
     select 
		p_name, 
		p_type, 
		s_name, 
		n_name, 
		r_name, 
		o_orderdate, 
		sum(l_extendedprice) 
     from 
		orders o, 
		part p, 
		supplier s, 
		lineitem l, 
		nation n, 
		region r
     where 
		p_partkey = l_partkey
		and l_orderkey = o_orderkey
		and s_suppkey = l_suppkey
		and s_nationkey = n_nationkey
		and n_regionkey = r_regionkey
     group by 
		cube(p_name, s_name, o_orderdate)
	) AS Cube1;
```

##### Question 1 : Opérations algébriques

Faire un slice sur Cube1, garder la ou les tranches dont le p_type se termine par COPPER puis on fait un dice pour sélectionner les dates antérieures au 01/01/1993 rollup sur type avant la sélection sur COPPER, puis rollup sur annee et projection agrégative sur les années inférieures à 1993. Fournisseur proposant les meilleurs prix 

```sql
select *
from (
	select 
		p_type, 
		l_suppkey, 
		sum(l_extendedprice)
	from 
		Lineitem l, 
		Orders o, 
		part p
	where 
		o_orderdate < to_date('01/01/93') 
		and p_type like '%NICKEL'
		and l_orderkey = o_orderkey
		and l_partkey = p_partkey
	group by 
		cube(p_type, l_suppkey)
) AS Cube1;
```

##### Question 2 : Navigation
[a]

```sql
select 
	* 
from 
	Cube2
group by 
	s_name rollup (r_name, n_name, s_name);
```

[b]

```sql
select 
	*
from 
	Cube2
group 
	by s_name, 
	n_name
union
select 
	*
from 
	Cube2
group by 
	s_name, 
	r_name;
```
[c]

```sql
select 
	* 
from 
	Cube2
group by 
	cube(n_name, r_name),
	s_name, 
	r_name;
```


##### Question 3 : Présentation du résultat
[a]
```sql
select 
		p_type, 
		decode(grouping(s_name), 1, 'tous_fournisseurs', s_name), 
		decode( grouping(n_name), 1, 'tous_pays', n_name),
		decode( grouping(r_name), 1, 'tous_continents', r_name),
		sum(l_extendedprice)
from 
		Cube2
group by 
		p_type, 
		rollup (r_name, n_name, s_name); 

```

##### Question 4 : Cube en 3D avec dimension temporelle
[a]

```sql
create view Date_commande as
	select 
		to_char(o_orderdate, 'DD') as orderdate, 
		to_char(o_orderdate, 'MM') as numMois, 
		to_char(o_orderdate, 'YY') as numAnnee
	from 
		orders;
	where
		o_orderdate<to_date('01/03/93','DD/MM/YY')  
	order by 
		orderdate;
```
[b]
select *
from (
     select 
		p_name, 
		p_type, 
		s_name,	
		n_name,
		r_name, 
		o_orderdate,
		sum(l_quantity) 
     from 
		orders o, 
		part p, 
		supplier s, 
		lineitem l, 
		nation n, 
		region r, 
		Date_commande d
     where 
		p_partkey = l_partkey
		and l_orderkey = o_orderkey
		and s_suppkey = l_suppkey
		and s_nationkey = n_nationkey
		and n_regionkey = r_regionkey
     group by 
		(p_name, s_name) rollup(d.orderdate, numMois, numAnnee)
	) AS Cube1;
	 
	 
##### Question 5 : Interrogation d'un cube
[a]	 
	 
	 
```sql
create view les_cubes as
	select 
		p_name as nom_prod,
		p_type as type_prod,
		s_name as fournisseur,
		n_name as pays, 
		r_name as continent, 
		sum (l_quantity * l_extendedprice) as montant_ventes
	from 
		orders o, 
		part p,
		supplier s, 
		lineitem l,
		nation n, 
		region r
    where 
		p_partkey = l_partkey
		and l_orderkey = o_orderkey
		and s_suppkey = l_suppkey
		and s_nationkey = n_nationkey
		and n_regionkey = r_regionkey
    group by 
		cube(p_name, s_name, o_orderdate);
```
[b] R1

```sql
select 
	sum(l_extendedprice) as montant_ventes
from 
	orders o,
	part p, 
	supplier s, 
	lineitem l,
	nation n,
	region r
where 
	p_partkey = l_partkey
	and l_orderkey = o_orderkey
	and s_suppkey = l_suppkey
	and s_nationkey = n_nationkey
	and n_regionkey = r_regionkey
	and r_name = 'AFRICA';	 
```	



[sujet]:http://www-bd.lip6.fr/ens/bdmd2013/index.php/Cube
[BDLE]:http://dac.lip6.fr/master/ues-2014-2015/bdle-2014-2015/
[TPC-H Benchmark]:http://www-master.ufr-info-p6.jussieu.fr/2005/IMG/naacke/bdwa/bdwa2006/extra/tme/tpch-spec-2.3.0.pdf


