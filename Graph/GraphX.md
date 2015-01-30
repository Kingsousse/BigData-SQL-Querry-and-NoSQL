https://www.sics.se/~amir/files/download/dic/answers6.pdf


# question1

```scala

import org.apache.spark.graphx._

import org.apache.spark.rdd.RDD

 val fusers=sc.textFile("/user/etu11/facebook_users_prop.csv",4)
 
 val edges = sc.textFile("/user/etu11/facebook_edges_prop.csv")
 
 val RDD_users = fusers.map(x=>x.split(","))
 val RDD_edges = edges.map(x=>x.split(","))

 val users:RDD[(VertexId, (String,String,Int))] = RDD_users.map(x => (x(0).toLong,(x(1),x(2),x(3).toInt)))

 val relationships: RDD[Edge[(String,Int)]] = RDD_edges.map(x=> Edge(x(0).toLong,x(1).toLong,(x(2),x(3).toInt)))
 
 val graph = Graph.apply(users,relationships) 
 

```

# question2

```scala

/*afficher tous les noeuds*/
graph.vertices.collect.foreach(println(_))

scala> graph.vertices.collect.filter{ case(id,(name,prenom,age)) => name == "Kendall" }.foreach { case(id,(name,prenom,age)) => println(s"$name $prenom") }
```

# question3

```scala

scala> graph.triplets.filter(t=> (t.attr._1 == "colleague" && t.attr._2 >= 70 && t.srcAttr._1 == "Kendall")).collect.foreach(t => println(s"${t.dstId}"))
2092
2454
2564
2646

```

# question4

```scala

scala> graph.triplets.filter(t=> (t.attr._1 == "colleague" && t.attr._2 >= 70 && t.srcAttr._1 == "Kendall")).collect.foreach(t => println(s"${t.dstId}"))
2092
2454
2564
2646

```
