Parallel Programming
====

These are some notes for the 3rd course of Scala Specialization on Cousera.

<!-- TOC -->

- [1. Week 1 Basics](#1-week-1-basics)
  - [1.1. JVM and parallelism](#11-jvm-and-parallelism)
    - [1.1.1. Process](#111-process)
    - [1.1.2. Threads](#112-threads)
    - [1.1.3. Creating/Starting Threads](#113-creatingstarting-threads)
    - [1.1.4. Example: Starting Threads](#114-example-starting-threads)
    - [1.1.5. Atomicity](#115-atomicity)
    - [1.1.6. The Synchronized Block](#116-the-synchronized-block)
    - [1.1.7. Composition with the synchronized block](#117-composition-with-the-synchronized-block)
    - [1.1.8. Deadlocks](#118-deadlocks)
    - [1.1.9. Memory Model](#119-memory-model)
    - [1.1.10. Summary](#1110-summary)
  - [1.2. Running Computations in Parallel](#12-running-computations-in-parallel)
    - [1.2.1. Basic parallel construct](#121-basic-parallel-construct)
    - [1.2.2. Example: computing p-norm](#122-example-computing-p-norm)
    - [1.2.3. Signature of parallel](#123-signature-of-parallel)
    - [1.2.4. Underlying Hardware Architecture Affects Performance](#124-underlying-hardware-architecture-affects-performance)
    - [1.2.5. Combining computations of different length with parallel](#125-combining-computations-of-different-length-with-parallel)
    - [1.2.6. Example: Monte Carlo Method to Estimate $\pi$](#126-example-monte-carlo-method-to-estimate-\pi)
  - [1.3. First-Class Tasks](#13-first-class-tasks)
    - [1.3.1. More flexible construct for parallel computation](#131-more-flexible-construct-for-parallel-computation)
    - [1.3.2. Task interface](#132-task-interface)
  - [1.4. How Fast are Parallel Programs](#14-how-fast-are-parallel-programs)
    - [1.4.1. Work and depth](#141-work-and-depth)
    - [1.4.2. Rules for depth and work](#142-rules-for-depth-and-work)
    - [1.4.3. Computing time bound for given parallelism](#143-computing-time-bound-for-given-parallelism)
    - [1.4.4. Parallelism and Amdahl's Law](#144-parallelism-and-amdahls-law)
  - [1.5. Benchmarking Parallel Programs](#15-benchmarking-parallel-programs)
    - [1.5.1. Testing and Benchmarking](#151-testing-and-benchmarking)
    - [1.5.2. Benchmarking Parallel Programs](#152-benchmarking-parallel-programs)
    - [1.5.3. Performance Factors](#153-performance-factors)
    - [1.5.4. Measurement Methodologies](#154-measurement-methodologies)
    - [1.5.5. ScalaMeter](#155-scalameter)
    - [1.5.6. Using ScalaMeter](#156-using-scalameter)
    - [1.5.7. JVM Warmup](#157-jvm-warmup)
    - [1.5.8. ScalaMeter Warmers](#158-scalameter-warmers)
    - [1.5.9. ScalaMeter Measures](#159-scalameter-measures)
- [2. Week 2 Task-Parallelism](#2-week-2-task-parallelism)
  - [2.1. Parallel Sorting](#21-parallel-sorting)
    - [2.1.1. Merge Sort](#211-merge-sort)
    - [2.1.2. Copying the Array](#212-copying-the-array)
  - [2.2. Parallelism And Collections](#22-parallelism-and-collections)
    - [2.2.1. Functional programming and collections](#221-functional-programming-and-collections)
    - [2.2.2. Choice of data structures](#222-choice-of-data-structures)
  - [2.3. Parallel Mapping](#23-parallel-mapping)
    - [2.3.1. Map: meaning and properties](#231-map-meaning-and-properties)
    - [2.3.2. Map as function on lists](#232-map-as-function-on-lists)
    - [2.3.3. Sequential map of an array producing an array](#233-sequential-map-of-an-array-producing-an-array)
    - [2.3.4. Parallel map of an array producing an array](#234-parallel-map-of-an-array-producing-an-array)
    - [2.3.5. Parallel map on immutable trees](#235-parallel-map-on-immutable-trees)
    - [2.3.6. Comparison of arrays and immutable trees](#236-comparison-of-arrays-and-immutable-trees)
  - [2.4. Parallel Fold (Reduce) Operation](#24-parallel-fold-reduce-operation)
    - [2.4.1. Fold (Reduce): meaning and properties](#241-fold-reduce-meaning-and-properties)
    - [2.4.2. Associative operation](#242-associative-operation)
    - [2.4.3. Trees for expressions](#243-trees-for-expressions)
    - [2.4.4. Folding (reducing) trees](#244-folding-reducing-trees)
    - [2.4.5. Associativity stated as tree reduction](#245-associativity-stated-as-tree-reduction)
    - [2.4.6. Order of elements in a tree](#246-order-of-elements-in-a-tree)
    - [2.4.7. Towards a reduction for arrays](#247-towards-a-reduction-for-arrays)
  - [2.5. Associative operation](#25-associative-operation)
    - [2.5.1. Using sum: array norm](#251-using-sum-array-norm)
    - [2.5.2. Floating point operation](#252-floating-point-operation)
    - [2.5.3. Associative operations on tuples](#253-associative-operations-on-tuples)
    - [2.5.4. Example: average](#254-example-average)
    - [2.5.5. Associativity through symmetry and commutativity](#255-associativity-through-symmetry-and-commutativity)
  - [2.6. Parallel Scan Operation](#26-parallel-scan-operation)
    - [2.6.1. `scanLeft`: meaning and properties](#261-scanleft-meaning-and-properties)
    - [2.6.2. Sequential Scan](#262-sequential-scan)
    - [2.6.3. High-level approach: express `scan` using `map` and `reduce`](#263-high-level-approach-express-scan-using-map-and-reduce)
- [3. Week 3 Data-ParaLLelism](#3-week-3-data-parallelism)
- [4. Week 4 Data Structures](#4-week-4-data-structures)

<!-- /TOC -->

# 1. Week 1 Basics
## 1.1. JVM and parallelism

  We assume:

  - Our parallel programming model applied for **multicore** or **multiprocessor** systems with shared memory.
  - Operating system and the JVM as the underlying runtim environments.

### 1.1.1. Process
  
  **Operating System** : software that manages hardware and software resources, and shedules program executions.
  
  **Process** : an instance of a program that is executing in the OS.
  
  The OS multiplexes many different processes and a limited number of CPUs, so that they get **time slices** of execution. This mechanism is called **multitasking**.
  
  Two different processes cannot access each other's memorty directly, i.e., they are isolated.

### 1.1.2. Threads

  **Thread** : Each process can contain multiple independent concurrency units called **threads**.

  Threads can be started from within the same program, and they share the same memory address space.

  Each thread has a program counter and a program stack.

### 1.1.3. Creating/Starting Threads

  Each JVM process starts with a **main thread**.

  To start additional threads:
  
  1. Define a Thread subclass.
  2. Instantiate a new Thread object.
  3. Call `start` method on the Thread object.

  The Thread subclass defines the code that the thread will excute. The same custom `Thread` subclass can be used to start multiple threads.

### 1.1.4. Example: Starting Threads

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

### 1.1.5. Atomicity

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

### 1.1.6. The Synchronized Block

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

### 1.1.7. Composition with the synchronized block
  
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

### 1.1.8. Deadlocks

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

### 1.1.9. Memory Model

  Memory model is a set of rules that describes how threads interact when accessing shared memory.

  Java Memory Model - the memory model for the JVM

  Following is two rules of JVM(a subset of the whole rules) which should be remembered in this course:

  1. Two threads writing to separate locations in memory do not need synchronization.
  2. A thread X that calls `join` method on another thread Y is guaranteed to observe all the writes by thread Y after `join` returns.

### 1.1.10. Summary

  The parallelism constructs in the remainder of the course are implemented in terms of:

  - threads
  - synchronization primitives such as `synchronized`

## 1.2. Running Computations in Parallel

### 1.2.1. Basic parallel construct
  
  Given expressions `e1` and `e2` , compute them in parallel and return the pair of results

```scala
parallel(e1, e2)
```

### 1.2.2. Example: computing p-norm

  Parallelism could be done by a recursive algorithm.

### 1.2.3. Signature of parallel

```scala
def parallel[A, B](taskA: =>A, taskB: =>B): (A, B) = {...}
```

  - returns the same value as given
  - benefit: `parallel(a, b)` can be faster than `(a, b)`
  - it takes its arguments as `by name`, indicated with `=> A` and `=> B`

  For parallelism, need to pass unevaluated computations(call `by name`).

### 1.2.4. Underlying Hardware Architecture Affects Performance

  Memory is bottleneck. Multi-processors share the memory space of RAM. The computation time can not be less that the time it takes to fetch the data from memory to processors.

### 1.2.5. Combining computations of different length with parallel

```scala
val(v1, v2) = parallel(e1, e2)
```

The minimum time that this parallel expression time is the maximum of the running times of `e1` and `e2`.

### 1.2.6. Example: Monte Carlo Method to Estimate $\pi$

## 1.3. First-Class Tasks

### 1.3.1. More flexible construct for parallel computation

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

### 1.3.2. Task interface

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

## 1.4. How Fast are Parallel Programs

  **Performance** : a key motivation for paralellism

  How to estimate it?

  - empirical measurement
  - asymptotic analysis

  Asymptotic analysis is important to understand how algorithms scale when:

  - inputs get larger
  - we have more hardware parallelism available

  We examine worst-cast(as opposed to average) bounds.

### 1.4.1. Work and depth

  We would like to speak about the asymptotic complexity of parallel code
  
  - but this depends on available parallel resources
  - we introduce two measures for a program
  
  Work `W(e)` : number of steps `e` would take if there was no parallelism
  
  - this is simply the sequential execution time
  - treat all `parallel(e1, e2)` as `(e1, e2)`
  
  Depth $D(e)$ : number of steps if we had unbounded parallelism

  - we take maximum of running times for arguments of parallel

### 1.4.2. Rules for depth and work

  Key rules are:

  - $W(parallel(e1, e2)) = W(e1) + W(e2) + c2$
  - $D(parallel(e1, e2)) = max(D(e1), D(e2)) + c1$
  
  If we divide work in equal parts, for depth it counts only once!

  For parts of code where we do not use `parallel` explicityly, we must add up costs. For function call or operation $f(e1, ..., en)$:

  - $W(f(e1, ..., en)) = W(e1) + ... + W(en) + W(f)(v1, ..., vn)$
  - $D(f(e1, ..., en)) = D(e1) + ... + D(en) + D(f)(v1, ..., vn)$
  
  $vi$ denotes values of $ei$, if $f$ is primitive operation on *integers*, the $W(f)$ and $D(f)$ ane constant function, regardless of $vi$.

  Note: we assume(reasonably) that constants are such that $D \le W$.

### 1.4.3. Computing time bound for given parallelism

  Suppose we know `W(e)` and `D(e)` and our platfrom has `P` parallel threads.

  It is reasonalbe to use this estimate for running time:

  `D(e) + W(e) / P`

### 1.4.4. Parallelism and Amdahl's Law

  Suppose that we have two parts of a sequential computaion:

  - part1 takes fraction `f` of the computation time, e.g., 40%.
  - part2 takes the remaining `1 - f` fraction of time, e.g., 60%, and we can speed it up.
  
  If we make part2 `P` times faster the speedup is

  `1 / (f + (1 - f) / P)`

  For `P = 100` and `f = 0.4` we obtain 2.46. Even if we speed the second part infinitely, we can obtain at most `1 / 0.4 = 2.5` speed up.

## 1.5. Benchmarking Parallel Programs

### 1.5.1. Testing and Benchmarking

  **Testing** : ensures that parts of the program are behaving according to the intended behavior.
  
  **Benchmarking** : computes performance metrics for parts of the program.

  Typically, *testing* yields a binary output - a program or its part is either correct or not. *Benchmarking* usually yields a continous value, which denotes the extent to which the program is correct.

### 1.5.2. Benchmarking Parallel Programs
  
  - Performance benefits are the main reason why we are writing parallel programs in the first place.
  - Benchmarking parallel programs is even more important than benchmarking sequential programs.

### 1.5.3. Performance Factors

  Performance(specifically, running time) is subject to many factors:

  - processor speed
  - number of proce ssors
  - memory access latency and throughput(affectc contention)
  - cache behavior(e.g. false sharing, associativity effects)
  - running behavior(e.g. garbage collection, JIT complication, thread scheduling)

  See [What Every Programmer Should Know About Memory, by Ulrich Drepper][WEPSKAM].

### 1.5.4. Measurement Methodologies

  Measuring performance is difficult - usually, the a performance metric is a random variable.

  - multiple repetions
  - statistical treatment: mean and variance
  - eliminating outliers
  - ensuring steady state(warm-up)
  - preventing anomalies(GC, JIT complication, aggressive optimizations)
  
  See [Statistically Rigorous Java Performance Evaluation, by Georges, Buytaert and Eechhout][SRJPE].

### 1.5.5. ScalaMeter

  ScalaMeter is a benchmarking and performance regression testing framework of JVM.

  -  performance regression testing: comparing performance of the current program run against known previous runs
  -  benchmarking: measuring performance of the current(part of the) program
  
### 1.5.6. Using ScalaMeter

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

### 1.5.7. JVM Warmup

  When a JVM programs starts, it undergoes a period of warmup, after which it achieves its maximum performance.

  1. first, the program is *interpreted*
  2. then, parts of the program are compiled into machine code
  3. later, the JVM may choose to apply additional dynamic optimizations
  4. eventually, the program reaches *steady state*

### 1.5.8. ScalaMeter Warmers

  Usually, we want to measure steady state program performance.

  ScalaMeter Warmer objects run the benchmarked code until detecting steady state.

```scala
import org.scalameter._

val time = withWarmer(new Warmer.Default) measure {
  (0 until 1000000).toArray
}
```

### 1.5.9. ScalaMeter Measures

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

# 2. Week 2 Task-Parallelism

## 2.1. Parallel Sorting

  Sort in parallel

### 2.1.1. Merge Sort

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

### 2.1.2. Copying the Array

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

## 2.2. Parallelism And Collections

  Parallel processing of collections is important. It is one of the main applications of parallelsim today.

  We examine conditions when this can be done:

  - properties of collections: ablility to split, combine
  - properties of operations: associativity, independence

### 2.2.1. Functional programming and collections

  Opertations on collections are key to functional programming

  `map`: apply function to each element

  - `List(1,3,8).map(x => x*x) == List(1,9,64)`

  `fold`: combine elements with a given collection

  - `List(1,3,8).fold(100)((s,x) => s + x) == 112`

  `scan`: combine folds of all list prefixes

  - `List(1,3,8).scan(100)((s,x) => s + x) == List(100, 101, 104, 112)`

### 2.2.2. Choice of data structures

  We use `List` to speicfy the results of operations. Lists are not good for parallel implementations because we cannot efficiently

  - split them in half (need to search for the middle)
  - combine them (concatenation needs linear time)

  We use for now these alternatives:

  - `arrays`: imperative (recall array sum)
  - `trees`: can be implemented functionally

## 2.3. Parallel Mapping

### 2.3.1. Map: meaning and properties

  Map applies a given function to each list element

  `List(a1, a2, ..., a3).map(f) == List(f(a1), f(a2), ..., f(an))`

  Properties to keep in mind:

  - `list.map(x => x) == list`
  - `list.map(f.compose(g)) = list.map(g).map(f)` 

  Recall `(f.compose(g))(x) == f(g(x))`

### 2.3.2. Map as function on lists

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

### 2.3.3. Sequential map of an array producing an array

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

### 2.3.4. Parallel map of an array producing an array

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

### 2.3.5. Parallel map on immutable trees

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

### 2.3.6. Comparison of arrays and immutable trees

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

## 2.4. Parallel Fold (Reduce) Operation

### 2.4.1. Fold (Reduce): meaning and properties

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

### 2.4.2. Associative operation

  Operation `f: (A,A) => A` is **associative** iff for every $x, y, z$:

  $f(x, f(y, z)) = f(f(x, y), z)$

  If we write $f(a, b)$ in infix form as $a \otimes b$, associativity becomes

  $x \otimes (y \otimes z) = (x \otimes y) \otimes z$

  Consequence: consider two expressions with same list of operands connected with $\otimes$, but different parentheses. Then these expressions evaluate to the same result, for exmaple:

  $(x \otimes y) \otimes (z \otimes w) = (x \otimes (y \otimes z)) \otimes w = ((x \otimes y) \otimes z) \otimes w$

### 2.4.3. Trees for expressions

  Each expression built from values connected with $\otimes$ can be represented as a tree

  - leaves are the values
  - nodes are $\otimes$

### 2.4.4. Folding (reducing) trees

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

### 2.4.5. Associativity stated as tree reduction

  If `f` denotes $\otimes$, in Scala we can write this also as:

```scala
reduce(Node(Leaf(x), Node(Leaf(y), Leaf(z))), f) == 
reduce(Node(Node(Leaf(x), Leaf(y)), Leaf(z)), f)
```

### 2.4.6. Order of elements in a tree

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

### 2.4.7. Towards a reduction for arrays

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

## 2.5. Associative operation

  Operation `f: (A,A) => A` is **associative** iff for every $x, y, z$:

  $f(x, f(y, z)) = f(f(x, y), z)$

  Operation `f: (A,A) => A` is **commutative** iff for every $x, y$:

  $f(x, y) = f(y, x)$

  For correctness of **reduce**, we need (just) associativity.

### 2.5.1. Using sum: array norm

  $\sum_{i=s}^{t-1} |a_i|^p$ corresponds to `reduce(map(a, power(abs(_), p)), _ + _)`.

  `map` can be used together with `reduce` to avoid intermediate collections.

### 2.5.2. Floating point operation

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

### 2.5.3. Associative operations on tuples

  Suppose `f1: (A1,A1) => A1` and `f2, (A2,A2) => A2` are associative.

  Then `f: ((A1,A2),(A1,A2)) => (A1,A2)` defined by

  `f((x1,x2),(y1,y2)) = (f1(x1,y1),f2(x2,y2))`

  is also associative.

  It's similar to construct associative operations on for n-tuples.

### 2.5.4. Example: average

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

### 2.5.5. Associativity through symmetry and commutativity

  If `f` satisfies `f(f(x,y),z) = f(f(y,z),x)`(symmetry) and `f(x,y) = f(y,x)`(commutativity) for every `x, y, z`, we have `f(f(x,y),z) = f(x, f((y,z)))`(associativity).

## 2.6. Parallel Scan Operation

### 2.6.1. `scanLeft`: meaning and properties

  `scanLeft`: list of the folds of all list prefixes

  `List(1,3,8).scanLeft(100)((s,x) => s + x) == List(100, 101, 104, 112)`

  `scanRight` is different from `scanLeft`

  `List(1,3,8).scanRight(100)((s,x) => s + x) == List(112, 104, 101, 100)`

### 2.6.2. Sequential Scan

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

### 2.6.3. High-level approach: express `scan` using `map` and `reduce`

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
```

# 3. Week 3 Data-ParaLLelism
# 4. Week 4 Data Structures


[WEPSKAM]: https://www.akkadia.org/drepper/cpumemory.pdf
[SRJPE]: https://dri.es/files/oopsla07-georges.pdf