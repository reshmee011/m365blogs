---
title: 'Clojure If, cond, case'
date: Sun, 13 Dec 2015 21:33:00 +0000
draft: false
tags: ['Clojure', 'Clojure If', 'cond case condp', 'Uncategorized']
---

Clojure has lots of operators for dealing with conditional statements: if, cond, condp, case and when. Each of these is useful for different situations. Let's take a look at them in turn: **if** **if** expression in Clojure returns a value similar to ternary operator in c# compared to a statement The ternary expression in C# using the ? operator as follows `remark = (rating>10) ? "positive":"negative"` The expression is evaluated, if it is true the "then" part is executed and result returned otherwise the "else" part is  run and returned. In Clojure the syntax is `if( condition value-if-true value-if-false)` In Clojure if on false and nil return false and the rest return true. A sequence on empty collection returns nil , thus falsey.

Syntax

Definition

Result

(if true :truthy :falsey)

truthy

(if (Object) :truthy : falsey) ;

objects are true

truthy

(if \[\] :truthy : falsey);

empty collections are true

truthy

(if false  :truthy  :falsey)

 falsey

(if nil :truthy :falsey)

 nil is false

falsey

(if (seq \[\]):truthy :falsey);

seq on empty collection is false

falsey

The "else" in clojure if is optional

Syntax

Definition

Result

if(even? 6) "even" "odd")

 if expression with "else" part

even

if(even? 6) "even")

  if expression without "else" part

even

if(even? 7) "even")

  if expression without "else" part but evaluates to false.

nil

**if/do** The do operator allows to wrap multiple expressions per branch in each of the if branch. The last value of the expression is returned. `(if (even? 5) (do (println "even") "not odd") (do (println "odd") "not even"))` ![OddNotEven](https://reshmeeauckloo.files.wordpress.com/2015/12/oddnoteven.jpg) In the above example odd in printed in REPL and "not even" is returned as the value of the if operation.

#### when

The when operator is like a combination of if  and do but with no else branch. For example

(when true (println "true"))

![when.JPG](https://reshmeeauckloo.files.wordpress.com/2015/12/when.jpg)

**if-let** if-let is useful when the result is needed `(defn filter-evens [coll] (if-let [evens (seq (filter even? coll))] (println (str "Evens are:" evens)) (println "No evens.")))` ![filter_Evens](https://reshmeeauckloo.files.wordpress.com/2015/12/filter_evens.jpg) ` (filter-evens [1 2 3 4]) ;;Evens are : (2 4) (filter-evens [1 3]) ;;No evens ` In above example, the result of applying even filter to the collection is bound to collection evens. Empty collections return true while empty sequences return false. The seq operator is used to convert the collection into a sequence object so that if none of the values in the collection are even, false is returned and else part is executed instead of true returned. **cond** Unlike if cond can take a series of tests. For each test , there needs to be an accompanying expression. If one test evaluates to true, the expression is returned : else expression is optional and evaluated if none of the tests evaluate to true. (let \[x 5\] ;;binds 5 to x (cond (< x 2) "x is less than 2" (< x 10) "x is less than 10" :else "x is greater than or equal to 10")) ![cond](https://reshmeeauckloo.files.wordpress.com/2015/12/cond.jpg) In above example,  the value 5 is bound to symbol x and a series of tests are applied with a corresponding return value. The number 5 met the test "(< x 10)" thus returning "x is less than 10". **condp** condp is similar to cond except it takes a shared predicate (condition), instead of passing  test expressions, test values are passed. ` (defn evaluate [x] (condp > x 5 "x is less than 5" 10 "x is less than 10" "x is greater than or equal to 10")) ;; default argument to condp without test value (evaluate 9) (evaluate 3) (evaluate 120) ` ![condp.JPG](https://reshmeeauckloo.files.wordpress.com/2015/12/condp.jpg) **case** case is similar to switch case in C# or  JavaScript. The expression corresponding to the value passed will be evaluated. A default expression can be specified without any test value if there is no matching clause for the value passed. ![case.JPG](https://reshmeeauckloo.files.wordpress.com/2015/12/case.jpg) ` (defn value [x] (case x 5 "x is 5" 10 "x is 10" "x isn't 5 or 10")) (value 11) ;;"x isn't 5 or 10" (value 10);;"x is 10`