## Collections

### Vocabulary

Scala has a rich collections library. You've already seen the most commonly used collection types in previous chapters—arrays, lists, sets, and maps—but there is more to the story. In this chapter, we'll start by giving an overview of how these types relate to each other in the collections inheritance hierarchy. We'll also briefly describe these and various other collection types that you may occasionally want to use, including discussing their tradeoffs of speed, space, and requirements on input data.

### 17.1 Overview of the library

The Scala collections library involves many classes and traits. As a result, it can be challenging to get a big picture of the library by browsing the Scaladoc documentation. In Figure 17.1, we show just the traits you need to know about to understand the big picture.

The main trait is Iterable, which is the supertrait of both mutable and immutable variations of sequences (Seqs), sets, and maps. Sequences are ordered collections, such as arrays and lists. Sets contain at most one of each object, as determined by the == method. Maps contain a collection of keys mapped to values.

Iterable is so named because it represents collection objects that can produce an Iterator via a method named elements:

```
  def elements: Iterator[A]
```

The A in this example is the type parameter to Iterator, which indicates the type of element objects contained in the collection. The Iterator returned by elements is parameterized by the same type. An Iterable[Int]'s elements method, for example, will produce an Iterator[Int].

Iterable provides dozens of useful concrete methods, all implemented in terms of the Iterator returned by elements, which is the sole abstract method in trait Iterable. Among the methods defined in Iterable are many higher-order methods, most of which you've already seen in previous chapters. Some examples are map, flatMap, filter, exists, and find. These higher-order methods provide concise ways to iterate through collections for specific purposes, such as to transform each element and produce a new collection (the map method) or find the first occurrence of an element given a predicate (the find method).

![Figure17.1](https://github.com/kunSong/Note/blob/master/ProgrammingInScala/res/drawable/Figure17.1.jpg)

Figure 17.1 - Class hierarchy for Scala collections.

An Iterator has many of the same methods as Iterable, including the higher-order ones, but does not belong to the same hierarchy. As shown in Figure 17.2, trait Iterator extends AnyRef. The difference between Iterable and Iterator is that trait Iterable represents types that can be iterated over (i.e., collection types), whereas trait Iterator is the mechanism used to perform an iteration. Although an Iterable can be iterated over multiple times, an Iterator can be used just once. Once you've iterated through a collection with an Iterator, you can't reuse it. If you need to iterate through the same collection again, you'll need to call elements on that collection to obtain a new Iterator.

The many concrete methods provided by Iterator are implemented in terms of two abstract methods, next and hasNext:

```
  def hasNext: Boolean
  def next: A
```

The hasNext method indicates whether any elements remain in the iteration. The next method returns the next element.

Although most of the Iterable implementations you are likely to encounter will represent collections of a finite size, Iterable can also be used to represent infinite collections. The Iterator returned from an infinite collection could, for example, calculate and return the next digit of π each time its next method was invoked.

![Figure17.2](https://github.com/kunSong/Note/blob/master/ProgrammingInScala/res/drawable/Figure17.2.jpg)

Figure 17.2 - Class hierarchy for Iterator.

### 17.2 Sequences


Sequences, classes that inherit from trait Seq, let you work with groups of data lined up in order. Because the elements are ordered, you can ask for the first element, second element, 103rd element, and so on. In this section, we'll give you a quick tour of the most important sequences.

**Lists**

Perhaps the most important sequence type to know about is class List, the immutable linked-list described in detail in the previous chapter. Lists support fast addition and removal of items to the beginning of the list, but they do not provide fast access to arbitrary indexes because the implementation must iterate through the list linearly.

This combination of features might sound odd, but they hit a sweet spot that works well for many algorithms. The fast addition and removal of initial elements means that pattern matching works well, as described in Chapter 15. The immutability of lists helps you develop correct, efficient algorithms because you never need to make copies of a list. Here's a short example showing how to initialize a list and access its head and tail:

```
  scala> val colors = List("red", "blue", "green")
  colors: List[java.lang.String] = List(red, blue, green)
  
  scala> colors.head
  res0: java.lang.String = red
  
  scala> colors.tail
  res1: List[java.lang.String] = List(blue, green)
```

For an introduction to lists see Step 8 in Chapter 3, and for the details on using lists, see Chapter 16. Lists will also be discussed in Chapter 22, which provides insight into how lists are implemented in Scala.

**Arrays**

Arrays allow you to hold a sequence of elements and efficiently access an element at an arbitrary position, both to get or update the element, with a zero-based index. Here's how you create an array whose size you know, but for which you don't yet know the element values:

```
  scala> val fiveInts = new Array[Int](5)
  fiveInts: Array[Int] = Array(0, 0, 0, 0, 0)
```

Here's how you initialize an array when you do know the element values:

```
  scala> val fiveToOne = Array(5, 4, 3, 2, 1)
  fiveToOne: Array[Int] = Array(5, 4, 3, 2, 1)
```

As mentioned previously, arrays are accessed in Scala by placing an index in parentheses, not square brackets as in Java. Here's an example of both accessing and updating an array element:

```
  scala> fiveInts(0) = fiveToOne(4)
  
  scala> fiveInts
  res1: Array[Int] = Array(1, 0, 0, 0, 0)
```

Scala arrays are represented in the same way as Java arrays. So, you can seamlessly use existing Java methods that return arrays.[1]

You have seen arrays in action many times in previous chapters. The basics are in Step 7 in Chapter 3. Several examples of iterating through the elements of an array with a for expression are shown in Section 7.3. Arrays also figure prominently in the two-dimensional layout library of Chapter 10.

**List buffers**

Class List provides fast access to the head of the list, but not the end. Thus, when you need to build a list by appending to the end, you should consider building the list backwards by prepending elements to the front, then when you're done, calling reverse to get the elements in the order you need.

Another alternative, which avoids the reverse operation, is to use a ListBuffer. A ListBuffer is a mutable object (contained in package scala.collection.mutable), which can help you build lists more efficiently when you need to append. ListBuffer provides constant time append and prepend operations. You append elements with the += operator, and prepend them with the +: operator. When you're done building, you can obtain a List by invoking toList on the ListBuffer. Here's an example:

```
  scala> import scala.collection.mutable.ListBuffer
  import scala.collection.mutable.ListBuffer
  
  scala> val buf = new ListBuffer[Int]             
  buf: scala.collection.mutable.ListBuffer[Int] = ListBuffer()
  
  scala> buf += 1                                  
  
  scala> buf += 2                                  
  
  scala> buf     
  res11: scala.collection.mutable.ListBuffer[Int]
    = ListBuffer(1, 2)
  
  scala> 3 +: buf                                  
  res12: scala.collection.mutable.Buffer[Int]
    = ListBuffer(3, 1, 2)
  
  scala> buf.toList
  res13: List[Int] = List(3, 1, 2)
```

Another reason to use ListBuffer instead of List is to prevent the potential for stack overflow. If you can build a list in the desired order by prepending, but the recursive algorithm that would be required is not tail recursive, you can use a for expression or while loop and a ListBuffer instead. You'll see ListBuffer being used in this way in Section 22.2.

**Array buffers**

An ArrayBuffer is like an array, except that you can additionally add and remove elements from the beginning and end of the sequence. All Array operations are available, though they are a little slower due to a layer of wrapping in the implementation. The new addition and removal operations are constant time on average, but occasionally require linear time due to the implementation needing to allocate a new array to hold the buffer's contents.

To use an ArrayBuffer, you must first import it from the mutable collections package:

```
  scala> import scala.collection.mutable.ArrayBuffer
  import scala.collection.mutable.ArrayBuffer
```

When you create an ArrayBuffer, you must specify a type parameter, but need not specify a length. The ArrayBuffer will adjust the allocated space automatically as needed:

```
  scala> val buf = new ArrayBuffer[Int]()
  buf: scala.collection.mutable.ArrayBuffer[Int] = 
    ArrayBuffer()
```

You can append to an ArrayBuffer using the += method:

```
  scala> buf += 12
  
  scala> buf += 15
  
  scala> buf
  res16: scala.collection.mutable.ArrayBuffer[Int] = 
    ArrayBuffer(12, 15)
```

All the normal array methods are available. For example, you can ask an ArrayBuffer its length, or you can retrieve an element by its index:

```
  scala> buf.length
  res17: Int = 2
  
  scala> buf(0)
  res18: Int = 12
```

**Queues**

If you need a first-in-first-out sequence, you can use a Queue. Scala's collection library provides both mutable and immutable variants of Queue. Here's how you can create an empty immutable queue:

```
  scala> import scala.collection.immutable.Queue
  import scala.collection.immutable.Queue
  
  scala> val empty = new Queue[Int]             
  empty: scala.collection.immutable.Queue[Int] = Queue()
```

You can append an element to an immutable queue with enqueue:

```
  scala> val has1 = empty.enqueue(1)
  has1: scala.collection.immutable.Queue[Int] = Queue(1)
```

To append multiple elements to a queue, call enqueue with a collection as its argument:

```
  scala> val has123 = has1.enqueue(List(2, 3))
  has123: scala.collection.immutable.Queue[Int] = Queue(1,2,3)
```

To remove an element from the head of the queue, you use dequeue:

```
  scala> val (element, has23) = has123.dequeue
  element: Int = 1
  has23: scala.collection.immutable.Queue[Int] = Queue(2,3)
```

On immutable queues, the dequeue method returns a pair (a Tuple2) consisting of the element at the head of the queue, and the rest of the queue with the head element removed.
You use a mutable queue similarly to how you use an immutable one, but instead of enqueue, you use the += and ++= operators to append. Also, on a mutable queue, the dequeue method will just remove the head element from the queue and return it. Here's an example:

```
  scala> import scala.collection.mutable.Queue                
  import scala.collection.mutable.Queue
  
  scala> val queue = new Queue[String]
  queue: scala.collection.mutable.Queue[String] = Queue()
  
  scala> queue += "a"
  
  scala> queue ++= List("b", "c")
  
  scala> queue
  res21: scala.collection.mutable.Queue[String] = Queue(a, b, c)
  
  scala> queue.dequeue
  res22: String = a
  
  scala> queue
  res23: scala.collection.mutable.Queue[String] = Queue(b, c)
```

**Stacks**

If you need a last-in-first-out sequence, you can use a Stack, which also comes in both mutable and immutable versions in the Scala collections library. You push an element onto a stack with push, pop an element with pop, and peek at the top of the stack without removing it with top. Here's an example of a mutable stack:

```
  scala> import scala.collection.mutable.Stack
  import scala.collection.mutable.Stack
  
  scala> val stack = new Stack[Int]           
  stack: scala.collection.mutable.Stack[Int] = Stack()
  
  scala> stack.push(1)
  
  scala> stack
  res1: scala.collection.mutable.Stack[Int] = Stack(1)
  
  scala> stack.push(2)
  
  scala> stack
  res3: scala.collection.mutable.Stack[Int] = Stack(1, 2)
  
  scala> stack.top
  res8: Int = 2
  
  scala> stack
  res9: scala.collection.mutable.Stack[Int] = Stack(1, 2)
  
  scala> stack.pop    
  res10: Int = 2
  
  scala> stack    
  res11: scala.collection.mutable.Stack[Int] = Stack(1)
```

**Strings (via RichString)**

One other sequence to be aware of is RichString, which is a Seq[Char]. Because Predef has an implicit conversion from String to RichString, you can treat any string as a Seq[Char]. Here's an example:

```
  scala> def hasUpperCase(s: String) = s.exists(_.isUpperCase)
  hasUpperCase: (String)Boolean
  
  scala> hasUpperCase("Robert Frost")
  res14: Boolean = true
  
  scala> hasUpperCase("e e cummings")
  res15: Boolean = false
```

In this example, the exists method is invoked on the string named s in the hasUpperCase method body. Because no method named "exists" is declared in class String itself, the Scala compiler will implicitly convert s to RichString, which has the method. The exists method treats the string as a Seq[Char], and will return true if any of the characters are upper case.[2]

### 17.3 Sets and maps

You have already seen the basics of sets and maps in previous chapters, starting with Step 10 in Chapter 3. In this section, we'll give more insight into their use and show you a few more examples.

As mentioned previously, the Scala collections library offers both mutable and immutable versions of sets and maps. The hierarchy for sets is shown in Figure 3.2 here, and the hierarchy for maps is shown in Figure 3.3 here. As these diagrams show, the simple names Set and Map are used by three traits each, residing in different packages.

By default when you write "Set" or "Map" you get an immutable object. If you want the mutable variant, you need to do an explicit import. Scala gives you easier access to the immutable variants, as a gentle encouragement to prefer them over their mutable counterparts. The easy access is provided via the Predef object, which is implicitly imported into every Scala source file. Listing 17.1 shows the relevant definitions.

```
    object Predef {
      type Set[T] = scala.collection.immutable.Set[T]
      type Map[K, V] = scala.collection.immutable.Map[K, V]
      val Set = scala.collection.immutable.Set
      val Map = scala.collection.immutable.Map
      // ...
    }
```

Listing 17.1 - Default map and set definitions in Predef.

The "type" keyword is used in Predef to define Set and Map as aliases for the longer fully qualified names of the immutable set and map traits.[3] The vals named Set and Map are initialized to refer to the singleton objects for the immutable Set and Map. So Map is the same as Predef.Map, which is defined to be the same as scala.collection.immutable.Map. This holds both for the Map type and Map object.

If you want to use both mutable and immutable sets or maps in the same source file, one approach is to import the name of the package that contains the mutable variants:

```
  scala> import scala.collection.mutable
  import scala.collection.mutable
```

You can continue to refer to the immutable set as Set, as before, but can now refer to the mutable set as mutable.Set. Here's an example:

```
  scala> val mutaSet = mutable.Set(1, 2, 3)          
  mutaSet: scala.collection.mutable.Set[Int] = Set(3, 1, 2)
```

**Using sets**

The key characteristic of sets is that they will ensure that at most one of each object, as determined by ==, will be contained in the set at any one time. As an example, we'll use a set to count the number of different words in a string.

The split method on String can separate a string into words, if you specify spaces and punctuation as word separators. The regular expression "[ !,.]+" will suffice: it indicates the string should be split at each place that one or more space and/or punctuation characters exist:

```
  scala> val text = "See Spot run. Run, Spot. Run!"
  text: java.lang.String = See Spot run. Run, Spot. Run!
  
  scala> val wordsArray = text.split("[ !,.]+")    
  wordsArray: Array[java.lang.String] =
     Array(See, Spot, run, Run, Spot, Run)
```

To count the distinct words, you can convert them to the same case and then add them to a set. Because sets exclude duplicates, each distinct word will appear exactly one time in the set. First, you can create an empty set using the empty method provided on the Set companion objects:

```
  scala>  val words = mutable.Set.empty[String]
  words: scala.collection.mutable.Set[String] = Set()
```

Then, just iterate through the words with a for expression, convert each word to lower case, and add it to the mutable set with the += operator:

```
  scala> for (word <- wordsArray)
           words += word.toLowerCase
  
  scala> words
  res25: scala.collection.mutable.Set[String] =
    Set(spot, run, see)
```

Thus, the text contained exactly three distinct words: spot, run, and see. The most commonly used methods on both mutable and immutable sets are shown in Table 17.1.

**Using maps**

Maps let you associate a value with each element of the collection. Using a map looks similar to using an array, except that instead of indexing with integers counting from 0, you can use any kind of key. If you import the scala.collection.mutable package, you can create an empty mutable map like this:

```
  scala> val map = mutable.Map.empty[String, Int]
  map: scala.collection.mutable.Map[String,Int] = Map()
```

**Common operations for sets**

What it is                       |  What it does
---------------------------------|-----------------------------------------------------------------
val nums = Set(1, 2, 3)          |  Creates an immutable set (nums.toString returns Set(1, 2, 3))
nums + 5                         |  Adds an element (returns Set(1, 2, 3, 5))
nums - 3                         |  Removes an element (returns Set(1, 2))
nums ++ List(5, 6)               |  Adds multiple elements (returns Set(1, 2, 3, 5, 6))
nums -- List(1, 2)               |  Removes multiple elements (returns Set(3))
nums ** Set(1, 3, 5, 7)          |  Takes the intersection of two sets (returns Set(1, 3))
nums.size                        |  Returns the size of the set (returns 3)
nums.contains(3)                 |  Checks for inclusion (returns true)
import scala.collection.mutable  |  Makes the mutable collections easy to access
val words =                      | 
mutable.Set.empty[String]        |  Creates an empty, mutable set (words.toString returns Set())
words += "the"                   |  Adds an element (words.toString returns Set(the))
words -= "the"                   |  Removes an element, if it exists (words.toString returns Set())
words ++= List("do", "re", "mi") |  Adds multiple elements (words.toString returns Set(do, re, mi))
words --= List("do", "re")       |  Removes multiple elements (words.toString returns Set(mi))
words.clear                      |  Removes all elements (words.toString returns Set())

Note that when you create a map, you must specify two types. The first type is for the keys of the map, the second for the values. In this case, the keys are strings and the values are integers.

Setting entries in a map looks similar to setting entries in an array:

```
  scala> map("hello") = 1
  
  scala> map("there") = 2
  
  scala> map
  res28: scala.collection.mutable.Map[String,Int] =
    Map(hello -> 1, there -> 2)
```

Likewise, reading a map is similar to reading an array:

```
  scala> map("hello")
  res29: Int = 1
```

Putting it all together, here is a method that counts the number of times each word occurs in a string:

```
  scala> def countWords(text: String) = {
           val counts = mutable.Map.empty[String, Int]
           for (rawWord <- text.split("[ ,!.]+")) {
             val word = rawWord.toLowerCase
             val oldCount = 
               if (counts.contains(word)) counts(word)
               else 0
             counts += (word -> (oldCount + 1))
           }
           counts
         }
  countWords: (String)scala.collection.mutable.Map[String,Int]
  
  
  scala> countWords("See Spot run! Run, Spot. Run!")
  res30: scala.collection.mutable.Map[String,Int] =
    Map(see -> 1, run -> 3, spot -> 2)
```

Given these counts, you can see that this text talks a lot about running, but not so much about seeing.

The way this code works is that a mutable map, named counts, maps each word to the number of times it occurs in the text. For each word in the text, the word's old count is looked up, that count is incremented by one, and the new count is saved back into counts. Note the use of contains to check whether a word has been seen yet or not. If counts.contains(word) is not true, then the word has not yet been seen and zero is used for the count.

Many of the most commonly used methods on both mutable and immutable maps are shown in Table 17.2.

**Common operations for maps**

What it is                          |What it does
------------------------------------|-------------------------------------------------------------------------------------
val nums = Map("i" -> 1, "ii" -> 2) |Creates an immutable map (nums.toString returns Map(i -> 1, ii -> 2))
nums + ("vi" -> 6)                  |Adds an entry (returns Map(i -> 1, ii -> 2, vi -> 6))
nums - "ii"                         |Removes an entry (returns Map(i -> 1))
nums ++ List("iii" -> 3, "v" -> 5)  |Adds multiple entries (returns Map(i -> 1, ii -> 2, iii -> 3, v -> 5))
nums -- List("i", "ii")             |Removes multiple entries (returns Map())
nums.size                           |Returns the size of the map (returns 2)
nums.contains("ii")                 |Checks for inclusion (returns true)
nums("ii")                          |Retrieves the value at a specified key (returns 2)
nums.keys                           |Returns the keys (returns an Iterator over the strings "i" and "ii")
nums.keySet                         |Returns the keys as a set (returns Set(i, ii))
nums.values                         |Returns the values (returns an Iterator over the integers 1 and 2)
nums.isEmpty                        |Indicates whether the map is empty (returns false)
import scala.collection.mutable     |Makes the mutable collections easy to access
val words =                         |
mutable.Map.empty[Stri|ng, Int]     |Creates an empty, mutable map
words += ("one" -> 1)               |Adds a map entry from "one" to 1 (words.toString returns Map(one -> 1))
words -= "one"                      |Removes a map entry, if it exists (words.toString returns Map())
words ++= List("one" -> 1,          |
"two" ->| 2, "three" -> 3)          |Adds multiple map entries (words.toString returns Map(one -> 1, two -> 2, three -> 3))
words --= List("one", "two")        |Removes multiple objects (words.toString returns Map(three -> 3))

**Default sets and maps**

For most uses, the implementations of mutable and immutable sets and maps provided by the Set(), scala.collection.mutable.Map(), etc., factories will likely be sufficient. The implementations provided by these factories use a fast lookup algorithm, usually involving a hashtable, so they can quickly decide whether or not an object is in the collection.

The scala.collection.mutable.Set() factory method, for example, returns a scala.collection.mutable.HashSet, which uses a hashtable internally. Similarly, the scala.collection.mutable.Map() factory returns a scala.collection.mutable.HashMap.

The story for immutable sets and maps is a bit more involved. The class returned by the scala.collection.immutable.Set() factory method, for example, depends on how many elements you pass to it, as shown in Table 17.3. For sets with fewer than five elements, a special class devoted exclusively to sets of each particular size is used, to maximize performance. Once you request a set that has five or more elements in it, however, the factory method will return immutable HashSet.

Table 17.3 - Default immutable set implementations

Number of elements  | Implementation
--------------------|-------------------------------------
0                   | scala.collection.immutable.EmptySet
1                   | scala.collection.immutable.Set1
2                   | scala.collection.immutable.Set2
3                   | scala.collection.immutable.Set3
4                   | scala.collection.immutable.Set4
5 or more           | scala.collection.immutable.HashSet

Similarly, the scala.collection.immutable.Map() factory method will return a different class depending on how many key-value pairs you pass to it, as shown in Table 17.4. As with sets, for immutable maps with fewer than five elements, a special class devoted exclusively to maps of each particular size is used, to maximize performance. Once a map has five or more key-value pairs in it, however, an immutable HashMap is used.

Table 17.4 - Default immutable map implementations

Number of elements  | Implementation
--------------------|-------------------------------------
0                   | scala.collection.immutable.EmptyMap
1                   | scala.collection.immutable.Map1
2                   | scala.collection.immutable.Map2
3                   | scala.collection.immutable.Map3
4                   | scala.collection.immutable.Map4
5 or more           | scala.collection.immutable.HashMap

The default immutable implementation classes shown in Tables 17.3 and 17.4 work together to give you maximum performance. For example, if you add an element to an EmptySet, it will return a Set1. If you add an element to that Set1, it will return a Set2. If you then remove an element from the Set2, you'll get another Set1.

**Sorted sets and maps**

On occasion you may need a set or map whose iterator returns elements in a particular order. For this purpose, the Scala collections library provides traits SortedSet and SortedMap. These traits are implemented by classes TreeSet and TreeMap, which use a red-black tree to keep elements (in the case of TreeSet) or keys (in the case of TreeMap) in order. The order is determined by the Ordered trait, which the element type of the set, or key type of the map, must either mix in or be implicitly convertable to. These classes only come in immutable variants. Here are some TreeSet examples:

```
  scala> import scala.collection.immutable.TreeSet
  import scala.collection.immutable.TreeSet
  
  scala> val ts = TreeSet(9, 3, 1, 8, 0, 2, 7, 4, 6, 5)
  ts: scala.collection.immutable.SortedSet[Int] =
    Set(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
  
  scala> val cs = TreeSet('f', 'u', 'n')
  cs: scala.collection.immutable.SortedSet[Char] = Set(f, n, u)
```

And here are a few TreeMap examples:

```
  scala> import scala.collection.immutable.TreeMap
  import scala.collection.immutable.TreeMap
  
  scala> var tm = TreeMap(3 -> 'x', 1 -> 'x', 4 -> 'x')
  tm: scala.collection.immutable.SortedMap[Int,Char] =
    Map(1 -> x, 3 -> x, 4 -> x)
  
  scala> tm += (2 -> 'x')
  
  scala> tm
  res38: scala.collection.immutable.SortedMap[Int,Char] =
    Map(1 -> x, 2 -> x, 3 -> x, 4 -> x)
```

**Synchronized sets and maps**

In Section 1.1, we mentioned that if you needed a thread-safe map, you could mix the SynchronizedMap trait into whatever particular map implementation you desired. For example, you could mix SynchronizedMap into HashMap, as shown in Listing 17.2. This example begins with an import of two traits, Map and SynchronizedMap, and one class, HashMap, from package scala.collection.mutable. The rest of the example is the definition of singleton object MapMaker, which declares one method, makeMap. The makeMap method declares its result type to be a mutable map of string keys to string values.

```
    import scala.collection.mutable.{Map,
        SynchronizedMap, HashMap}
  
    object MapMaker {
  
      def makeMap: Map[String, String] = {
  
          new HashMap[String, String] with
              SynchronizedMap[String, String] {
  
            override def default(key: String) =
              "Why do you want to know?"
          }
      }
    }
```

Listing 17.2 - Mixing in the SynchronizedMap trait.

The first statement inside the body of makeMap constructs a new mutable HashMap that mixes in the SynchronizedMap trait:

```
  new HashMap[String, String] with
    SynchronizedMap[String, String]
```

Given this code, the Scala compiler will generate a synthetic subclass of HashMap that mixes in SynchronizedMap, and create (and return) an instance of it. This synthetic class will also override a method named default, because of this code:

```
  override def default(key: String) =
    "Why do you want to know?"
```

If you ask a map to give you the value for a particular key, but it doesn't have a mapping for that key, you'll by default get a NoSuchElementException. If you define a new map class and override the default method, however, your new map will return the value returned by default when queried with a non-existent key. Thus, the synthetic HashMap subclass generated by the compiler from the code in Listing 17.2 will return the somewhat curt response string, "Why do you want to know?", when queried with a non-existent key.
Because the mutable map returned by the makeMap method mixes in the SynchronizedMap trait, it can be used by multiple threads at once. Each access to the map will be synchronized. Here's an example of the map being used, by one thread, in the interpreter:

```
  scala> val capital = MapMaker.makeMap  
  capital: scala.collection.mutable.Map[String,String] = Map()
  
  scala> capital ++ List("US" -> "Washington",
              "Paris" -> "France", "Japan" -> "Tokyo")
  res0: scala.collection.mutable.Map[String,String] =
    Map(Paris -> France, US -> Washington, Japan -> Tokyo)
  
  scala> capital("Japan")
  res1: String = Tokyo
  
  scala> capital("New Zealand")
  res2: String = Why do you want to know?
  
  scala> capital += ("New Zealand" -> "Wellington")
  
  scala> capital("New Zealand")                    
  res3: String = Wellington
```

You can create synchronized sets similarly to the way you create synchronized maps. For example, you could create a synchronized HashSet by mixing in the SynchronizedSet trait, like this:

```
  import scala.collection.mutable
  
  val synchroSet =
    new mutable.HashSet[Int] with
        mutable.SynchronizedSet[Int]
```

Finally, if you are thinking of using synchronized collections, you may also wish to consider the concurrent collections of java.util.concurrent instead. Alternatively, you may prefer to use unsynchronized collections with Scala actors. Actors will be covered in detail in Chapter 30.

### 17.4 Selecting mutable versus immutable collections

For some problems, mutable collections work better, and for others, immutable collections work better. When in doubt, it is better to start with an immutable collection and change it later if you need to, because immutable collections can be easier to reason about than mutable ones.
It can also sometimes be worthwhile to go the opposite way. If you find some code that uses mutable collections becoming complicated and hard to reason about, consider whether it would help to change some of the collections to immutable alternatives. In particular, if you find yourself worrying about making copies of mutable collections in just the right places, or thinking a lot about who "owns" or "contains" a mutable collection, consider switching some of the collections to their immutable counterparts.

Besides being potentially easier to reason about, immutable collections can usually be stored more compactly than mutable ones if the number of elements stored in the collection is small. For instance an empty mutable map in its default representation of HashMap takes up about 80 bytes and about 16 more are added for each entry that's added to it. An empty immutable Map is a single object that's shared between all references, so referring to it essentially costs just a single pointer field. What's more, the Scala collections library currently stores immutable maps and sets with up to four entries in a single object, which typically takes up between 16 and 40 bytes, depending on the number of entries stored in the collection.[4] So for small maps and sets, the immutable versions are much more compact than the mutable ones. Given that many collections are small, switching them to be immutable can give important space savings and performance advantages.

To make it easier to switch from immutable to mutable collections, and vice versa, Scala provides some syntactic sugar. Even though immutable sets and maps do not support a true += method, Scala gives a useful alternate interpretation to +=. Whenever you write a += b, and a does not support a method named +=, Scala will try interpreting it as a = a + b. For example, immutable sets do not support a += operator:

```
  scala> val people = Set("Nancy", "Jane")
  people: scala.collection.immutable.Set[java.lang.String] =
    Set(Nancy, Jane)
  
  scala> people += "Bob"
  <console>:6: error: reassignment to val
         people += "Bob"
                ^
```

If you declare people as a var, instead of a val, however, then the collection can be "updated" with a += operation, even though it is immutable. First, a new collection will be created, and then people will be reassigned to refer to the new collection:

```
  scala> var people = Set("Nancy", "Jane")
  people: scala.collection.immutable.Set[java.lang.String] = 
    Set(Nancy, Jane)
  
  scala> people += "Bob"
  
  scala> people
  res42: scala.collection.immutable.Set[java.lang.String] = 
    Set(Nancy, Jane, Bob)
```

After this series of statements, the people variable refers to a new immutable set, which contains the added string, "Bob". The same idea applies to any method ending in =, not just the += method. Here's the same syntax used with the -= operator, which removes an element from a set, and the ++= operator, which adds a collection of elements to a set:

```
  scala> people -= "Jane"
  
  scala> people ++= List("Tom", "Harry")
  
  scala> people
  res45: scala.collection.immutable.Set[java.lang.String] = 
    Set(Nancy, Bob, Tom, Harry)
```

To see how this is useful, consider again the following Map example from Section 1.1:

```
  var capital = Map("US" -> "Washington", "France" -> "Paris")
  capital += ("Japan" -> "Tokyo")
  println(capital("France")) 
```

This code uses immutable collections. If you want to try using mutable collections instead, all that is necessary is to import the mutable version of Map, thus overriding the default import of the immutable Map:

```
  import scala.collection.mutable.Map  // only change needed!
  var capital = Map("US" -> "Washington", "France" -> "Paris")
  capital += ("Japan" -> "Tokyo")
  println(capital("France")) 
```

Not all examples are quite that easy to convert, but the special treatment of methods ending in an equals sign will often reduce the amount of code that needs changing.
By the way, this syntactic treatment works on any kind of value, not just collections. For example, here it is being used on floating-point numbers:

```
  scala> var roughlyPi = 3.0
  roughlyPi: Double = 3.0
  
  scala> roughlyPi += 0.1
  
  scala> roughlyPi += 0.04
  
  scala> roughlyPi
  res48: Double = 3.14
```

The effect of this expansion is similar to Java's assignment operators +=, -=, *=, etc., but it is more general because every operator ending in = can be converted.

### 17.5 Initializing collections

As you've seen previously, the most common way to create and initialize a collection is to pass the initial elements to a factory method on the companion object of your chosen collection. You just place the elements in parentheses after the companion object name, and the Scala compiler will transform that to an invocation of an apply method on that companion object:

```
  scala> List(1, 2, 3)
  res0: List[Int] = List(1, 2, 3)
  
  scala> Set('a', 'b', 'c')
  res1: scala.collection.immutable.Set[Char] = Set(a, b, c)
  
  scala> import scala.collection.mutable
  import scala.collection.mutable
  
  scala> mutable.Map("hi" -> 2, "there" -> 5)
  res2: scala.collection.mutable.Map[java.lang.String,Int] =
    Map(hi -> 2, there -> 5)
  
  scala> Array(1.0, 2.0, 3.0)
  res3: Array[Double] = Array(1.0, 2.0, 3.0)
```

Although most often you can let the Scala compiler infer the element type of a collection from the elements passed to its factory method, sometimes you may want to create a collection but specify a different type from the one the compiler would choose. This is especially an issue with mutable collections. Here's an example:

```
  scala> import scala.collection.mutable
  import scala.collection.mutable
  
  scala> val stuff = mutable.Set(42)
  stuff: scala.collection.mutable.Set[Int] = Set(42)
  
  scala> stuff += "abracadabra"
  <console>:7: error: type mismatch;
   found   : java.lang.String("abracadabra")
   required: Int
         stuff += "abracadabra"
                  ^
```

The problem here is that stuff was given an element type of Int. If you want it to have an element type of Any, you need to say so explicitly by putting the element type in square brackets, like this:

```
  scala> val stuff = mutable.Set[Any](42)
  stuff: scala.collection.mutable.Set[Any] = Set(42)
```

Another special situation is if you want to initialize a collection with another collection. For example, imagine you have a list, but you want a TreeSet containing the elements in the list. Here's the list:

```
  scala> val colors = List("blue", "yellow", "red", "green")
  colors: List[java.lang.String] =
    List(blue, yellow, red, green)
```

You cannot pass the colors list to the factory method for TreeSet:

```
  scala> import scala.collection.immutable.TreeSet
  import scala.collection.immutable.TreeSet
  
  scala> val treeSet = TreeSet(colors)                 
  <console>:6: error: no implicit argument matching
    parameter type (List[java.lang.String]) =>
      Ordered[List[java.lang.String]] was found.
         val treeSet = TreeSet(colors)
                       ^
```

Instead, you'll need to create an empty TreeSet[String] and add to it the elements of the list with the TreeSet's ++ operator:

```
  scala> val treeSet = TreeSet[String]() ++ colors
  treeSet: scala.collection.immutable.SortedSet[String] =
     Set(blue, green, red, yellow)
```

**Converting to array or list**

If you need to initialize a list or array with another collection, on the other hand, it is quite straightforward. As you've seen previously, to initialize a new list with another collection, simply invoke toList on that collection:

```
  scala> treeSet.toList
  res54: List[String] = List(blue, green, red, yellow)
```

Or, if you need an array, invoke toArray:

```
  scala> treeSet.toArray
  res55: Array[String] = Array(blue, green, red, yellow)
```

Note that although the original colors list was not sorted, the elements in the list produced by invoking toList on the TreeSet are in alphabetical order. When you invoke toList or toArray on a collection, the order of the elements in the resulting list or array will be the same as the order of elements produced by an iterator obtained by invoking elements on that collection. Because a TreeSet[String]'s iterator will produce strings in alphabetical order, those strings will appear in alphabetical order in the list resulting from invoking toList on that TreeSet.

Keep in mind, however, that conversion to lists or arrays usually requires copying all of the elements of the collection, and thus may be slow for large collections. Sometimes you need to do it, though, due to an existing API. Further, many collections only have a few elements anyway, in which case there is only a small speed penalty.

**Converting between mutable and immutable sets and maps**

Another situation that may arise occasionally is the need to convert a mutable set or map to an immutable one, or vice versa. To accomplish this, you can use the technique shown previously to initialize a TreeSet with the elements of a list. If you have a mutable collection, and want to convert it to a immutable one, for example, create an empty immutable collection and add the elements of the mutable one via the ++ operator. Here's how you'd convert the immutable TreeSet from the previous example to a mutable set, and back again to an immutable one:

```
  scala> import scala.collection.mutable
  import scala.collection.mutable
  
  scala> treeSet
  res5: scala.collection.immutable.SortedSet[String] =
    Set(blue, green, red, yellow)
  
  scala> val mutaSet = mutable.Set.empty ++ treeSet
  mutaSet: scala.collection.mutable.Set[String] =
    Set(yellow, blue, red, green)
  
  scala> val immutaSet = Set.empty ++ mutaSet
  immutaSet: scala.collection.immutable.Set[String] =
    Set(yellow, blue, red, green)
```

You can use the same technique to convert between mutable and immutable maps:

```
  scala> val muta = mutable.Map("i" -> 1, "ii" -> 2)
  muta: scala.collection.mutable.Map[java.lang.String,Int] =
     Map(ii -> 2, i -> 1)
  
  scala> val immu = Map.empty ++ muta
  immu: scala.collection.immutable.Map[java.lang.String,Int] =
     Map(ii -> 2, i -> 1)
```

### 17.6 Tuples

As described in Step 9 in Chapter 3, a tuple combines a fixed number of items together so that they can be passed around as a whole. Unlike an array or list, a tuple can hold objects with different types. Here is an example of a tuple holding an integer, a string, and the console:

```
  (1, "hello", Console)
```

Tuples save you the tedium of defining simplistic data-heavy classes. Even though defining a class is already easy, it does require a certain minimum effort, which sometimes serves no purpose. Tuples save you the effort of choosing a name for the class, choosing a scope to define the class in, and choosing names for the members of the class. If your class simply holds an integer and a string, there is no clarity added by defining a class named AnIntegerAndAString.

Because tuples can combine objects of different types, tuples do not inherit from Iterable. If you find yourself wanting to group exactly one integer and exactly one string, then you want a tuple, not a List or Array.

A common application of tuples is returning multiple values from a method. For example, here is a method that finds the longest word in a collection and also returns its index:

```
  def longestWord(words: Array[String]) = {
    var word = words(0)
    var idx = 0
    for (i <- 1 until words.length)
      if (words(i).length > word.length) {
        word = words(i)
        idx = i
      }
    (word, idx)
  }
```

Here is an example use of the method:

```
  scala> val longest = 
           longestWord("The quick brown fox".split(" "))
  longest: (String, Int) = (quick,1)
```

The longestWord function here computes two items: word, the longest word in the array, and idx, the index of that word. To keep things simple, the function assumes there is at least one word in the list, and it breaks ties by choosing the word that comes earlier in the list. Once the function has chosen which word and index to return, it returns both of them together using the tuple syntax (word, idx).
To access elements of a tuple, you can use method _1 to access the first element, _2 to access the second, and so on:

```
  scala> longest._1
  res56: String = quick
  
  scala> longest._2
  res57: Int = 1
```

Additionally, you can assign each element of the tuple to its own variable,[5] like this:

```
  scala> val (word, idx) = longest
  word: String = quick
  idx: Int = 1
  
  scala> word
  res58: String = quick
```

By the way, if you leave off the parentheses you get a different result:

```
  scala> val word, idx = longest
  word: (String, Int) = (quick,1)
  idx: (String, Int) = (quick,1)
```

This syntax gives multiple definitions of the same expression. Each variable is initialized with its own evaluation of the expression on the right-hand side. That the expression evaluates to a tuple in this case does not matter. Both variables are initialized to the tuple in its entirety. See Chapter 18 for some examples where multiple definitions are convenient.

As a note of warning, tuples are almost too easy to use. Tuples are great when you combine data that has no meaning beyond "an A and a B." However, whenever the combination has some meaning, or you want to add some methods to the combination, it is better to go ahead and create a class. For example, do not use a 3-tuple for the combination of a month, a day, and a year. Make a Date class. It makes your intentions explicit, which both clears up the code for human readers and gives the compiler and language opportunities to help you catch mistakes.

### 17.7 Conclusion

This chapter has given an overview of the Scala collections library and the most important classes and traits in it. With this foundation you should be able to work effectively with Scala collections, and know where to look in Scaladoc when you need more information. In the next chapter, we'll turn our attention from the Scala library back to the language, and discuss Scala's support for mutable objects.

### Footnotes for Chapter 17:

[1] The difference in variance of Scala and Java's arrays—i.e., whether Array[String] is a subtype of Array[AnyRef]—will be discussed in Section 19.3.

[2] The code given here of Chapter 1 presents a similar example.

[3] The type keyword will be explained in more detail in Section 20.6.

[4] The "single object" is an instance of Set1 through Set4, or Map1 through Map4, as shown in Tables 17.3 and 17.4.

[5] This syntax is actually a special case of pattern matching, as described in detail in Section 15.7.