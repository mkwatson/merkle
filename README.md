# Merkle

A Clojure library for computing and comparing hash trees over sorted kv
collections. Allows you to efficiently find the differing pairs between two
such collections without exchanging the collections themselves. Useful in the
synchronization of distributed systems.

Note: kv.linear is not as efficient as it could be at identifying linear
regions.

Note: kv.linear has no way to limit the depth of the trees it produces right
now.

Note: kv.linear rarely identifies regions as identical which are not actually
so; might be an issue with hash collisions over small-cardinality values like
bytes. If values are bytes, deltas between hashes on the order of 10-1000
entries may miss differences around 5% of the time. If values are ~10-20
character strings, diffs are incomplete less than 1 in 10000 tries.

## Usage

```clj
(use 'merkle.kv.linear)

; Set up two maps with some differences
(def map1 (sorted-map :a 1 :b 2 :c 3 :d 4))
(def map2 (sorted-map :a 1 :b 2 :c 0))

; Compute a merkle tree of each
(def t1 (tree map1))
(def t2 (tree map2))

; Find pairs of map1 which, if applied to map2, would make it a superset of
; map1:
(def d1 (diff map1 t1 t2))
; => ([:c 3] [:d 4])

; And the inverse:
user=> (def d2 (diff map2 t2 t1))
; => ([:c 0])

; We can merge map2's differences back into map1:
(into map1 d2)
; => {:a 1, :b 2, :c 0, :d 4}

; And merge map1's differences into map2:
(into map2 d1)
; => {:a 1, :b 2, :c 3, :d 4}

; Provided a commutative merge function, exchanging diffs is monotonically
; convergent:
(merge-with max map1 d2)
; => {:a 1, :b 2, :c 3, :d 4}
(merge-with max map2 d1)
; => {:a 1, :b 2, :c 3, :d 4}
```

## License

Copyright © 2013 Kyle Kingsbury

Distributed under the Eclipse Public License, the same as Clojure.
