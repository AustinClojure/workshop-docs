# REPL

Start the REPL by typing `lein repl`. You are automatically placed into the user namespace.

Using the `hello-world` function from the previous chapter in the REPL would look like this: 

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
user> (clojure.repl/doc hello-world)
-------------------------
user/hello-world
([])
  ye olde 'Hello, World'
```

You can do `(doc hello-world)` if you have the `clojure.repl` namespace already loaded.

You can look up any function in Clojure! Here is a couple: 

```
user> (doc clojure.string/join)
-------------------------
clojure.string/join
([coll] [separator coll])
  Returns a string of all elements in coll, as returned by (seq coll),
   separated by an optional separator.
   
   
user=> (doc first)
-------------------------
clojure.core/first
([coll])
  Returns the first item in the collection. Calls seq on its
    argument. If coll is nil, returns nil.
````



Next [Chapter 4: Start Project](/Pages/4-start-project.md)
