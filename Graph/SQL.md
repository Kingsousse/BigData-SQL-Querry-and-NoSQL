# Cube : manipulation de données multi-dimensionnellles OLAP

![alt text](http://upload.wikimedia.org/wikipedia/commons/9/95/Cube.png "screen 1")


### Contexte
Dans le cadre du cours Bases de Données Large Echelle [BDLE] UE (5I852), on est ramené à répondre aux question du [sujet].

### Environnement technologique 
- Oracle 10G
- Emacs
- [Tables Nodes_Facebook & Edges_Facebook]

##### Question 1 : 

une requête SQL qui retourne les voisins de l'utilisateurs 'Kendall' (il existe une seule personne avec ce nom), en considérant que le graphe est dirigé. 


```sql
select 
	nf.name
from 
	nodes_facebook nf, 
	edges_facebook ef 
where 
	nf.id = ef.neighbor 
	and ef.usr = (
					select 
						id 
					from 
						nodes_facebook 
					where 
						name = 'Kendall' 
				);
```


La même requête en considérant que le graphe est non-dirigé. 

```sql
select 
	nf.name 
from 
	nodes_facebook nf, 
	edges_facebook ef 
where 
	nf.id = ef.neighbor 
	and ef.usr = (
					select 
						id 
					from 
						nodes_facebook 
					where 
						name = 'Kendall' 
				) 
union	select 
			nf.name 
		from 
			nodes_facebook nf, 
			edges_facebook ef 
		where 
			nf.id = ef.usr 
			and ef.neighbor = (
								select 
									id 
								from 
									nodes_facebook 
								where 
									name = 'Kendall'
							);
```

alternative

```sql
select 
	nf.name 
from 
	nodes_facebook nf,
	edges_facebook ef 
where (
		nf.id = ef.neighbor 
		and ef.usr = (
						select 
							id 
						from 
							nodes_facebook 
						where 
							name = 'Kendall' 
					)
	) or(
		nf.id = ef.usr 
		and ef.neighbor = (
							select 
								id 
							from 
								nodes_facebook 
							where 
								name = 'Kendall' 
						)
		);
```

La même question pour un graphe non dirigé


##### Question 2 : 

 les noms des amis des amis de Kendall (à distance 2 de Kendall), en considérant d'abord que le graphe est dirigé.

```sql
select 
	distinct(tnf.id),
	tnf.name 
from 
	nodes_facebook tnf,
	edges_facebook tef  
where
	tnf.id = tef.neighbor 
	and tef.Usr in (
						select 
							nf.id 
						from 
							nodes_facebook nf, 
							edges_facebook ef 
						where 
							nf.id = ef.neighbor 
							and ef.usr = (
											select 
												id 
											from 
												nodes_facebook 
											where 
												name = 'Kendall' 
										)
					);
```

La même question pour un graphe non dirigé. 


```sql
select 
	distinct(tnf.id),
	tnf.name 
from 
	nodes_facebook tnf,
	edges_facebook tef  
where ( 
	tnf.id = tef.neighbor 
	and tef.Usr in (
					select 
						nf.id 
					from 
						nodes_facebook nf, 
						edges_facebook ef 
					where (
						nf.id = ef.neighbor 
						and ef.usr = (
										select 
											id 
										from 
											nodes_facebook 
										where 
										name = 'Kendall' 
									)
							) 
					or (
						nf.id = ef.usr 
						and ef.neighbor = (
											select 
												id 
											from 
												nodes_facebook 
											where 
												name = 'Kendall' 
											)
						)
					)
	)
or ( 
	tnf.id = tef.usr 
	and tef.neighbor in (
						select 
							nf.id 
						from 
							nodes_facebook nf, 
							edges_facebook ef 
						where (
								nf.id = ef.neighbor 
								and ef.usr = (
												select 
													id 
												from 
													nodes_facebook 
												where 
													name = 'Kendall' 
											)
							) or(
								nf.id = ef.usr 
								and ef.neighbor = (
													select 
														id 
													from 
														nodes_facebook 
													where 
														name = 'Kendall' 
												)
								)
						)
	);
```

##### Question 3 : 

Les noms des utilisateurs atteignables à une distance donnée à partir de l'utilisateur 'Kendall'

```sql
with amis(id, name, niveau) as
(
	select 
		distinct id, 
		name, 
		0 
	from 
		nodes_facebook nf 
	where 
		nf.name = 'Kendall'
	union all select 
		nf.id, 
		nf.name, 
		am.niveau +1 
	from 
		nodes_facebook nf, 
		edges_facebook ef, 
		amis am 
	where
		ef.usr = am.id 
		and nf.id = ef.neighbor 
		and niveau < 2
)
select 
	distinct(id), 
	name, 
	niveau 
from 
	amis 
where 
	niveau = 2;  
```





[Tables Nodes_Facebook & Edges_Facebook]:http://snap.stanford.edu/data/egonets-Facebook.html
[sujet]:http://dac.lip6.fr/master/wp-content/uploads/2014/09/TME11.pdf
[BDLE]:http://dac.lip6.fr/master/ues-2014-2015/bdle-2014-2015/
[TPC-H Benchmark]:http://www-master.ufr-info-p6.jussieu.fr/2005/IMG/naacke/bdwa/bdwa2006/extra/tme/tpch-spec-2.3.0.pdf