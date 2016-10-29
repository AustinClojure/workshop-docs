# Chapter 2: Basics

## Outline

### Datatypes
* List
* Vector
* Map
* Set

### Doing something
* Using a function
* Writing a function
* Defining a var
* Namespaces
  


## More Reading

### Dive, Dive!

We are working under the assumption that the reader has some programming experience under their belt.  We presume such experience is with a mainstream imperative language.


Often when someone fluent in another programming language encounters Clojure (or Lisp) code for the first time, the reaction is akin to staring into an abyss of complexity, confusion and puzzlement.

Lots of parens mixed with familiar and unfamiliar words, no obvious structure, etc.  Whiskey,  Tango, and Foxtrot already?

In reality, what is going on is actually quite a bit simpler than what you might be using every day, but things do work differently, and need to be thought about differently.  And this thinking differently is where the benefit seems to lie.

So let's dive all the way to the bottom and climb back out, hopefully dispelling any lingering mysticism.

It's way simpler than you might be expecting.

### Data Structures- the building blocks

There are three very important things to keep in mind when you are trying to get your head around Lisp, and by extension Clojure, source code:

  1. code is data
  2. code is data
  3. and finally, **code is data**.

The stuff you build in your text editor is just that - **text**.  That text is then processed by an entity known as the **reader**, which (barring errors) will transform it into **data**.

It does this be employing a certain lexicographical rules to the incoming text which interprets certain sequences of characters as **literals**.  

In Clojure, these are examples of things that are treated as **literals**:


| literal            | internal type  | explaination |
| -------------    | -------------  | -------------|
| nil              | nil  | the non-value, e.g. NULL / missing |
| false      | java.lang.boolean |  logical false |
| true       | java.lang.boolean |  logical true |
| 0, -42, 105  | java.lang.Long  | integer numbers |
| 1.1, -1.2e20 | java.lang.double | ieee floating point numbers |
| 42.3M , 1M | java.math.BigDecimal | arbitrary precision numbers |
| 1/3, 22/7 |  clojure.lang.Ratio | numbers that are repreented internally as a ratio |
| eggs&ham, y+2,  peach, whiskey  | clojure.lang.Symbol | symbols (note the "&" and "+" are not special!) |
| :name, :owner | clojure.lang.Keyword | keywords - special - they are interned |
| \H, \e, \l, \l \o | java.lang.Character | single character constants |
| "apple", "red"    | java.lang.String | string literal|

This should all feel pretty natural to anybody that has used any programming language, although there might be a few surprises:

   - Characters that you might expect to be "special" are not - aka there are no hard tokens like '=', '+' '&' etc.  In Clojure, these are just part of possible characters in symbols.

   -  Symbols usually don't have an analog in other languages - perhaps they are what you might construe as an 'identifier' in those languages.

   - Keywords are likely a new concept as well.  These are human-readable strings that get "interned" into machine-friendly values.

   - character constants, the '\\' .vs. single  quote "'".  In Lisps, the single-quote has an entirely different role (it ends up turning into the (quote) special form which will only serve to confuse at this point).

### Values and Evaluation

A subtle point exists between the notion of an **literal** 3042 and the integer **value** 3042, which is obtained by **evaluating** the literal.  It's easy overlook, since they "appear" to be essentially the same thing.

But there are other sources of **values** too.  These are **containers** (collections of values), evaluation of a **function** (transform a set of input values into an output value), and a **function** itself is also a value (a concept known as **first class** functions)

### Containers   

| reader syntax | explaination
| --------------    | -------------
| ( v ... )  | a **list** (think singley-linked)
| [ v ... ]  | a **vector** (random accessable)
| #\{ v ... \} | a **set** (distinct values)
| \{ k v ... \} | a **map** (associated key/value pairs)

Containers compose recursively because they contain values, and containers are themselves values.

```
(1 2 3)          ; a list

(1 (2 3 4) 5)    ; a nested list

(1 {:name "fred" :age 42} (4 (5 6))) ; even nuttier

(sing "song") ; list of a symbol and a string

```

### Forms are the Code
A list is clearly **data**.  But a list can also be **code** if it structured in such a way which allows it to be **evaluated**.  Lists of this sort are called **forms**, and this is the stuff we use to build our **code**.  So **code** is just **data** that can be **evaluated**.

so this is a list (consisting of a symbol, integer, integer), which also happens to be a tiny program.

```
(+ 10 20)
```

We can experiment directly with building these 'tiny programs' and seeing the outcome, for example:

```Cloujure
user> 10          ;; evaluate a simple literal
10

user> +           ;; the value of symbol '+',
                  ;; which is the built-in function '+'
#function[clojure.core/+]

user> (+ 10 20)   ;; evaluate a form (call function  
                  ;; '+' with two arguments)
30

user> (* 5 (+ 10 20))  ;; and of course we can nest these...
150

```
And now we are starting to see the **code**, which consists of a number of nested **forms** that can be **evaluated** to produce **values**.

If you understand how the internals of a typical compiler might work, you probably recognize that what we have effectively specified the Abstract Syntax Tree (AST) of our program using nothing except for data.

It's really that simple.  You start with a library of functions and special forms, use these to compose more functions and forms, which you eventually use and compose to implement whatever program you set out to build.

You are in effect building upon and continuing a bootstrap process beginning from general tools (the built in primitives) to the specific ones (your libraries and programs).

Many Lisps have been implemented starting with only a very minimal kernel (maybe a dozen forms) and bootstrapping into full-blown implementation building upon that minimal kernel.

Clojure itself does a considerable amount of this type of bootstrapping to implement its own core library, so a great deal of Clojure is implemented in Clojure itself.  There is a formidable Java kernel to be sure, but there is also a lot of Clojure involved in the implementation.

So there is no division of "us .vs. them" when it comes to what you can build this way.  Are you extending the language, or writing your program?  Is there really any difference?  Did you ever want/need there to be?

### Namespaces

Namespaces are used to segregate names into organized subdivisions.

Clojure code is always running in some namespace, and this is typically an amalgam of your code plus some other namespaces you have 'used' (and henced merged) with yours through a variety of techniques.

The namespace 'clojure.core' contains all the core language elements, and typically you want these close at hand when writing your own code - so you merge it into yours.

Other namespaces can be referred to by spelling them out for example 'clojure.core/+' or 'org.goofco.mylib/foobar'.  It looks like a Java convention, because it pretty much is.

The topic can become complex, so for now, just be aware that they exist, and they keep names organized in separate namespaces.

### Vars

Variables are created with `def`

```
(def a "apple")

(def weight 5)
```

These get defined in the namespace of the file they are defined in.

Note that these are not the customary "bash in place" variables you might be used to.  We are simply **binding** a **symbol** to a **value** here, and making that available in the namespace we are working in.


### Functions
Functions - those formerly mystical **values** that can be **evaluated** to produce **values** from other **values** - where do they come from?

Why, there is a form to create them, of course:

```Clojure
(def uppername
  (fn [name]
   (clojure.string/upper-case name)))

```
Now we have a function that we can invoke which does what you would expect.  It **binds** name to the argument passed, and then uses that to call another function ('upper-case', which is in the namespace 'clojure.string') and returns that value as its result.

```Clojure

(uppername "gilgamesh")  ;; evaluate this function
==> "GILGAMESH"

```
In fact, that particular drill is so common, there is a short-cut (defn):

```Clojure
(defn uppername
    "a function to convert a name to upper case"
    [name]
    (clojure.string/upper-case name))
```
accomplishes the just same thing, and also lets you add some documentation as well.

Clojure defines a named function using this syntax:

```clojure
(defn name
  optional-doc-string?
  params-vector
  exprs*)
```

1. `defn` - start of the 'defn' form
2. `name` - what you call the function.
3. `doc-string?` - an optional description of the function.
4. `params-vector` - a vector of symbols naming the functions arguments.
5. `exprs*` - expressions that make up the function.  The value of the last one is the return value from the function.

The 'defn' form is commonly used a lot to create functions you will refer to later in your code.

But functions really don't need names at all - they can be anonymous, which is useful in situations where the function is simply used as an argument to another function.  Function definitions are just values, and they can be used and forgotten just like other values.

An example of this would be found in something like the **map** function, whose job is to transform a sequence (collection) into a new sequence by applying a function to each value in the source collection:

```Clojure
(map (fn [name]
         (clojure.string/uppercase name))
      ['Billy' 'Bob' 'Sally'])
```      
In this case, we simply created an anonymous temporary function, used it in map, and then forgot about it.  Functions are just values, and they can be passed to to other functions, stored in collections, or use just like any other value.  This is where the concepts of "functional programming" and "first class functions" begin.


Next [Chapter 3: REPL](/Pages/3-repl.md)
