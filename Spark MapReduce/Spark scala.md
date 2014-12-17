# Utilisation de Scala sous Spark

### Contexte
Dans le cadre du cours Bases de Données Large Echelle [BDLE] UE (5I852), on est ramené à faire du traduction des  requetes sql en MapReduce et à répondre aux question du [sujet].

### Environnement technologique 
- Hadoop
- Spark 1.1.1
- Linux debian
- [movielens]

##### Exercice 1

####### 1

```scala
val data=sc.textFile("/Vrac/bdle/data/wordcount-small.txt")
/*data: org.apache.spark.rdd.RDD[String] = MappedRDD[1] at textFile at <console>:12*/
data.take(10)
/*res0: Array[String] = Array(De.mw De 1 10302, De Customer-Relationship-Management 3 96858, De Include 3 24679, EN.mw EN 1 4693, En.d disease 3 153755, En.d linking 3 23139, En.d players 3 21289, En.d position 1 0, En.d select 3 42214, En.d updates 3 24145)
*/
```
 
####### 2
```scala
val list = data.map(x=>x.split(" ")).collect
/*list: Array[Array[String]] = Array(Array(De.mw, De, 1, 10302), Array(De, Customer-Relationship-Management, 3, 96858), Array(De, Include, 3, 24679), Array(EN.mw, EN, 1, 4693), Array(En.d, disease, 3, 153755), Array(En.d, linking, 3, 23139), Array(En.d, players, 3, 21289), Array(En.d, position, 1, 0), Array(En.d, select, 3, 42214), Array(En.d, updates, 3, 24145))...
*/	
val col3=list.map(x=>x(2)).take(100)
```

####### 3
```scala
val couple=list.map(x=>(x(0),x(2).toInt))
couple take 10
/*couple: Array[(String, Int)] = Array((De.mw,1), (De,3), (De,3), (EN.mw,1), (En.d,3), (En.d,3), (En.d,3), (En.d,1), (En.d,3), (En.d,3))*/
```

####### 4
```scala
val groupe = couple.reduceByKey(_+_)
groupe take 10
/*groupe: Array[(String, Int)] = Array((De.mw,1), (aa,41), (ab.mw,37), (ace.mw,184), (Fr,5), (aa.d,4), (af.wd,17), (ace,347), (af.b,57), (aa.b,15))*/
```

####### 5
```scala
val couplePref=list.map(x=>(x(0).split("\\.")(0), x(2)))
/*couplePref: Array[(String, String)] = Array((De,1), (De,3), (De,3), (EN,1), (En,3), (En,3), (En,3), (En,1), (En,3), (En,3), (En,1), (En,1), (En,1), (En,3), (En,3), (En,3), (En,3), (En,3), (En,1), (En,2), (En,1), (En,1), (En,3), (En,1), (En,3), (En,3), (En,3), (En,3), (En,1), (En,3), (En,1), (En,3), (En,2), (En,3), (En,3), (Fr,2), (Fr,3), (Www,1), (Www,2), (Www,2), (Www,3), (Www,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,12), (aa,6), (aa,1), (aa,1), (aa,4), (aa,6), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,6), (aa,3), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (aa,1), (ab,1), (ab,37), (ab,2), (ab,1), (ab,1), (ab,1), (ab,1), (ab,1), (ab,5), (ab,15), (...
*/
val coupleRDD = sc.parallelize(couplePref)
val groupePref=couplePref.reduceByKey(_+_)
```

##### Exercice 2

####### 1

```scala
val f1=sc.textFile("/Vrac/bdle/data/ratings.dat")
val f2=sc.textFile("/Vrac/bdle/data/users.dat")
```

####### 2

```scala
val c1=f1.map(x=>x.split("::"))
val c2=f2.map(x=>x.split("::"))
```

####### 3

```scala
var d1 = rat.map(x=>x.split("::")).map(x=>(x(0),x(2).toInt))
d1.reduceByKey(_+_)
```

####### 4

```scala
cpt.collect.toSeq.sortBy(-_._2)
```

[sujet]:http://dac.lip6.fr/master/wp-content/uploads/2014/10/TME1-Etudiants.pdf
[BDLE]:http://dac.lip6.fr/master/ues-2014-2015/bdle-2014-2015/
[TPC-H Benchmark]:http://www-master.ufr-info-p6.jussieu.fr/2005/IMG/naacke/bdwa/bdwa2006/extra/tme/tpch-spec-2.3.0.pdf
[movielens]:http://grouplens.org/datasets/movielens/
