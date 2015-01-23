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

# Calculs sur des graphes en M/R - Spark

####### Simple pageRank programme

```scala
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */


import org.apache.spark.SparkContext._
import org.apache.spark.{SparkConf, SparkContext, HashPartitioner}

/**
 * Computes the PageRank of URLs from an input file. Input file should
 * be in format of:
 * URL         neighbor URL
 * URL         neighbor URL
 * URL         neighbor URL
 * ...
 * where URL and their neighbors are separated by space(s).
 */
object SparkPageRank {
  def main(args: Array[String]) {
    val sparkConf = new SparkConf().setAppName("PageRank V1")
    var iters = args(1).toInt
    val ctx = new SparkContext(sparkConf)
    val lines = ctx.textFile(args(0), 4)
    val links = lines.map{ s =>
      val parts = s.split(",")
      (parts(0), parts(1))
    }.groupByKey().map(url=>(url._1, (url._2, url._2.size))).cache()

    var ranks = links.groupByKey().map(url => (url._1,1.0))

    for (i <- 1 to iters) {
      val contribs = links.join(ranks).flatMap{ case (url, ((links,size), rank)) =>
      	  links.map(url => (url, rank / size))
      }
      ranks = contribs.reduceByKey(_ + _).mapValues(0.15 + 0.85 * _)
    }

    val output = ranks.collect()
    output.foreach(tup => println(tup._1 + " has rank: " + tup._2 + "."))

    ctx.stop()
  }
}

```

la résultat c'est : 
```shell
2441 has rank: 0.24586389596627822.
2764 has rank: 0.3137267666755489.
1873 has rank: 0.9046737910222268.
1004 has rank: 0.1548035284241112.
1983 has rank: 0.16392253840636256.
3394 has rank: 4.557221058883569.
3299 has rank: 0.8495475867568956.
128 has rank: 0.17799237814508012.
2595 has rank: 0.4256211143208093.
601 has rank: 0.24330216560982992.
2063 has rank: 0.1721382862814687.
3680 has rank: 0.3069048530883141.
2504 has rank: 0.5141683893943134.
917 has rank: 0.1793903503367873.
821 has rank: 0.19129007871028003.
1073 has rank: 0.17874194051812156.
931 has rank: 0.17039190019470415.
458 has rank: 0.20799388841323513.
3783 has rank: 0.2948242898863366.
2720 has rank: 0.15373551698210836.
1282 has rank: 0.2635769227632171.
98 has rank: 0.17870408907402094.
2665 has rank: 0.15048308022815618.
```

####### 


[sujet]:http://dac.lip6.fr/master/wp-content/uploads/2014/10/TME1-Etudiants.pdf
[BDLE]:http://dac.lip6.fr/master/ues-2014-2015/bdle-2014-2015/
[TPC-H Benchmark]:http://www-master.ufr-info-p6.jussieu.fr/2005/IMG/naacke/bdwa/bdwa2006/extra/tme/tpch-spec-2.3.0.pdf
[movielens]:http://grouplens.org/datasets/movielens/
