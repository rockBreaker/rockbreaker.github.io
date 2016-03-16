# I-hate-tech-blogs

My first blog post.
In this blog I will be choosing a clojure function a day to use, talk about and generally discuss. 

***WARNING MY SPELLING IS WORSE THAN MY CLOJURE***

******************************************************
#There's a function for that:

###First Post -  [mapv](https://clojuredocs.org/clojure.core/mapv).

Simply put, it's map but instead of returning a *lazy-sequence* it returns a *evaluated* vector. 

###Examples

````
(mapv inc [1 2 3])
=> [2 3 4]  ;; **An evaluated vector**
(map inc [1 2 3])
=> (2 3 4)   ;; An evaluated list (as its in a REPL, normally its a lazy-seq)

(mapv + [1 2 3] [2 3 4]) **multiple collection call**
=> [3 5 7]
(map + [1 2 3] [2 3 4])
=> (3 5 7)
````

###So why use mapv?
Short answer, it saves you time. 

You can just code `(into [] (map inc [1 2 3]))`

But mapv is a nice _shorthand_ [re-work this bit.] for doing the above.

Why do you want to return it as a vector?
I have no idea, thats your design choice. 



###Under the hood:

* Single collection
  * uses reduce and transient vectors.   

I think they use transient vectors as its more efficent to create a transient vector and change it then create a new vector. (so essentially an optimization).
[TODO- find out why they use reduce]
 
* 2-3    collections
  * uses map function coll-1 coll-2 coll-3 into a vector

I think they explicitly write out the 2 and 3 arity versions for efficiency? (check this is an optimization). 


* 3+     collections
  * uses  apply map function coll-1 coll-2 coll-3 &colls into a vector

This is the base case for mapv'ing over more than 3 collections. The reason its apply map function over the collections and &coll, is because under the hood they use & colls for more than 3 collections called with mapv. [re-work this bit. Talking about & coll is a list/seq of things.] 

[source-code](https://github.com/clojure/clojure/blob/clojure-1.7.0/src/clj/clojure/core.clj#L6607)

```
(defn mapv
  "Returns a vector consisting of the result of applying f to the
  set of first items of each coll, followed by applying f to the set
  of second items in each coll, until any one of the colls is
  exhausted.  Any remaining items in other colls are ignored. Function
  f should accept number-of-colls arguments."
  {:added "1.4"
   :static true}
  ([f coll]
     (-> (reduce (fn [v o] (conj! v (f o))) (transient []) coll)
         persistent!))
  ([f c1 c2]
     (into [] (map f c1 c2)))
  ([f c1 c2 c3]
     (into [] (map f c1 c2 c3)))
  ([f c1 c2 c3 & colls]
     (into [] (apply map f c1 c2 c3 colls))))
 
     