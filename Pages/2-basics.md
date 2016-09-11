# Chapter 2: Basics

## Data Structures

### Strings, Numbers

### Vector

### Map


## Language Contructs 

### Namespaces

### Vars

### Functions

Clojure defines a function using this syntax:

 ```clojure
 (defn name
   doc-string?
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
 
 Later we will see how to create anonymous functions and when you would want to use one.
 
 
Next [Chapter 3: REPL](/Pages/3-repl.md)
