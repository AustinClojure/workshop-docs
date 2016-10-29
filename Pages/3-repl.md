# REPL and Setup

Startug the REPL:

* Using [NightCode](https://sekao.net/nightcode/) the REPL is the window in the bottom.
* Using the command line:  start the REPL by typing `lein repl`. You are automatically placed into the user namespace.


Consider this `hello-world` function:
 
 ```clojure
 (defn hello-world
   "ye olde 'Hello, World'"
   []
   "hello world")
 ```

`hello-world` takes no arguments and returns the string "Hello, World".

```
user> (hello-world)
"Hello, World"
```

If you forget the ( ) around the function name, you'll see something like this:

```
user> hello-world
\#object[user$hello_world 0xf388310 "user$hello_world@f388310"]
```

You can see the docs and parameters for a function by calling `doc`.

```
user> (doc hello-world)
-------------------------
user/hello-world
([])
  ye olde 'Hello, World'
```

You can look up any function in Clojure! Here are a couple examples: 

```
user> (doc clojure.string/join)
-------------------------
clojure.string/join
([coll] [separator coll])
  Returns a string of all elements in coll, as returned by (seq coll),
   separated by an optional separator.
   
   
user> (doc first)
-------------------------
clojure.core/first
([coll])
  Returns the first item in the collection. Calls seq on its
    argument. If coll is nil, returns nil.
````



Next [Chapter 4: Start Project](/Pages/4-start-project.md)
