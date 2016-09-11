# Chapter 5: More Code Changes

Now let's change the app's main page from "Hello, World" to something a little more chatty.

### Adding Hiccup

We need to write code that will generate HTML. To do this, we will use a library called `hiccup`. We don't have this library yet, so we're going to add it. Adding a new library requires two steps:

1) Add the library to the dependency section of the `project.clj` file. This tells lein your program needs another program.

Add hiccup by updating the `project.clj` file to look like this:

```clojure
  :dependencies [[org.clojure/clojure "1.8.0"]
                 [compojure "1.4.0"]
                 [ring/ring-defaults "0.1.5"]
                 [hiccup "1.0.5"]]
```

2) Import the library into the namespace you will use it in by adding the import to the `ns` declaration. Our ns declaration will be part of the `handler.clj` file:

```clojure
(ns chatter.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [hiccup.page :as page]))
```

Let's use hiccup to generate the html by changing `app-routes`:


```clojure
(defroutes app-routes
  (GET "/" []
       (page/html5
        [:head
         [:title "chatter"]]
        [:body
         [:h1 "Our Chat App"]]))
  (route/not-found "Not Found"))
```

Once the code is updated, let's try it out. In the command line, start the server:

    $: lein ring server

Now `http://localhost:3000` displays "Our Chat App".  Right-click and select `View Page Source` to see  it's now proper HTML complete with `head`, `title`, and `body`.

The hiccup function(*) `page/html5` generates an HTML page. It expects Clojure vectors with symbols representing corresponding HTML tags. Hiccup will automatically add the closing tag when it reaches the end of the vector.


Compare the hiccup to HTML in `View Page Source` to the HTML we wrote by hand earlier.

#### How Does Hiccup Work?

>_Vectors_ are a Clojure data structure used to contain sequences of things, including other vectors. Vectors are often written using square brackets. For example, `[1 2 3]` is a vector containing the numbers 1, 2, and 3. Hiccup uses vectors of keywords to represent sections of HTML.

>
>_Keywords_ are names that begin with a colon.  `:title`, `:x`, and `:favorite-color` are all keywords. Clojure often uses keywords where other languages use strings. If you were to use Clojure to query a database, Clojure would probably use keywords to represent the column names.  Hiccup uses keywords to represent the names of HTML elements.  

> Where HTML uses `<body>`, hiccup would expect `:body`. Where HTML uses `<title>`, hiccup uses
> `:title`. Because the keywords are enclosed in a vector, the closing of the HTML tag is unnecessary.  The closing of the surrounding vector signals where the HTML section ends.

A problem with our new `app-routes` is that it has two different functions right now. Its main role is to take the incoming request and decide what to do.  Right now it's doing that, but it is also generating a full HTML page. As we add more pages, this will become too complicated to manage. We'll get ahead of the game by splitting out the task of generating the HTML into a helper function.

```clojure
(defn index-page
  "This generates the HTML for displaying messages"
  []
  (page/html5
   [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]]))

(defroutes app-routes
  (GET "/" [] (index-page))
  (route/not-found "Not Found"))
```

`index-page` is a function that takes no arguments (for now!). It calls a hiccup function `page/html5` to generate html from a vector representing the `head` sections and a vector representing the `body` elements of the html.

Save `handler.clj`, and refresh the browser to make sure our page still works. From the outside, we shouldn't see a change. The page should still display "Our Chat App" and the html should be identical. Now, let's double check our git status:



### Adding Messages

Our app is not displaying messages, nor do we have a way of adding messages. Let's make that happen now.

Let's change the app so it displays messages. We'll represent the messages as a vector of maps. Each map will have a `:name` and `:message`key and the corresponding value, in quotes.  For example, the code below will represent blue's first post.

```clojure
(def messages {:name "blue" :message "blue's first post"})

```

This is a map with two keys named `messages`. 
<ol>
<li>`:name` is "blue", because blue posted it</li>
<li>`:message` is the content of the post and its value is "blue's first post".</li>
</ol>

To get a value from a map, pass the map and key into the `get` function. For example,

```clojure
(get messages :name)
```
When the keys are keywords, you can also use the keyword as a function that takes the map and returns the values.

```clojure
(:name messages)
```
returns `"blue"`.

Maps are everywhere in Clojure and are used for many things where other languages might use objects.

Let's call the vector simply `chat-messages` and hard code some samples to get started. Add a chat-messages variable to `handler.clj`.

After the ns expression, add:

```clojure
(def chat-messages [{:name "blue" :message "hello, world"}
                    {:name "red" :message "red is my favorite color"}
                    {:name "green" :message "green makes it go faster"}])
```

Next, we'll modify the HTML to display the messages.  We will also add a parameter to the `index-page` function so that we can give it a messages we want displayed.

```clojure
(defn index-page
  "This generates the HTML for displaying messages"
  [messages]
  (page/html5
   [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]
    [:p messages]]))
(defroutes app-routes
  (GET "/" [] (index-page chat-messages))
  (route/not-found "Not Found"))
```

Save `handler.clj` and refresh the browser. 

This blows up spectacularly.

![](https://github.com/clojurebridge-minneapolis/track1-chatter/blob/master/images/illegal-argument-exception.jpg "illegal-argument-exception")

[]()

This is a stack trace - it gives us an idea what the program was doing when it hit the problem. Ignore all the files that aren't ones you wrote for the project. In my case, the first file of interest is
`handler.clj`, line 14, the `index-page` function.

The exception message on the top, `"... is not a valid element name"`, is a clue to what's wrong.  Elements are what fragments of html are called.  Hiccup is responsible for generating html from Clojure symbols. The problem is that we've got a map with symbols in it and hiccup thinks they're html.  They're not, so it creates an error.

We can fix the issue by converting our maps to strings.

```clojure
(defn index-page
  "This generates the HTML for displaying messages"
  [messages]
  (page/html5
   [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]
    [:p (str messages)]]))
```

Save `handler.clj`, and refresh the browser.

This fixes the exception but it's ugly.

![ugly hack](https://github.com/clojurebridge-minneapolis/track1-chatter/blob/master/images/ugly.jpg "ugly hack")

Let's take the messages and put them in a table using HTML's `table`, `tr`, and `td` elements.  We're going to write a function that takes a
message and creates an HTML row. Then, inside a `table`, we're going to apply that function to all of our messages.



```clojure
(defn table-row [data]
  [:tr [:td (:name data)] [:td (:message data)]])
```

Now our `index-page` looks like:

```clojure
(defn index-page
  "This generates the HTML for displaying messages"
  [messages]
  (page/html5
     [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]
    [:p
     [:table
      (map table-row messages)]]]))
```

The `map` function takes a function and a collection and calls it on each element. But our `table-row` function is not that complex and we only need that function here, so lets make it an Anonymous function!

Clojure uses `defn` to create a function, but those functions are named. Sometimes, we want a specialized function that isn't reusable. For those cases, Clojure has a way of creating an anonymous function.

```clojure
(fn params-vector expression)
```

Our function is going to take a message, we'll call it "m" within the function, and extract both the `:name` and `:message`, wrapping them in `:td` to make table cells and putting them both within a `:tr` to make the row.  Since the keys to
the message hashmap are keywords, we can use them as functions to get the values.  In Clojure, the anonymous function looks like:

```clojure
(fn [m] [:tr [:td (:name m)] [:td (:message m)]])
```

Making a new collection by applying a function to all of the elements of an existing collection is such a common thing that Clojure has it functionality predefined. It's a function called `map`. This is different than "mapping" (the function) over a collection of maps (hash tables), which is what we are doing.


Now our map function looks like this:

```clojure
(map (fn [m] [:tr [:td (:name m)] [:td (:message m)]]) messages)
```

Now our `index-page` looks like:

```clojure
(defn index-page
  "This generates the HTML for displaying messages"
  [messages]
  (page/html5
    [:head
      [:title "chatter"]]
    [:body
     [:h1 "Our Chat App"]
     [:p
      [:table
       (map (fn [m] [:tr [:td (:name m)] [:td (:message m)]]) messages)]]]))
```

Save `handler.clj`, then refresh the browser.  Our hard-coded messages should now display in the page.

![hard coded messages](https://github.com/clojurebridge-minneapolis/track1-chatter/blob/master/images/hardcoded.jpg "hard coded messages")


Next [Chapter 6: Add Form](/Pages/6-add-form.md)
