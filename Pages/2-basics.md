# Chapter 2: Basics

## Data Structures

### Strings, Numbers, Characters

Strings always need double quotes! Single quotes had special meanings so don't use them as string boundaries!

```
user=> (class "hello")
java.lang.String
```

Characters are like special types of strings which contain s single character, they are prefixed with a \.

```
user=> (class \a)
java.lang.Character
```

Numbers are actually instances of Java classes:

```
user=> (class 1)
java.lang.Long

user=> (class 1.5)
java.lang.Double
```

You can use them directly though without special instantiation.

### List 

The most basic contruct in Lisp is the List ( ) indicated by parenthesis. It is called a s-expression or sexp. Everything in Clojure starts as a List. The first thing in a list is always the name of a function, if the list only contains data it starts with a single apostophe `\`(1 2 3)` or like `(list (1 2 3))`

### Vector

This is an collection of values. They do not need to be the same type!

```
user=> (vec 1 2 3)
[1 2 3]


user=> [1 "hello" \a]
[1 "hello" \a]

;; Created from a list 
user=> (vec '(1 2 3))
[1 2 3]
```

### Keywords

Keywords start with a colon and are symbolic identifiers that evaluate to themselves. Similar to ruby symbols. 

### Map

A map is a key/value data structure defined like this: 

```
{:name "Bob :address "123 Awesome St"}
```

Keywords are often used as keys although strings *may* be used but it't not as common.

## Language Contructs 

### Namespaces

Namespaces are containers for your functions. If you are an OOP programmer you can _imagine_ it is like a class but only for about 5 seconds because you can't instantiate it! But just imagine it, thinking of a class you recently wrote. 

We will see more about these later, but in a new file ther will be a `(ns my-awesome-namespace)` at the top of your file along with any libraries to require.  

### Vars

Variables are created with `def`

```
(def a "apple")

(def weight 5)
```

### Functions

Here's an anonymous function: 

```
((fn [name] (clojure.string/upper-case name)) "bob")
```

Oh boy, lots of parens! Lets break it down, starting in the innermost expression:

```
(clojure.string/upper-case name)
```

Calls the upper-case function from the clojure.string namespace, passing in the value represented by the name variable. 

```
(fn [name] (clojure.string/upper-case name))
```

This is a anonymous function (not named) that takes a `name` param and passes it to the `clojure.string/upper-case` function.

Putting it together it looks like ( `anonymous function` "bob") .. Remember, first thing in the  ( ) is a function, then params for the function. see? not so bad!! 

Clojure defines a named function using this syntax:

```clojure
(defn name
  optional-doc-string?
  params-vector
  expression)
```

1. `defn` - introduces the defn expression.
2. `name` - what you call the function.
3. `doc-string?` - an optional description of the function.
4. `params-vector` - a vector of symbols naming the functions arguments.
5. `expression` - the body of the function.

Example: 

`hello-world`, a traditional first function, might be programmed in Clojure like:

```clojure
(defn hello-world
  "ye olde 'Hello, World'"
  []
  "Hello, World")
```
 

 
 
Next [Chapter 3: REPL](/Pages/3-repl.md)
