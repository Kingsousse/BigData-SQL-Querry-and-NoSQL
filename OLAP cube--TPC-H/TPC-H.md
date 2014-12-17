# Requêtes décisionnelles avec TPC H

### Contexte
Dans le cadre du cours Bases de Données Large Echelle [BDLE] UE (5I852), on est ramené à faire des requetes sql d'aide à la décision sur la base TPC-H Benchmark et à répondre aux question du [sujet].

### Environnement technologique 
- Oracle 10G
- Emacs

##### R1
Résumé des prix

```sql
select
        l_returnflag,
        l_linestatus,
        sum(l_quantity) as sum_qty,
        sum(l_extendedprice) as sum_base_price,
        sum(l_extendedprice*(1-l_discount)) as sum_disc_price,
        sum(l_extendedprice*(1-l_discount)*(1+l_tax)) as sum_charge,
        avg(l_quantity) as avg_qty,
        avg(l_extendedprice) as avg_price,
        avg(l_discount) as avg_disc,
        count(*) as count_order
from
        lineitem
where
        l_shipdate <= date '1998-12-01' - interval '[DELTA]' day (3)
group by
        l_returnflag,
        l_linestatus
order by
        l_returnflag,
        l_linestatus;
```

##### R2
Fournisseur proposant les meilleurs prix 
```sql
select
        s_acctbal,
        s_name,
        n_name,
        p_partkey,
        p_mfgr,
        s_address,
        s_phone,
        s_comment
from
        part,
        supplier,
        partsupp,
        nation,
        region
where
        p_partkey = ps_partkey
        and s_suppkey = ps_suppkey
        and p_size = [SIZE]
        and p_type like '%[TYPE]'
        and s_nationkey = n_nationkey
        and n_regionkey = r_regionkey
        and r_name = '[REGION]'
        and ps_supplycost = (
                                select
                                    min(ps_supplycost)
                                from
                                    partsupp, supplier,
                                    nation, region
                                where
                                    p_partkey = ps_partkey
                                    and s_suppkey = ps_suppkey
                                    and s_nationkey = n_nationkey
                                    and n_regionkey = r_regionkey
                                    and r_name = '[REGION]'
                            )
order by
        s_acctbal desc,
        n_name,
        s_name,
        p_partkey;
```

##### R1
Commandes à expédier en priorité

```sql
select
        l_orderkey,
        sum(l_extendedprice*(1-l_discount)) as revenue,
        o_orderdate,
        o_shippriority
from
        customer,
        orders,
        lineitem
where
        c_mktsegment = '[SEGMENT]'
        and c_custkey = o_custkey
        and l_orderkey = o_orderkey
        and o_orderdate < date '[DATE]'
        and l_shipdate > date '[DATE]'
group by
        l_orderkey,
        o_orderdate,
        o_shippriority
order by
        revenue desc,
        o_orderdate;
```


### Notes
Pour tous les réponse de tous les requetes dans le TME, on peut le trouver dans le [TPC BENCHMARKTM H] 




[sujet]:http://www-poleia.lip6.fr/~doucet/CoursBDWA/td-tpc-H.pdf
[BDLE]:http://dac.lip6.fr/master/ues-2014-2015/bdle-2014-2015/
[TPC BENCHMARKTM H]:http://www-master.ufr-info-p6.jussieu.fr/2005/IMG/naacke/bdwa/bdwa2006/extra/tme/tpch-spec-2.3.0.pdf


