# Chapter 3: Starting The Project

### Creating a Clojure project

We're going to start by asking Leiningen to start a web application for us.

```
  $: lein new compojure chatter
```


`lein` is a tool for managing Clojure projects.

The command above requests permission to create a new compojure project called "Chatter". This command results in the creation of a directory`chatter` which appears as a folder in your computer's directory (Finder on Mac, Windows Explorer on Windows)

In the terminal, move into the project "Chatter" directory by entering,

```
  $: cd chatter
```

Start the server with this command

```
  $: lein ring server
```

`lein` will download a bunch of stuff from the Internet the first time it runs. After this finishes, your default browser opens a page that says, "Hello World".

Notice the address bar.

* "localhost" is an alias for your computer.

* 3000 is the port number where your server is listening for requests.

In the web browser, right-click and select `View Page Source`. You see that it's not even an HTML document, it's just the _string_ "Hello World".

In the terminal you see the message, "Started server on port 3000". This means your server is working and you will be able to view your app in the browser. We initiated this connection earlier when we entered `lein ring server` in the command line.

Stop the server connection by hitting "Ctrl + C" in the command line.

Let's take a closer look at what's in the chatter directory. In LightTable, it looks like this:

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

1. `GET`: this is the type of HTTP request we want to handle. GETs are requests for information or data.
2. `"/"`: this is the name of the web page.  `/` means the top level.
3. `[]`: this is an empty parameter vector.  When you do a search or fill out a form, the parameters narrow the result.
4. `"Hello World"`: the result that gets sent back to the requesting browser.

After the GET request is the second route definition `app-route`, which is:

```clojure
(route/not-found "Not Found")
```

This means when the server gets any kind of request other than GET, it should return "Not Found."


In the editor, the third and final expression of the `handler.clj` file says,

```clojure
(def app ...
```

`def` is how you declare a variable in Clojure.
The format is: `(def name doc-string? init?)`

1. `def`: introduces the def expression.
2. `name`: the name you want to give the variable.
3. `doc-string?`: an optional description on what the variable is and how it is meant to be used.
4. `init?`: an optional value the variable will be set to. If unset, the variable is unbound.

The _name_ of the variable is `app` and it's being assigned the result of ```(wrap-defaults app-routes site-defaults)```

* `wrap-defaults` is imported from the ring library, and it combines our routes and the defaults. When we start the server, it's going to look for the routes associated with the `app` variable and use those to decide how to handle HTTP requests.
* `app-routes` is what we're calling our set of routes
* `site-defaults`are default request/response handling details imported into the namespace from ring library.


Stop the server by going back to the terminal and holding the "Ctrl+C".


Next [Chapter 5: Build Page](/Pages/5-build-page.md)

