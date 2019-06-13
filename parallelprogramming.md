Parallel Programming
====

These are some notes for the 3rd course of Scala Specialization on Cousera.

- [Parallel Programming](#parallel-programming)
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
    - [1.2.6. Example: Monte Carlo Method to Estimate $\pi$](#126-example-monte-carlo-method-to-estimate-pi)
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
  - [Parallel Sorting](#parallel-sorting)
    - [Merge Sort](#merge-sort)
    - [Copying the Array](#copying-the-array)
  - [Parallelism and collections](#parallelism-and-collections)
    - [Functional programming and collections](#functional-programming-and-collections)
    - [Choice of data structures](#choice-of-data-structures)
    - [Map: meaning and properties](#map-meaning-and-properties)
    - [Map as function on lists](#map-as-function-on-lists)
    - [Sequential map of an array producing an array](#sequential-map-of-an-array-producing-an-array)
    - [Parallel map of an array producing an array](#parallel-map-of-an-array-producing-an-array)
    - [Parallel map on immutable trees](#parallel-map-on-immutable-trees)
    - [Comparison of arrays and immutable trees](#comparison-of-arrays-and-immutable-trees)
- [3. Week 3 Data-ParaLLelism](#3-week-3-data-parallelism)
- [4. Week 4 Data Structures](#4-week-4-data-structures)

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

## Parallelism and collections

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

# 3. Week 3 Data-ParaLLelism
# 4. Week 4 Data Structures


[WEPSKAM]: https://www.akkadia.org/drepper/cpumemory.pdf
[SRJPE]: https://dri.es/files/oopsla07-georges.pdf