---
title: 'Clojure Destructuring'
date: Wed, 09 Dec 2015 09:55:41 +0000
draft: false
tags: ['Clojure', 'Clojure Destructuring', 'Functional Programming']
---

Destructuring allows to extract values from datastructures and bind them to symbols without explicitly looping/traversing the datastructure. It allows concise and simple coding.

### **Sequential Destructuring**

Provides vector of symbols to bind by position. **Syntax:** \[symbol another-symbol\] \["value" "another-value"\] `(def listNumbers[7 8 9 10 11]) ;;Bind a, b, c to first 3 values in stuff (let [[a b c] listNumbers] (list (+ a b) (+ b c)))` Result: (15,17) ![SequentialDestructuring](https://reshmeeauckloo.files.wordpress.com/2015/12/sequentialdestructuring.jpg) It binds to nil if there's no data, i.e. there are not enough items in the vector to bind to. ` (let [[a b c d e f] listNumbers] (list d e f)) ` Result : (10 11 nil) ![Nil_Destructuring](https://reshmeeauckloo.files.wordpress.com/2015/12/nil_destructuring.jpg) In the above example there is no data at 6th position in ListNumbers collection to bind to f explaining the nil being returned. **&** symbol can get "everything else" with value being a sequence ` (def listNumbers[7 8 9 10 11]) (let [[a & others] listNumbers] (list others)) ` Result : (8 9 10 11 ![&amp;_Others](https://reshmeeauckloo.files.wordpress.com/2015/12/others.jpg) \_  symbol is used for values that can be discarded First element is discarded in the example below ` (def listNumbers[7 8 9 10 11])` (let \[\[\_ & others\] listNumbers\] (list others))) ) Result : (8 9 10 11) ![_ Discard](https://reshmeeauckloo.files.wordpress.com/2015/12/discard.jpg)

### **Associative Destructuring**

Provides map of symbols to bind by key. **Syntax: {symbol :key, another-symbol :another-key} {:key "value" :another-key "another-value"}** `(def m {:a 7 :b 4}) (let [{a :a, b :b} m] [a b])` Result : \[7 4\] ![AssociativeDestructuring](https://reshmeeauckloo.files.wordpress.com/2015/12/associativedestructuring.jpg) **keys**  are inferred from vector of symbols to bind ` (def m {:a 7 :b 4}) (let [{:keys [a b]} m] [a b]) ` Result : \[7 4\] ![Keys_Map](https://reshmeeauckloo.files.wordpress.com/2015/12/keys_map.jpg) If a key is not found in the map, the symbol is bound to `nil`. ` (let [{:keys [a b c]} m] [a b c]) ` Result: \[7 4 nil\] ![Map_Nil](https://reshmeeauckloo.files.wordpress.com/2015/12/map_nil.jpg) **:or** provides default values for missing keys with a map of values ` (def m {:a 7 :b 4}) (let [{:keys [a b c] :or {c 3}} m] [a b c]) ` Result: \[7 4 3\] ![Or_Map](https://reshmeeauckloo.files.wordpress.com/2015/12/or_map.jpg) **Named Arguments ** Vector of keys can be bound to **&** to emulate optional named arguments. ` (defn people [planet & {:keys [adults children]}] (+ adults children))` (people "Earth" :adults 1 :children 5) Result:6 ![NamedArguments](https://reshmeeauckloo.files.wordpress.com/2015/12/namedarguments.jpg)