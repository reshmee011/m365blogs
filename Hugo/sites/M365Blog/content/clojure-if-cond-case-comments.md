---
title: 'Clojure If, cond, case'
date: Sun, 13 Dec 2015 21:33:00 +0000
draft: false
tags: ['Clojure', 'Clojure If', 'cond case condp', 'Uncategorized']
---


#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2015-12-14 21:34:59">Dec 1, 2015</time>

@Timothy: Thanks for the clarification. I have updated the post.
<hr />
#### 
[g2-260e732e8993cfdedb432ca04381d6ca]( "philipp@meier.name") - <time datetime="2015-12-16 09:17:21">Dec 3, 2015</time>

You might consider these conditional macros, too: when-let, cond->, cond->> and maybe some->
<hr />
#### 
[g2-260e732e8993cfdedb432ca04381d6ca]( "philipp@meier.name") - <time datetime="2015-12-16 09:18:43">Dec 3, 2015</time>

Alsobe aware that case tests for identity, not for equality.
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2015-12-14 09:31:58">Dec 1, 2015</time>

Hi Timothy Thanks for your feedback. I agree that empty collections are truthy as in the example you provided. However empty sequences are falsey (if (seq \[\]):truthy :falsey); ;;=> falsey
<hr />
#### 
[timothypbuckley]( "timothypbuckley@gmail.com") - <time datetime="2015-12-14 15:21:38">Dec 1, 2015</time>

(seq \[\]) is falsey because it converts an empty sequence (which is truthy) to nil (which is falsey). So it's not correct to say that an empty sequence is falsey, because by themselves (be they an empty set, hashmap, vector or list), they are truthy. (seq nil) ;;=> nil (seq '()) ;;=> nil (seq "") ;;=> nil (seq '(1)) ;;=> (1) (seq \[1 2\]) ;;=> (1 2) (seq "abc") ;;=> (\\a \\b \\c) https://clojuredocs.org/clojure.core/seq
<hr />
#### 
[timothypbuckley]( "timothypbuckley@gmail.com") - <time datetime="2015-12-14 07:00:58">Dec 1, 2015</time>

"In Clojure the syntax is if( boolean then optional)" You miswrote the syntax, it's: (if condition val-if-true val-if-false ) Where val-if-false is nil if not provided. "In Clojure false, nil and empty sequence return false and the rest return true." Empty sequences are truthy, not falsey. (if \[\] "truthy" "falsey") ;;=> "truthy"
<hr />
