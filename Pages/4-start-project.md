.# Chapter 3: Starting The Project

### Creating a Clojure project

We're going to start by asking Leiningen to start a web application for us.

```
  $: lein new compojure chatter
```

The command creates a new compojure project called "Chatter". This command results in the creation of a directory `chatter`.

Start the server with this command

```
  $: lein ring server-headless
```

`lein` will download a bunch of stuff from the Internet the first time it runs. After this finishes, open your browser to `http://localhost:3000`. It will display a message "Hello World".

Stop the server connection by hitting "Ctrl + C" in the command line.

Let's take a closer look at what's in the chatter directory. It looks like this:

```
.
├── project.clj
├── README.md
├── resources
│   └── public
├── src
│   └── chatter
│       └── handler.clj
└── test
    └── chatter
        └── handler_test.clj

6 directories, 4 files
```

`project.clj` is a Clojure file that describes what our project does and what libraries it needs to run.

`README.md` is a _markdown_ file (.md), which describes this project.

`resources` is a folder where you store static HTML, CSS, and images.

`src` is where the source code lives.

`test` is where the tests are stored.

### A closer look at the src directory

In the editor, open the file `src/chatter/handler.clj`.

The file that ends with ".clj" indicates this is a Clojure file. Clojure programs are made up of _expressions_. Expressions are either a single name, number, string, or a list of expressions beginning with a paren (or parenthesis). These expressions make your app appear and function in a web browser.


In the `src/chatter/handler.clj` file, the first expression `(ns chatter.handler ...` tells Clojure what we want to call the namespace (ns) being defined in this file. In this case we want to call it "chatter.handler". You can think of a namespace as similar to a module in ruby or python. A container for some code.

The sub-expression, the expression below, begins with `(:require ...`. This is importing the ring and compojure libraries. Those are low-level Clojure libraries for building web apps.

The second expression is `(defroutes app-routes ...`. _defroutes_ is specific to Clojure web apps.  It creates a set of routes and gives them the name `app-routes`.  The expressions after symbol `app-routes` are the route definitions. There are two here.

The first is:

```clojure
(GET "/" [] "Hello World")
```

There are four parts to this expression:

1. `GET`: Name of function, GET. GETs are requests for information or data.
2. `"/"`: this is the url of the page.  
3. `[]`: this is an empty parameter vector.  When you do a search or fill out a form, the parameters narrow the result.
4. `"Hello World"`: the result that gets sent back to the requesting browser.

After the GET request is the second route definition `app-route`, which is:

```clojure
(route/not-found "Not Found")
```

This means when the server gets any kind of request other than `GET /`, it should return "Not Found."

Stop the server by going back to the terminal and doing "Ctrl+C".

Next [Chapter 5: Reorganize some things](/Pages/5-reorg.md)

