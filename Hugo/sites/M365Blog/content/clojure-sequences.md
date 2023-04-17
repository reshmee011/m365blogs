---
title: 'Clojure Sequences'
date: Thu, 10 Dec 2015 09:32:23 +0000
draft: false
tags: ['Clojure', 'Clojure Sequences', 'Coljure Sequences', 'Uncategorized']
---

Sequences include data structure lists, vectors and lazy sequences. Functions that can be used with sequences can be found using the clojure cheatsheet. http://clojure.org/cheatsheet Below are some examples using vectors and lists as sequences.

 Syntax

 Definition

 Result

 (seq '(1 2 3 4))

 If collection is not empty, return seq object else nil

 (1 2 3 4)

 (first \[1 2 3 4\])

 returns the first element

 1

 (rest {:a 1 :b 2 :c 3 :d 4})

 returns a sequence of the rest of elements after first element

 (\[:b 2\]\[:c 3\]\[:d 4\])

 (cons 1 '( 2 3 4))

 returns a new sequence: first is 1, rest is coll

 (1 2 3 4)

**Lazy Sequences** Lazy Sequences are integral part of Clojure. They are not actually data structures but are objects that contain function which return next item in the sequence. Lazy sequences allow for gen­er­ated log­i­cal sequence which does­n’t have to exist in mem­o­ry. It makes it pos­si­ble to have infi­nite sequences which can be cached and used whenever needed. Examples of functions used with lazy sequences are:

Syntax

Definition

Result

 (range)

 generates lazy sequence of numbers infinitely

(0 1 2 3 4 5...n)

 (range 3)

 returns first three numbers from the sequence.

 (0 1 2)

 (range 1 7 2)

 returns numbers with increment of 2 starting from 1 and ending 7

 (1 3 5)

 (range 1 4)

 returns numbers starting at 1 and ending at 4

 (1 2 3)

(take 4  (range))

 take first 4 numbers

 (0 1 2 3)

(drop 4 (range))

similar to Take except means discarding the first 4 items

(4 5 6 7...n)

(iterate inc 5)

 iterate returns a lazy sequence starting at 5 using the inc function

(5 6 7 8 9 10 11 ....n)

(take 4 (iterate (partial \* 2) 5))

starting at 5 using the  multiply operation 2.

(5 10 20 40)

 (map #(\* % %) \[0 1 2 3\])

 map iterates through each element in the vector and calculate the square of the number

 (0 1 4 9 )

 (filter even? (range))

filter takes some sequence and predicate and return sequence of elements where predicate is true. The example returns all even numbers

 (0 2 4 6 8....n)

 (re-seq #"\[aeiou\]" "clojure")

 returns sequence of successive matches of pattern in string

 ("o" "u" "e")

An example of the application of the iterate function is to generate Fibonacci numbers. Fibonacci returns where a number is found by adding up the two numbers before it. Starting with 0 and 1, the sequence goes 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, and so forth, i.e.  Xn = Xn-1 + Xn-2. Let's write it in clojure

1.  Start with vector \[0 1\] which is the starting Fibonacci sequence
2.  Generate a sequence of elements from this vector added to each other using the iterate function. Iterate takes the starting vector \[1 0\] and applies a function `(iterate ([0 1])`
3.   Define the function with single argument to which \[0 1\] will be passed. Destructure the argument into symbols a and b. a contains 0 while b  contains 1. `(iterate (fn [[a b]] [0 1]))`
4.  Inside the function, return a vector which is the sequence of next element. first element of the vector is b which is one and the second element of vector is a +b which is 1 also. `(iterate (fn  [[a b]] [b (+ a b)]) [0 1]))` Running above code in REPL produces the following sequence \[0 1\] \[1 1\] \[1 2\]\[2 3\]\[3 5\]\[5 8\]......\[n n\] [![Iterate](https://reshmeeauckloo.files.wordpress.com/2015/12/iterate.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/12/iterate.jpg)
5.  The first element of the vector returned needs to be returned
6.  The map function can be used to iterate over each vector and using function first , the first element of the vector can be extracted `(map first (iterate (fn  [[a b]] [b (+ a b)]) [0 1])))` [![mapfirst](https://reshmeeauckloo.files.wordpress.com/2015/12/mapfirst.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/12/mapfirst.jpg)
7.  Create a function to generate the Fibonacci which can be invoked `(def fibs (map first (iterate (fn [[a b]] [b (+ a b)]) [0 1])))` `;;return first five Fibonacci numbers (take 5 fibs) ;;drop first five Fibonacci numbers and return the rest up to infinite (drop 5 fibs)` [![Fibonacci](https://reshmeeauckloo.files.wordpress.com/2015/12/fibonacci.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/12/fibonacci.jpg) `(map inc (take 5 fibs))` [![mapinc](https://reshmeeauckloo.files.wordpress.com/2015/12/mapinc.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/12/mapinc.jpg) It returns the first 5 Fibonacci numbers added to 1

**Other Functions  on Sequences**

Syntax

Definition

Result

 (reduce + (range 4))

 reduce iterately invoke the function + with every element of the sequence

 6

  (reduce + 10 (range 4))

 reduce retains the result of the previous indication and uses it as first argument in subsequent function. Sum of the sequence + 10

 16

 (into #{} "hello")

  Into functions injects element of the sequence into a concrete data structure

 #{\\e \\h \\l \\o}

 (into {} \[\[:x 1\] \[:y 2\]\])

  pouring sequence of key value pairs into a map

 {:x 1, :y 2}

( some {2 :b 3 :c} \[1 nil 2 3\])

  some takes a predicate function and returns the first element of the sequence for which the predicate function returns logical true. Takes advantage of map being a function. Each element in the vector 1, nil, 2 and 3 is passed as argument to the map. True is returned when 2 is reached. 2 is a key in the map which returns the value b.

 b

 (interleave \[:a :b :c :d :e\] \[1 2 3 4 5\])

 returns a seq of the first item in each coll, then the second etc.

  (:a 1 :b 2 :c 3 :d 4 :e 5)

 (partition 3 \[1 2 3 4 5 6 7 8 9\])

 returns sequence of lists of 3 items each.

  ((1 2 3) (4 5 6) (7 8 9))

 (map vector \[:a :b :c :d :e\] \[1 2 3 4 5\])

 returns sequence of vectors associating key from first coll with value from second coll

  (\[:a 1\] \[:b 2\] \[:c 3\] \[:d 4\] \[:e 5\]

 (apply str (interpose \\,"asdf"))

 a series of functions are applied. Interpose add the "," to each letter in the string and returns a list (\\a \\, \\s \\,\\ d \\,\\ f \\) str joins the list of strings into one string.

 "a, s, d, f