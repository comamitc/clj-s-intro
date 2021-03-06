<style>
.reveal section img {
  border: none;
}
</style>

## CLJ(S) Intro
<img src="./clojure.png" width="250">
> Mitch Comardo, 2016

---

## Agenda

- Overview of Ecosystem
- Syntax
- Interop
- Fun Snippets!
- Some Tools

---

![Adventure!](./giphy.gif "Logo Title Text 1")

---

## Ecosystem

- Clojure(Script) is a LISP (**LIS**t **P**rocessor)
- Clojure - language that compiles to Java Byte Code
- ClojureScript - dialect of Clojure that targets JS
- Google Closure Compiler - used in the CLJS compilation process
- [Google Closure Library](https://google.github.io/closure-library/api/)
- Leiningen - JVM runtime build tool standard

---

## About Clojure(Script)

> **_Clojure has Great Defaults_**

- Clojure has a focus on simplicity
- Functional programs are easier to reason about; result in fewer lines of code
- Data is immutable by default
- Pure functions are the default
- Macros allow for extension of the language in a powerful way

---

## Closure Compiler Optimizations

How you wrote it:
```js
function hello(name) {
  alert('Hello, ' + name);
}
hello('New user');
```

With simple optimizations:
```js
function hello(a){alert("Hello, "+a)}hello("New user");
```

With advanced optimizations:
```js
alert("Hello, New user");
```

---

## Syntax

> Embrace the parens

---

## Simple

```clojure
; number
user=> 1.23
1.23

; string
user=> "foo"
"foo"

; keyword
user=> :foo
:foo

; vector
user=> [:bar 3.14 "hello"]
[:bar 3.14 "hello"]

; map
user=> {:msg "hello" :pi 3.14 :primes [2 3 5 7 11 13]}
{:msg "hello", :pi 3.14, :primes [2 3 5 7 11 13]}

; set
user=> #{:bar 3.14 "hello"}
#{"hello" 3.14 :bar}
```

---

## Less Simple

```clojure
; function
user=> (defn foo [& args] (apply str args))
#'user/foo

; symbol
user=> foo
#object[user$foo 0x439aa6a6 "user$foo@439aa6a6"]

; list (represents a "call")
user=> (foo :bar 3.14)
":bar3.14"
```

---

## `list` and evaluation

```clojure
; list is also a datastructure (that has some implications)
user=> (1 2 3)
ClassCastException java.lang.Long cannot be cast to clojure.lang.IFn  user/eval1254 (form-init966740207684451296.clj:1)

user=> (list 1 2 3)
(1 2 3)
; or
user=> '(1 2 3)
(1 2 3)
```

<small>_You generally don't use lists very often for typical sequential data in your programs_</small>

**More on data structures later**

---

## Let, Locals & Scope

```clojure
; let and locals
(let [width  10
      height 20
      depth  2]
  (println "hello from inside the `let`.")
  (* width
     height
     depth))
=> hello from inside the `let`.

; let blocks return the last expression's result
=> 400
```
```clojure
; You can redefine within a let
(let [x 2
      x (* x x)
      x (+ x 1)]
  x)
=> 5
```

---

## Data Structures

```clojure
; creating
(list 1 2 3)            ; ⇒ '(1 2 3)
(vector 1 2 3)          ; ⇒ [1 2 3]
(hash-map :a 1 :b 2)    ; ⇒ {:a 1 :b 2}
(hash-set :a :b :c)     ; ⇒ #{:a :b :c}
```

---

## Examine

```clojure
;; Vectors
(def v [:a :b :c])
(nth v 1)             ; ⇒ :b
(v 1)                 ; ⇒ :b  (same)
(first v)             ; ⇒ :a
(rest v)              ; ⇒ (:b :c)
(next v)              ; ⇒ (:b :c)
(last v)              ; ⇒ :c

;; Lists - same as vectors, but can't index.
```

```clojure
;; Maps
(def m {:a 1 :b 2})
(get m :a)            ; ⇒ 1
(m :a)                ; ⇒ 1       (same)
(:a m)                ; ⇒ 1       (same!)
(get m :x 44)         ; ⇒ 44      (if no :x, 44 is the default)
(keys m)              ; ⇒ (:a :b)
(vals m)              ; ⇒ (1 2)
;; Grab a key or a val from a single map entry:
(key (first m))       ; ⇒ :a
(val (first m))       ; ⇒ 1
;; Of course, note that maps are not ordered. (sorted-map)
```

---

## "Change" - Immutability

```clojure
;; Vectors
(def v   [:a :b :c])
(def li '(:a :b :c))
(conj v  :d)          ; ⇒ [:a :b :c :d]
(cons :d v)           ; ⇒ (:d :b :c :a)
(conj li :d)          ; ⇒ (:d :a :b :c)
(cons :d li)          ; ⇒ (:d :a :b :c)

v   ; ⇒ is still [:a :b :c]
li  ; ⇒ is still (:a :b :c)
```
```clojure
;; Maps
(def m {:a 1 :b 2})
(assoc m :c 3)        ; ⇒ {:a 1 :c 3 :b 2}
(dissoc m :b)         ; ⇒ {:a 1}

m   ; ⇒ is still {:a 1 :b 2}
```

---

## Sequences

> Just like in lodash, there are common functions that can be used across all sub-types of `Sequence`.

![Sequence!](./seq-funcs.png "Logo Title Text 1")

---

## namespaces

```clojure
; in foo.clj
(ns example.foo)

(defn my-add [& args] (apply + args))

; in bar.clj
(ns example.bar
  (:require [example.foo :as foo]))

(foo/my-add 1 2 3)
=> 6
; OR
(ns example.bar
  (:require [example.foo :refer [my-add]]))

(my-add 1 2 3)
```

---

## Functions

<small>Macros act as code-generating functions</small>

```clojure
; create a named function using the defn macro
; demostrates function arity
(defn greet
  "Takes a name returns a personalized (or generic) greeting!"
  ([] (greet "World"))
  ([username]
   (str "Hello " username)))

; the definition for the defn macro (over-simplified)
(defmacro myfn [name args body]
  `(def ~name (fn ~args ~body)))
```

---

## Func Utilities

```clojure
user=> (doc greet)
-------------------------
user/greet
([] [username])
  Takes a name returns a personalized (or generic) greeting!
nil

user=> (source greet)
(defn greet
  "Takes a name returns a personalized (or generic) greeting!"
  ([] (greet "World"))
  ([username]
   (str "Hello " username)))
```

---

## Destructuring

```clojure
; vectors
(def games [:chess :checkers :backgammon :cards])

(let [[game-a game-b game-c game-d] games]
  ...
  ...)

; map
(def concert {:band     "The Blues Brothers"
              :location "Palace Hotel Ballroom"
              :promos   "Ladies night, tonight"
              :perks    "Free parking"})

(let [{:keys [band location promos perks]} concert]
  ...
  ...)
```

---

## Mutability

```clojure
; atoms
(def my-atom (atom {:foo 1}))
;; ⇒ #'user/my-atom
@my-atom
;; ⇒ {:foo 1}
(swap! my-atom update-in [:foo] inc)
;; ⇒ {:foo 2}
@my-atom
;; ⇒ {:foo 2}
```

---

## MOAR!

There's so much more!
- `defmethod` & `defmulti`
- `defprotocol` & `defrecord`
- Threading Macros `(-> ...)`
- Conditional `let`

Check out
- http://clojure.org/api/cheatsheet
- http://cljs.info/cheatsheet/

---

## Interoperability - Clojure

```clojure
(.toUpperCase "fred")                  ; (.instanceMember instance args*)
-> "FRED"
(.getName String)                      ; (.instanceMember Classname args*)
-> "java.lang.String"
(.-x (java.awt.Point. 1 2))            ; (.-instanceField instance)
-> 1
(System/getProperty "java.vm.version") ; (Classname/staticMethod args*)
-> "1.6.0_07-b06-57"
Math/PI                                ; Classname/staticField
-> 3.141592653589793
```

---

## Interoperability - ClojureScript

_Let's get wild!_

```clojure
(def text js/globalName)
; JS output ⇒ namespace.text = globalName;

(def t1 (js/MyType.))
; JS output ⇒ namespace.t1 = new MyType;

; export makes sure the Compiler doesn't uglify this
; and attaches it to global
(def ^:export t2 (new js/MyType "Bob"))
; JS output ⇒ namespace.t2 = new MyType("Bob");
```

---

## Con't - Special Dot Notation

```clojure
;; CALLING METHODS
(.hello js/window "John")
; JS output ⇒ window.hello("John");

; OR
(. js/window (hello "John"))
```

```clojure
;; ACCESS MEMBERS
(aget js/object "prop1" "prop2" "prop3")
; JS output ⇒ object["prop1"]["prop2][prop3"];

; OR
(.. js/object -prop1 -prop2 -prop3)
; JS output ⇒ object.prop1.prop2.prop3;

```

---

## Con't - Objects

```clojure
(def js-object (js-obj  :a 1 :b [1 2 3] :c #{"d" true :e nil}))
; JS output ⇒ {":c" cljs.core.PersistentHashSet, ":b" cljs.core.PersistentVector, ":a" 1}

(def js-object (clj->js {:a 1 :b [1 2 3] :c #{"d" true :e nil}}))
; JS output ⇒ {"a": 1, "b": [1, 2, 3], "c": [null, "d", "e", true]}

(def js-object #js {:a 1 :b 2})
; JS output ⇒ {"b": 2, "a": 1};

(def cljs-object (js->clj #js {"b": 2, "a": 1} :keywordize-keys true))
; CLJS output ⇒ {:b 2 :a 1}
```

---

## Clojure Common Files (`.cljc`)

```clojure
(defn str->int [s]
  #?(:clj  (java.lang.Integer/parseInt s)
     :cljs (js/parseInt s)))
```

---

## some of my favorites

```clojure
; Anonymous functions
(fn [a b] (+ a b))

; OR
#(+ %1 %2)
```

```clojure
; Unused Variables in strict function signatures
(fn [db query _] (db/query query))
```

```clojure
; Threading Macros
(-> 1 (+ 2) (/ 4)) ;⇒ 0.75
; same as (/ (+ 1 2) 4)

(->> 1 (+ 2) (/ 4)) ;⇒ 1.333
; same as (/ 4 (+ 2 1))
```

---

# more favorites


```clojure
; function composition
(def comp-fn (comp #(+ 5 %) +))
(comp-fn 1 2 3) ;⇒ 11
```

```clojure
; partials
(def add-one (partial + 1))
(add-one 3) ;⇒ 4
```

---

## Tools

- [Leiningen](https://leiningen.org/) - Automate Clojure projects without setting your hair on fire
- [Figwheel](https://github.com/bhauman/lein-figwheel) - builds your ClojureScript code and hot loads it into the browser as you are coding
- [Clojars](https://clojars.org/) - dead easy community repository for open source Clojure libraries
- [CLJSJS](http://cljsjs.github.io/) - CLJSJS provides Javascript libraries and their appropriate extern files packaged up with `deps.cljs`
- [Parinfer](https://shaunlebron.github.io/parinfer/) - "parentheses inference for Lisp"

---



## Thanks

![:happy-mitch:](./27e5954c6bcd4905.png "Happy Mitch")
