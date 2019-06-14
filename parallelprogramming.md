Parallel Programming
====

These are some notes for the 3rd course of Scala Specialization on Cousera.

<!-- TOC -->

- [Week 1 Basics](#week-1-basics)
  - [JVM and parallelism](#jvm-and-parallelism)
    - [Process](#process)
    - [Threads](#threads)
    - [Creating/Starting Threads](#creatingstarting-threads)
    - [Example: Starting Threads](#example-starting-threads)
    - [Atomicity](#atomicity)
    - [The Synchronized Block](#the-synchronized-block)
    - [Composition with the synchronized block](#composition-with-the-synchronized-block)
    - [Deadlocks](#deadlocks)
    - [Memory Model](#memory-model)
    - [Summary](#summary)
  - [Running Computations in Parallel](#running-computations-in-parallel)
    - [Basic parallel construct](#basic-parallel-construct)
    - [Example: computing p-norm](#example-computing-p-norm)
    - [Signature of parallel](#signature-of-parallel)
    - [Underlying Hardware Architecture Affects Performance](#underlying-hardware-architecture-affects-performance)
    - [Combining computations of different length with parallel](#combining-computations-of-different-length-with-parallel)
    - [Example: Monte Carlo Method to Estimate $\pi$](#example-monte-carlo-method-to-estimate-\pi)
  - [First-Class Tasks](#first-class-tasks)
    - [More flexible construct for parallel computation](#more-flexible-construct-for-parallel-computation)
    - [Task interface](#task-interface)
  - [How Fast are Parallel Programs](#how-fast-are-parallel-programs)
    - [Work and depth](#work-and-depth)
    - [Rules for depth and work](#rules-for-depth-and-work)
    - [Computing time bound for given parallelism](#computing-time-bound-for-given-parallelism)
    - [Parallelism and Amdahl's Law](#parallelism-and-amdahls-law)
  - [Benchmarking Parallel Programs](#benchmarking-parallel-programs)
    - [Testing and Benchmarking](#testing-and-benchmarking)
    - [Benchmarking Parallel Programs](#benchmarking-parallel-programs-1)
    - [Performance Factors](#performance-factors)
    - [Measurement Methodologies](#measurement-methodologies)
    - [ScalaMeter](#scalameter)
    - [Using ScalaMeter](#using-scalameter)
    - [JVM Warmup](#jvm-warmup)
    - [ScalaMeter Warmers](#scalameter-warmers)
    - [ScalaMeter Measures](#scalameter-measures)
- [Week 2 Task-Parallelism](#week-2-task-parallelism)
  - [Parallel Sorting](#parallel-sorting)
    - [Merge Sort](#merge-sort)
    - [Copying the Array](#copying-the-array)
  - [Parallelism And Collections](#parallelism-and-collections)
    - [Functional programming and collections](#functional-programming-and-collections)
    - [Choice of data structures](#choice-of-data-structures)
  - [Parallel Mapping](#parallel-mapping)
    - [Map: meaning and properties](#map-meaning-and-properties)
    - [Map as function on lists](#map-as-function-on-lists)
    - [Sequential map of an array producing an array](#sequential-map-of-an-array-producing-an-array)
    - [Parallel map of an array producing an array](#parallel-map-of-an-array-producing-an-array)
    - [Parallel map on immutable trees](#parallel-map-on-immutable-trees)
    - [Comparison of arrays and immutable trees](#comparison-of-arrays-and-immutable-trees)
  - [Parallel Fold (Reduce) Operation](#parallel-fold-reduce-operation)
    - [Fold (Reduce): meaning and properties](#fold-reduce-meaning-and-properties)
    - [Associative operation](#associative-operation)
    - [Trees for expressions](#trees-for-expressions)
    - [Folding (reducing) trees](#folding-reducing-trees)
    - [Associativity stated as tree reduction](#associativity-stated-as-tree-reduction)
    - [Order of elements in a tree](#order-of-elements-in-a-tree)
    - [Towards a reduction for arrays](#towards-a-reduction-for-arrays)
  - [Associative operation](#associative-operation-1)
    - [Using sum: array norm](#using-sum-array-norm)
    - [Floating point operation](#floating-point-operation)
    - [Associative operations on tuples](#associative-operations-on-tuples)
    - [Example: average](#example-average)
    - [Associativity through symmetry and commutativity](#associativity-through-symmetry-and-commutativity)
  - [Parallel Scan Operation](#parallel-scan-operation)
    - [`scanLeft`: meaning and properties](#scanleft-meaning-and-properties)
    - [Sequential Scan](#sequential-scan)
    - [High-level approach: express `scan` using `map` and `reduce`](#high-level-approach-express-scan-using-map-and-reduce)
    - [Reusing intermediate reduce results by tree](#reusing-intermediate-reduce-results-by-tree)

<!-- /TOC -->

# Week 1 Basics
## JVM and parallelism

  We assume:

  - Our parallel programming model applied for **multicore** or **multiprocessor** systems with shared memory.
  - Operating system and the JVM as the underlying runtim environments.

### Process
  
  **Operating System** : software that manages hardware and software resources, and shedules program executions.
  
  **Process** : an instance of a program that is executing in the OS.
  
  The OS multiplexes many different processes and a limited number of CPUs, so that they get **time slices** of execution. This mechanism is called **multitasking**.
  
  Two different processes cannot access each other's memorty directly, i.e., they are isolated.

### Threads

  **Thread** : Each process can contain multiple independent concurrency units called **threads**.

  Threads can be started from within the same program, and they share the same memory address space.

  Each thread has a program counter and a program stack.

### Creating/Starting Threads

  Each JVM process starts with a **main thread**.

  To start additional threads:
  
  1. Define a Thread subclass.
  2. Instantiate a new Thread object.
  3. Call `start` method on the Thread object.

  The Thread subclass defines the code that the thread will excute. The same custom `Thread` subclass can be used to start multiple threads.

### Example: Starting Threads

  Run in sbt scala console with paste mode.

```
scala> :paste
// Entering paste mode (ctrl-D to finish)
```
```scala
class HelloThread extends Thread {
  override def run() {
    println("Hello world!")
  }
}

val t = new HelloThread

t.start()
t.join()
```

```
Hello world!
defined class HelloThread
t: HelloThread = Thread[Thread-8,5,]
```

  Try 2 threads at the same time.
```scala
class HelloThread extends Thread {
  override def run() {
    println("Hello ")
    println("world!")
  }
}

def main() {
  val t = new HelloThread
  val s = new HelloThread

  t.start()
  s.start()
  t.join()
  s.join()
}
```

```
scala> main()
Hello 
Hello 
world!
world!

scala> main()
Hello 
world!
Hello 
world!

scala> main()
Hello 
world!
Hello 
world!
```

### Atomicity

  The previous demo showed that separate statements in two threads can overlap.

  In some cases, we want to ensure that a sequence of statements in a specific thread executes at once.

  An operation is **atomic** if it appears as if it occurred instantaneously from the point of view of other threads.

  A demo:

```scala
private var uidCount = 0L
def getUniqueId(): Long = {
  uidCount = uidCount + 1
  uidCount
}

def startThread() = {
  val t = new Thread {
    override def run() {
      val uids = for (i <- 0 until 10) yield getUniqueId()
      println(uids)
    }
  }

t.start()
t
}

startThread
startThread
```
```
getUniqueId: ()Long
startThread: ()Thread
res0: Thread = Thread[Thread-53,5,run-main-group-2]

scala> Vector(1, 2, 3, 4, 5, 6, 7, 8, 10, 12)
Vector(1, 7, 9, 11, 13, 14, 15, 16, 17, 18)
```

### The Synchronized Block

  The `synchronized` block is used to achieve atomicity. Code block after a `synchronized` call on an object x is never executed by two threads at the same time.

```scala
 //The synchronized method must be invoded on an instance of some object.
private val x = new AnyRef {}
private var uidCount = 0L
def getUniqueId(): Long = x.synchronized {
  uidCount = uidCount + 1
  uidCount
}

def startThread() = {
  val t = new Thread {
    override def run() {
      val uids = for (i <- 0 until 10) yield getUniqueId()
      println(uids)
    }
  }

t.start()
t
}

startThread
startThread
```

```
Vector(2, 3, 4, 5, 6, 7, 8, 9, 10, 11)
Vector(1, 12, 13, 14, 15, 16, 17, 18, 19, 20)
getUniqueId: ()Long
startThread: ()Thread
res0: Thread = Thread[Thread-3,5,]
```

  Different threads use the synchronized block to agree on unique values.
  The synchronized block is an example of a synchronization primitive.

### Composition with the synchronized block
  
  Invocations of the synchronized block can nest.

```scala
class Account(private var amount: Int = 0) {
  def transfer(target: Account, n: Int) =
    this.synchronized {
      target.synchronized {
        this.amount -= n
        target.amount += n
      }
  }
}


def startThread(a: Account, b: Account, n: Int) = {
  val t = new Thread {
    override def run(): Unit = {
      for (i <- 0 until n) {
        a.transfer(b, 1)
      }
    }
  }
  t.start()
  t
}

val a1 = new Account(500000)
val a2 = new Account(700000)

val t = startThread(a1, a2, 150000)
val s = startThread(a2, a1, 150000)

t.join()
s.join()
```

### Deadlocks

  **Deadlock** is a scenario in which two or more threads compete for resources(such as monitor ownership), and wait for each to finish without releasing the already acquired resources.

  One approach to resolve deadlocks is to always acquire resources in the same order.
  This assume an ordering relationship on the resources.

```scala
private val x = new AnyRef {}
private var uidCount = 0L

def getUniqueId(): Long = {
  uidCount = uidCount + 1
  uidCount
}

class Account(private var amount: Int = 0) {

  val uid = getUniqueId()

  private def lockAndTransfer(target: Account, n: Int) =
    this.synchronized {
      target.synchronized {
        this.amount -= n
        target.amount += n
      }
    }

  def transfer(target: Account, n: Int) =
    if (this.uid < target.uid) this.lockAndTransfer(target, n)
    else target.lockAndTransfer(this, n)
}


def startThread(a: Account, b: Account, n: Int) = {
  val t = new Thread {
    override def run(): Unit = {
      for (i <- 0 until n) {
        a.transfer(b, 1)
      }
    }
  }
  t.start()
  t
}

val a1 = new Account(500000)
val a2 = new Account(700000)

val t = startThread(a1, a2, 150000)
val s = startThread(a2, a1, 150000)

t.join()
s.join()
```

### Memory Model

  Memory model is a set of rules that describes how threads interact when accessing shared memory.

  Java Memory Model - the memory model for the JVM

  Following is two rules of JVM(a subset of the whole rules) which should be remembered in this course:

  1. Two threads writing to separate locations in memory do not need synchronization.
  2. A thread X that calls `join` method on another thread Y is guaranteed to observe all the writes by thread Y after `join` returns.

### Summary

  The parallelism constructs in the remainder of the course are implemented in terms of:

  - threads
  - synchronization primitives such as `synchronized`

## Running Computations in Parallel

### Basic parallel construct
  
  Given expressions `e1` and `e2` , compute them in parallel and return the pair of results

```scala
parallel(e1, e2)
```

### Example: computing p-norm

  Parallelism could be done by a recursive algorithm.

### Signature of parallel

```scala
def parallel[A, B](taskA: =>A, taskB: =>B): (A, B) = {...}
```

  - returns the same value as given
  - benefit: `parallel(a, b)` can be faster than `(a, b)`
  - it takes its arguments as `by name`, indicated with `=> A` and `=> B`

  For parallelism, need to pass unevaluated computations(call `by name`).

### Underlying Hardware Architecture Affects Performance

  Memory is bottleneck. Multi-processors share the memory space of RAM. The computation time can not be less that the time it takes to fetch the data from memory to processors.

### Combining computations of different length with parallel

```scala
val(v1, v2) = parallel(e1, e2)
```

The minimum time that this parallel expression time is the maximum of the running times of `e1` and `e2`.

### Example: Monte Carlo Method to Estimate $\pi$

## First-Class Tasks

### More flexible construct for parallel computation

```scala
val(v1, v2) = parallel(e1, e2)
```

  alternatively using `task` construct:

```scala
val t1 = task(e1)
val t2 = task(e2)
val v1 = t1.join
val v2 = t2.join
```

  `t = task e` starts computation `e` "in the background"

  - `t` is a `task`, which performs computation of `e`
  - current computation proceeds in parallel with `t`
  - to obtain the result of `e`, use `t.join`
  - `t.join` blocks and waits until the results is computed
  - subsequent `t.join` calls quickly return the same result

### Task interface

  Here is a minimal interface for tasks:

```scala
def task(c: => A): Task[A]

trait Task[A] {
  def join: A
}
```

  `task` and `join` establish maps between computations and tasks

  In terms of the value computed the equation `task(e).join==e` holds. We can omit writing `.join` if we also define an `implicit` conversion:

```scala
implicit def getJoin[T](x: Task[T]): T = x.join
```

## How Fast are Parallel Programs

  **Performance** : a key motivation for paralellism

  How to estimate it?

  - empirical measurement
  - asymptotic analysis

  Asymptotic analysis is important to understand how algorithms scale when:

  - inputs get larger
  - we have more hardware parallelism available

  We examine worst-cast(as opposed to average) bounds.

### Work and depth

  We would like to speak about the asymptotic complexity of parallel code
  
  - but this depends on available parallel resources
  - we introduce two measures for a program
  
  Work `W(e)` : number of steps `e` would take if there was no parallelism
  
  - this is simply the sequential execution time
  - treat all `parallel(e1, e2)` as `(e1, e2)`
  
  Depth $D(e)$ : number of steps if we had unbounded parallelism

  - we take maximum of running times for arguments of parallel

### Rules for depth and work

  Key rules are:

  - $W(parallel(e1, e2)) = W(e1) + W(e2) + c2$
  - $D(parallel(e1, e2)) = max(D(e1), D(e2)) + c1$
  
  If we divide work in equal parts, for depth it counts only once!

  For parts of code where we do not use `parallel` explicityly, we must add up costs. For function call or operation $f(e1, ..., en)$:

  - $W(f(e1, ..., en)) = W(e1) + ... + W(en) + W(f)(v1, ..., vn)$
  - $D(f(e1, ..., en)) = D(e1) + ... + D(en) + D(f)(v1, ..., vn)$
  
  $vi$ denotes values of $ei$, if $f$ is primitive operation on *integers*, the $W(f)$ and $D(f)$ ane constant function, regardless of $vi$.

  Note: we assume(reasonably) that constants are such that $D \le W$.

### Computing time bound for given parallelism

  Suppose we know `W(e)` and `D(e)` and our platfrom has `P` parallel threads.

  It is reasonalbe to use this estimate for running time:

  `D(e) + W(e) / P`

### Parallelism and Amdahl's Law

  Suppose that we have two parts of a sequential computaion:

  - part1 takes fraction `f` of the computation time, e.g., 40%.
  - part2 takes the remaining `1 - f` fraction of time, e.g., 60%, and we can speed it up.
  
  If we make part2 `P` times faster the speedup is

  `1 / (f + (1 - f) / P)`

  For `P = 100` and `f = 0.4` we obtain 2.46. Even if we speed the second part infinitely, we can obtain at most `1 / 0.4 = 2.5` speed up.

## Benchmarking Parallel Programs

### Testing and Benchmarking

  **Testing** : ensures that parts of the program are behaving according to the intended behavior.
  
  **Benchmarking** : computes performance metrics for parts of the program.

  Typically, *testing* yields a binary output - a program or its part is either correct or not. *Benchmarking* usually yields a continous value, which denotes the extent to which the program is correct.

### Benchmarking Parallel Programs
  
  - Performance benefits are the main reason why we are writing parallel programs in the first place.
  - Benchmarking parallel programs is even more important than benchmarking sequential programs.

### Performance Factors

  Performance(specifically, running time) is subject to many factors:

  - processor speed
  - number of proce ssors
  - memory access latency and throughput(affectc contention)
  - cache behavior(e.g. false sharing, associativity effects)
  - running behavior(e.g. garbage collection, JIT complication, thread scheduling)

  See [What Every Programmer Should Know About Memory, by Ulrich Drepper][WEPSKAM].

### Measurement Methodologies

  Measuring performance is difficult - usually, the a performance metric is a random variable.

  - multiple repetions
  - statistical treatment: mean and variance
  - eliminating outliers
  - ensuring steady state(warm-up)
  - preventing anomalies(GC, JIT complication, aggressive optimizations)
  
  See [Statistically Rigorous Java Performance Evaluation, by Georges, Buytaert and Eechhout][SRJPE].

### ScalaMeter

  ScalaMeter is a benchmarking and performance regression testing framework of JVM.

  -  performance regression testing: comparing performance of the current program run against known previous runs
  -  benchmarking: measuring performance of the current(part of the) program
  
### Using ScalaMeter

  Add ScalaMeter as a dependency:

```
libraryDependencies += "com.storm-enroute" %% "scalameter-core" % "0.6"
```

  Import the contents of the ScalaMeter package, and measure:

```scala
import org.ScalaMeter._

val time = measure {
  (0 until 1000000).toArray
}

println(s"Array initialization time: $time ms")
```

### JVM Warmup

  When a JVM programs starts, it undergoes a period of warmup, after which it achieves its maximum performance.

  1. first, the program is *interpreted*
  2. then, parts of the program are compiled into machine code
  3. later, the JVM may choose to apply additional dynamic optimizations
  4. eventually, the program reaches *steady state*

### ScalaMeter Warmers

  Usually, we want to measure steady state program performance.

  ScalaMeter Warmer objects run the benchmarked code until detecting steady state.

```scala
import org.scalameter._

val time = withWarmer(new Warmer.Default) measure {
  (0 until 1000000).toArray
}
```

### ScalaMeter Measures

  - `Measure.Default`: plain running time
  - `IgnoringGC`: running time without GC pauses
  - `OutlierElimination`: remove statistical outliers
  - `MemoryFootprint`: memory footprint of an object
  - `GarbageCollectionCycles`: total number of GC pauses
  - newer ScalaMeter version can also measure method invocation counts and boxing counts

```scala
import org.scalameter._

val time = withMeasurer(new Measurer.MemoryFootprint) measure {
  (0 until 1000000).toArray
}
```

# Week 2 Task-Parallelism

## Parallel Sorting

  Sort in parallel

### Merge Sort

  1. recursivley sort the tow halves of the array in parallel
  2. sequentially merge the two array halves by copying into a temporary array
  3. copy the demporary array backi into the original array

```scala
def parMergeSort(xs: Array[Int], maxDepth: Int): Unit = {
  // At each level of the merge sort, we will alternate between the source array xs and the intermediate array ys
  val ys = new Array[Int](xs.length)

  def sort(from: Int, until: Int, depth: Int): Unit = {
    if (depth == maxDepth) {
      // sequential sort
      quickSort(xs, from, until - from)
    } else {
      val mid = (from + until) / 2
      parallel(sort(mid, until, depth + 1), sort(from, mid, depth + 1))
      val flip = (maxDepth - depth) % 2 == 0
      val src = if (flip) ys else xs
      val dst = if (flip) xs else ys
      merge(src, dst, from, mid, until)
    }
  }

  def merge(src: Array[Int], dst: Array[Int], from: Int, mid: Int, until: Int): Unit ={
    // Given an array src consisting of two sorted intervals, merge those interval into the dst array.
    // The merge implementation is sequential, so it will note be gone through here.
    // Exerise: How would you implement merge function in parallel?
  }

  sort(0, xs.length, 0)
}
```

### Copying the Array

```scala
def copy(src: Array[Int], target: Array[Int], from: Int, until: Int, depth: Int): Unit = {
  if (depth == maxDepth) {
    Array.copy(src, from, targt, from, until - from)
  } else {
    val mid = (from + until) / 2
    val right = parallel(
      copy(src, target, mid, until, depth + 1)
      copy(src, target, from, mid, depth + 1)
    )
  }
}

if (maxDepth % 2 == 0) copy(ys, xs, 0, xs.length, 0)
```

## Parallelism And Collections

  Parallel processing of collections is important. It is one of the main applications of parallelsim today.

  We examine conditions when this can be done:

  - properties of collections: ablility to split, combine
  - properties of operations: associativity, independence

### Functional programming and collections

  Opertations on collections are key to functional programming

  `map`: apply function to each element

  - `List(1,3,8).map(x => x*x) == List(1,9,64)`

  `fold`: combine elements with a given collection

  - `List(1,3,8).fold(100)((s,x) => s + x) == 112`

  `scan`: combine folds of all list prefixes

  - `List(1,3,8).scan(100)((s,x) => s + x) == List(100, 101, 104, 112)`

### Choice of data structures

  We use `List` to speicfy the results of operations. Lists are not good for parallel implementations because we cannot efficiently

  - split them in half (need to search for the middle)
  - combine them (concatenation needs linear time)

  We use for now these alternatives:

  - `arrays`: imperative (recall array sum)
  - `trees`: can be implemented functionally

## Parallel Mapping

### Map: meaning and properties

  Map applies a given function to each list element

  `List(a1, a2, ..., a3).map(f) == List(f(a1), f(a2), ..., f(an))`

  Properties to keep in mind:

  - `list.map(x => x) == list`
  - `list.map(f.compose(g)) = list.map(g).map(f)` 

  Recall `(f.compose(g))(x) == f(g(x))`

### Map as function on lists

```scala
// sequential definition
def mapSeq[A, B](lst: List[A], f: A => B): List[B] = lst match {
  case Nil => Nil
  case h :: t => f(h) :: mapSwq(t, f)
}
```

  We would like a version that parallelizes

  - computations of `f(h)` for different elements `h`
  - finding the elements themselves (list is not a good choice)

### Sequential map of an array producing an array

```scala
def mapASegSeq[A,B](inp: Array[A], left: Int, right: Int, f: A => B, out: Array[B]): Unit = {
  // Writes to out(i) for left <= i <= right-1
  var i = left
  while (i < right) {
    out(i) = f(inp(i))
    i = i+1
  }
}
```

### Parallel map of an array producing an array

```scala
def mapASegPar[A,B](inp: Array[A], left: Int, right: Int, f: A => B, out: Array[B]): Unit = {
  // Writes to out(i) for left <= i <= right-1
  if (right - left < threshold)
    mapASegSeq(inp, left, right, f, out)
  else {
    val mid = left + (right - left) / 2
    parallel(
      mapASegPar(inp, left, mid, f, out),
      mapASegPar(inp, mid, right, f, out)
    )
  }
}
```

  - writes need to be disjoint (otherwise: non-deterministic behavior)
  - threshold needs to be large enough (otherwise we lose efficiency)

### Parallel map on immutable trees

  Consider trees where

  - leaves store array segments
  - non-leaf node stores number of elements

  We assume that our tress are balanced: we can explore branches in parallel

```scala
sealed abstract class Tree[A] { val size: Int }
case class Leaf[A](a: Array[A]) extends Tree[A] {
  override val size = a.size
}
case class Node[A](l: Tree[A], r: Tree[A]) extends Tree[A] {
  override val size = l.size + r.size
}

def mapTreePar[A:Manifest,B:Manifest](t: Tree[A], f: A => B): Tree[B] = t match {
  case Leaf(a) => {
    val len = a.length; val b = new Array[B](len)
    var i = 0
    while (i < len) { b(i) = f(a(i)); i = i + 1 }
    Leaf(b)
  }
  case Node(l, r) => {
    val (lb,rb) = parallel(mapTreePar(l,f), mapTreePar(r,f))
    Node(lb,rb)
  }
}
```

### Comparison of arrays and immutable trees

  Arrays

  - (+) random access to elements, on shared memory can share array
  - (+) good memory locality
  - (-) imperative: must ensure parallel tasks write to disjoint parts
  - (-) expensive to concatenate

  Immutable trees:

  - (+) purely functional, produce new trees, keep old ones
  - (+) no need to worry about disjointness of writes by parallel tasks
  - (+) efficient to combine two trees
  - (-) high memory allocation overhead
  - (-) bad locality

## Parallel Fold (Reduce) Operation

### Fold (Reduce): meaning and properties

  Fold combines elements with a given operation

  `List(1,3,8).fold(100)((s,x) => s + x) == 112`

  Fold takes among others a binary operation, but variants differ:

  - whether they take an initial element or assume non-empty list
  - in which order they combine operations of collection

  `List(1,3,8).foldLeft(100)((s,x) => s - x) == ((100 - 1) - 3) - 8 == 88`

  `List(1,3,8).foldRight(100)((s,x) => s - x) == 1 - (3 - (8 - 100)) == -94`

  `List(1,3,8).reduceLeft((s,x) => s - x) == (1 - 3) - 8 == -10`

  `List(1,3,8).reduceRight(100)((s,x) => s - x) == 1 - (3 - 8) == 6`

  To enable parallel operations, we look at **associative** operations

  - addiction, string concatenation (but not minus)

### Associative operation

  Operation `f: (A,A) => A` is **associative** iff for every $x, y, z$:

  $f(x, f(y, z)) = f(f(x, y), z)$

  If we write $f(a, b)$ in infix form as $a \otimes b$, associativity becomes

  $x \otimes (y \otimes z) = (x \otimes y) \otimes z$

  Consequence: consider two expressions with same list of operands connected with $\otimes$, but different parentheses. Then these expressions evaluate to the same result, for exmaple:

  $(x \otimes y) \otimes (z \otimes w) = (x \otimes (y \otimes z)) \otimes w = ((x \otimes y) \otimes z) \otimes w$

### Trees for expressions

  Each expression built from values connected with $\otimes$ can be represented as a tree

  - leaves are the values
  - nodes are $\otimes$

### Folding (reducing) trees

  Result of evaluating the expression is given by a reduce of this tree.

  Sequential Version:

```scala
// sequential definition
// reduce can be thought of as relacing the constructor Node with given f
def reduce[A](t: Tree[A], f: (A,A) => A): A = t match {
  case Leaf(v) => v
  case Node(l, r) => f(reduce[A](l, f), reduce[A](r, f))  // Node -> f
}
```

  Parallel Version:
```scala
// parallel definition
def reduce[A](t: Tree[A], f: (A,A) => A): A = t match {
  case Leaf(v) => v
  case Node(l, r) => {
    val (lV, rV) = parallel(reduce[A](l, f), reduce[A](r, f))
    f(lV, rV)
  }
}
```

### Associativity stated as tree reduction

  If `f` denotes $\otimes$, in Scala we can write this also as:

```scala
reduce(Node(Leaf(x), Node(Leaf(y), Leaf(z))), f) == 
reduce(Node(Node(Leaf(x), Leaf(y)), Leaf(z)), f)
```

### Order of elements in a tree

  Observe: can use a list to describe the ordering of elements of a tree

```scala
def toList[A](t: Tree[A]): List[A] = t match {
  case Leaf(v) => List(v)
  case Node(l, r) => toList[A](l) ++ toList[A](r)
}
```

  Suppose we also have tree map:

```scala
def map[A,B](t: Tree[A], f: A => B): Tree[B] = t match {
  case Leaf(v) => Leaf(f(v))
  case Node(l, r) => Node(map[A,B](l, f), map[A,B](r, f))
}
```

  Express `toList` using `map` and `reduce`:

  `toList(t) == reduce(map(t, List(_)), _ ++ _)`

  Express associativety results consistency up to element order in Scala:

  Consequence of associativity(Scala): if `f: (A,A) => A` is associative, `t1: Tree[A]` and `t2: Tree[B]` satisfies `toList(t1) == toList(t2)`, then: `reduce(t1, f) == reduce(t2, f)`.

### Towards a reduction for arrays

  Often we work with collections where we only know the ordering and not the tree structure, e.g., arrays. We can convert it into a balanced tree then do tree reduction.

  Because of associativety, we can choose any tree that preserves the order of elements of the original collection.

  Tree reduction replaces `Node` constructor with `f`, so we can just use `f` directly instead of building tree nodes.

  When the segment is small, it is faster to process it sequentially.

```scala
// Parallel array reduce
// Implementation that computes fold of a given array segment with a given associative operation f
def reduceSeg[A](inp: Array[A], left: Int, right: Int, f: (A,A) => A): A = {
  if (right - left < threshold) {
    var res = inp(left); var i = left + 1
    while (i < right) { res = f(res, inp(i)); i = i + 1 }
    res
  } else {
    mid = (left + right) / 2
    val (a1, a2) = parallel(
      reduceSeg(inp, left, mid, f),
      reduceSeg(inp, mid, rigt, f)
    )
    f(a1, a2)
  }
}

def reduce[A](inp: Array[A], f: (A,A) => A): A = 
  reduceSeg(inp, 0, inp.length, f)
```

## Associative operation

  Operation `f: (A,A) => A` is **associative** iff for every $x, y, z$:

  $f(x, f(y, z)) = f(f(x, y), z)$

  Operation `f: (A,A) => A` is **commutative** iff for every $x, y$:

  $f(x, y) = f(y, x)$

  For correctness of **reduce**, we need (just) associativity.

### Using sum: array norm

  $\sum_{i=s}^{t-1} |a_i|^p$ corresponds to `reduce(map(a, power(abs(_), p)), _ + _)`.

  `map` can be used together with `reduce` to avoid intermediate collections.

### Floating point operation

  Addition is commutative but not associative

```scala
scala> val e = 1e-200
e: Double = 1.0E-200
scala> val x = 1e200
x: Double = 1.0E200
scala> val mx = -x
mx: Double = -1.0E200

scala> (x + mx) + e
res0: Double = 1.0E-200
scala> x + (mx + e)
res1: Double = 0.0
scala> (x + mx) + e = x + (mx + e)
res2: Boolean = false
```

  Multiplication is commutative but not associative

```scala
scala> (e * x) * x
res3: Double = 1.0E200
scala> e * (x * x)
res4: Double = Infinity
scala> (e * x) * x == e * (x * x)
res5: Boolean = false
```

### Associative operations on tuples

  Suppose `f1: (A1,A1) => A1` and `f2, (A2,A2) => A2` are associative.

  Then `f: ((A1,A2),(A1,A2)) => (A1,A2)` defined by

  `f((x1,x2),(y1,y2)) = (f1(x1,y1),f2(x2,y2))`

  is also associative.

  It's similar to construct associative operations on for n-tuples.

### Example: average

  Given a collction of integers, compute the average.

```scala
// Two reduction solution.
val sum = reduce(collection, _ + _)
val length = reduce(map(collection, (x: Int) => 1), _ + _)
sum / length

// single reduction solution.
// f((sum1, length1), (sum2, length2)) = (sum1 + sum2, length1 + length2)
val (sum, length) = reduce(map(collection, (x: Int) => (x, 1), f)
sum / length
```

### Associativity through symmetry and commutativity

  If `f` satisfies `f(f(x,y),z) = f(f(y,z),x)`(symmetry) and `f(x,y) = f(y,x)`(commutativity) for every `x, y, z`, we have `f(f(x,y),z) = f(x, f((y,z)))`(associativity).

## Parallel Scan Operation

### `scanLeft`: meaning and properties

  `scanLeft`: list of the folds of all list prefixes

  `List(1,3,8).scanLeft(100)((s,x) => s + x) == List(100, 101, 104, 112)`

  `scanRight` is different from `scanLeft`

  `List(1,3,8).scanRight(100)((s,x) => s + x) == List(112, 104, 101, 100)`

### Sequential Scan

  $List(a_1, a_2, a_3, ..., a_N).scanLeft(a_0)(f) = List(b_0, b_1, b_2, ..., b_N)$

  where $b_0 = a_0$ and $b_i = f(b_{i-1}, a_i)$ for $1 \le i \le N$. We also assume $f$ is associative in the remaining of this segment.

```scala
// Sequential scanLeft.
// Assume out.length >= inp.length + 1.
def scanLeft[A](inp: Array[A], a0: A, f: (A,A) => A, out: Array[A]): Unit = {
  out(0) = a0
  var a = a0
  var i = 0
  while (i < inp.length) {
    a = f(a,inp(i))
    i = i + 1
    out(i) = a
  }
}
```

### High-level approach: express `scan` using `map` and `reduce`

  Assume input is given in array `inp` and that you have `reduceSeg1` and `mapSeg` functions on array segments:

```scala
def reduceSeg1[A](inp: Array[A], left: Int, right: Int, a0: A, f: (A,A) => A): A

def mapSeg[A,B](inp: Array[A], left: Int, right: Int, fi: (Int, A) => B, out: Array[B]): Unit

def scanLeft[A](inp: Array[A], a0: A, f: (A,A) => A, out: Array[A]): Unit = {
  val fi = { (i: Int, v: A) => reduceSeg1(inp, 0, i, a0, f) }
  mapSeg(inp, 0, inp.length, fi, out)
  val last = inp.length - 1
  out(last + 1) = f(out(last), inp(last))
}
```

### Reusing intermediate reduce results by tree

  Reuse computations by saving the intermediate results in a tree. We assume that input collection is also (another) tree.

```scala
// Trees storing the input collection only have values in leaves
sealed abstract class Tree[A]
case class Leaf[A](a: A) extends Tree[A]
case class Node[A](l: Tree[A], r: Tree[A]) extends Tree[A]

// Trees storing intermediate values have (res) value in nodes
sealed abstract class TreeRes[A] { val res: A }
case class LeafRes[A](override val res: A) extends TreeRes[A]
case class NodeRes[A[(l: TreeRes[A], override val res: A, r: TreeRes[A])

def reduceRes[A](t: Tree[A], f: (A,A) => A): TreeRes[A] = t match {
  case Leaf(v): LeafRes(v)
  case Node(l, r): {
    val (tL, tR) = (reduceRes(l, f), reduceRes(r, f))
    val res =  f(tL.res, tR.res)
    NodeRes(lt, res, rt)
  }
}

```scala
// Parallel version for reduceRes
def upsweep[A](t: Tree[A], f: (A,A) => A): TreeRes[A] = t match {
  case Leaf(v) => LeafRes(v)
  case Node(l, r) => {
    val (tL, tR) = parallel(upsweep(l, f), upsweep(r, f))
    val res =  f(tL.res, tR.res)
    NodeRes(lt, res, rt)
  }
}
```

### Create final collection from tree

```scala
// a0 is reduce of all elements left of the tree
def downsweep[A](t: TreeRes[A], a0: A, f: (A, A) => A): Tree[A] = t match {
  case LeafRes(a) => Leaf(f(a0, a))
  case NodeRes(l, _, r) => {
    val (tL, tR) = parallel(
      downsweep(l, a0, f),
      downsweep(r, f(a0, l.res), f)
    )
    Node(tL, tR)
  }
}
```

### `scanLeft` on trees

```scala
def scanLeft[A](t: Tree[A], a0: A, f: (A,A) => A): Tree[A] = {
  val tRes = upsweep(t, f)
  val scan1 = downsweep(tRes, a0, f)
  prepend(a0, scan1)
}

def prepend[A](x: A, t: Tree[A]): Tree[A] = t match {
  case Leaf(v) => Node(Leaf(x), Leaf(v))
  case Node(l, r) => Node(prepend(x, l), r)
}
```

### Array reduce by tree

  The only difference compared to previouse `TreeRes`: each Leaf now keeps track of the array segment range `(from, to)` from which `res` is computed.

```scala
// Intermediate tree for array reduce
// We do not keep track of the array elements in the Leaf itself.
// We instead pass around a reference to the input array.
sealed abstract class TreeResA[A] { val res: A }
case class Leaf[A](from: Int, to: Int, override val res: A) extends TreeResA[A]
case class Node[A](l: TreeResA[A], override val res: A, r: TreeResA[A]) extends TreeResA[A]

def upsweep[A](inp: Array[A], from: Int, to: Int, f: (A,A) => A): TreeResA[A] = {
  if (to - from < threshold)
    Leaf(from, to, reduceSeg1(inp, from + 1, to, inp(from), f))
  else {
    val mid = from + (to - from) / 2
    val (tL, tR) = parallel(
      upsweep(inp, from, mid, f),
      upsweep(inp, mid, to, f)
    )
    Node(tL, f(tL.res, rR.res), tR)
  }
}

def downsweep[A](inp: Array[A], a0: A, f: (A,A) => A, t: TreeResA[A], out: Array[A]): Tree[A] = t macth {
  case Leaf(from, to, res) => scanLeftSeg(inp, from, to, a0, f, out)
  case Node(l, _, r) => {
    val (_,_) = parallel(
      downsweep(inp, a0, f, l, out),
      downsweep(inp, f(a0, l.res), f, r, out)
    )
  }
}

def scanLeft[A](inp: Array[A], a0: A, f: (A,A) => A, out: Array[A]) = {
  val t = upsweep(inp, 0, inp.length, f)
  downsweep(inp, a0, f, t, out)  // fills out [1..inp.length]
  out(0) = a0
}
```

# 3. Week 3 Data-ParaLLelism
# 4. Week 4 Data Structures


[WEPSKAM]: https://www.akkadia.org/drepper/cpumemory.pdf
[SRJPE]: https://dri.es/files/oopsla07-georges.pdf