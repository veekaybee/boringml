# Jaccard Similarity


Implemented two different ways: 

```python
import numpy as numpy
import typing
 
a = [1,2,3,4,5,11,12]
b = [2,3,4,5,6,8,9]

cats = ["calico", "tabby", "tom"]
dogs = ["collie", "tom","bassett"]

def jaccard(list1: list, list2: list)-> float:
	intersection = len(list(set(list1).intersection(list2)))
	union  = (len(set((list1)) + set(len(list2))) - intersection
	return float(intersection/union)

print(jaccard(cats,dogs))
```
## jaccardSimilarity in Scala

```scala

val aVals: Seq[Int] = Seq(1,2,3,4,5,11,12)
val bVals: Seq[Int]  = Seq(2,3,4,5,6,8,9)

def calculateJaccard[T](a: Seq[T], b: Seq[T]): Double = a.intersect(b).size / a.union(b).size.toDouble

println(calculateJaccard(aVals, bVals))
```
