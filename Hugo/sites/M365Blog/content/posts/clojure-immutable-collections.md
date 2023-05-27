---
title: 'Clojure Immutable Collections'
date: Sun, 06 Dec 2015 16:02:01 +0000
draft: false
tags: ['Clojure', 'Data Types Immutability', 'Uncategorized']
---

Clojure data is immutable. Values of simple data types are immutable , e.g. true, 4, 4.05. With immutability values never change, instead new values are generated  when new states  are needed. It means all states of the collection can be referred throughout the life cycle of the application. Advantages of immutability can be read from blog http://www.developer.com/lang/other/article.php/3874551/Clojure-Immutability-at-the-Language-Level.htm Clojure provides several immutable collections listed below **Lists** Below are the syntax associated with lists

Syntax

Definition

Result

 ()

 empty list

 (1 2 3)

  1 is interpreted as a function and fails

 '(1 2 3) or (list 1 2 3)

The quote sign means don't evaluate the list and is a macro for denoting list.

 (1 2 3)

(conj '(1 2 3) 1)

The conj add value 1 at beginning of list

 (1 1 2 3 )

![List_Data](https://reshmeeauckloo.files.wordpress.com/2015/12/list_data1.jpg) lst returns (1 2 3) in the screenshot because  of immutability. **Vectors** Vector is similar to array object in other languages like C# and JavaScript

Syntax

Definition

Result

 \[\]

 empty vector

 \[\]

 \[1 2 3\] (vector 1 2 3) (vec '(1 2 3))

vector can be defined using the \[\] symbols vector is a function to define a vector. vec is a function to convert list into vector

 \[1 2 3\]

(nth \[1 2 3\] 0\]

 returns element at position 0

 1

(conj \[1 2 3\]  1)

add 3 to vector

 \[1 2 3 1\]

![Vector_data](https://reshmeeauckloo.files.wordpress.com/2015/12/vector_data.jpg) **Map** Maps is similar to dictionary object in C#. The values stored are unordered.

Syntax

Definition

Result

 {}

 empty map

  {:a 1 :b 2}

Map collection containing a is a pointer to 1 b is a pointer to 2

   {:a 1 , :b 2}

(assoc  {:a 1 :b 2} :c 3)

 associate key c with the map

  {:c 3 , :a 1, :b 2 }

(dissoc {:a 1 :b 2} :a)

dissociates key a from map

 {:b 2}

(conj {:a 1 :b 2} \[:a 1\])

add vector key a to map

   {:c 1 , :a 1, :b 2 }

![Map_Data](https://reshmeeauckloo.files.wordpress.com/2015/12/map_data.jpg) Nested access to data in maps is possible using helper functions  via path specified by keys

Syntax

Definition

Result

 (def jsmith {:name "John Smith" :age 50, :address {:postcode "sw 12ap" :Street "Royal Road"}})

 defines a map object called jsmith

 (get jsmith :name)

 get function returns the key value

 "John Smith"

 (get-in jsmith \[:address :postcode\])

 get-in function returns the nested key

 "sw 12ap"

(assoc jsmith :age 40)

assoc functions associates value to key

 (def jsmith {:name "John Smith" :age 40, :address {:postcode "sw 12ap" :Street "Royal Road"}})

(assoc-in jsmith \[:address :postcode\] "n1 0lk")

assoc-in  associate value to nested key

 (def jsmith {:name "John Smith" :age 50, :address {:postcode "n1 0lk" :Street "Royal Road"}})

 (update-in jsmith \[:age\] inc)

 update-in functions is used to apply a function to the key value inc  function increments  value by 1

 (def jsmith {:name "John Smith" :age 51, :address {:postcode "sw 12ap" :Street "Royal Road"}})

![NestedAcessMap](https://reshmeeauckloo.files.wordpress.com/2015/12/nestedacessmap.jpg) **Sets** Set contains unordered list of distinct values

Syntax

Definition

Result

 #{}

 empty set

 #{:a :b}

 set with keys a and b

  (#{:a :b} :a)

 returns key a from set

  :a

 (conj #{:b :c} :a)

conj function adds key to set

 #{c: :b :a}

(contains? #{:a :e} :e)

contains function return a Boolean

true

 (contains? #{:a :e} :h)

 contains function return a Boolean

 false

![Set_Data](https://reshmeeauckloo.files.wordpress.com/2015/12/set_data1.jpg) The screenshot above has been produced using lein repl as for some reasons LightTable from my laptop could not work out # symbol in front of {}. ![error#](https://reshmeeauckloo.files.wordpress.com/2015/12/error.jpg)