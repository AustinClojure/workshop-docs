# Chapter 6: Building the Index Page

Now let's change the app's main page from "Hello, World" to something a little more chatty.

### Adding Hiccup

We need to write code that will generate HTML. To do this, we will use a library called `hiccup`. We don't have this library yet, so we're going to add it. Adding a new library requires two steps:

1) Add the library to the dependency section of the `project.clj` file.

Add hiccup by updating the `project.clj` dependencies vector to include this:

```clojure
[hiccup "1.0.5"]
```

2) Import the hiccup libraries into the handler namespace by adding to the `ns :require` declaration. Include these in `(:require )` form.

```clojure
[hiccup.core :as hiccup]
[hiccup.page :as page]
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

Once the code is updated, let's try it out. 

```
$: lein ring server-headless
```

Refresh your browser.

Now `http://localhost:3000` displays "Our Chat App".  Right-click and select `View Page Source` to see  it's now proper HTML complete with `head`, `title`, and `body`.

The hiccup function `page/html5` generates an HTML page. It expects Clojure vectors with symbols representing corresponding HTML tags. Hiccup will automatically add the closing tag when it reaches the end of the vector.

#### How Does Hiccup Work?

>_Vectors_ are a Clojure data structure used to contain sequences of things, including other vectors. Vectors are written using square brackets or the `(vec)` function. For example, `[1 2 3]` or `(vec 1 2 3)` are vectors containing the numbers 1, 2, and 3. Hiccup uses vectors of keywords to represent sections of HTML.

>_Keywords_ are names that begin with a colon.  `:title`, `:x`, and `:favorite-color` are all keywords. Clojure often uses keywords where other languages use strings. If you were to use Clojure to query a database, Clojure would probably use keywords to represent the column names.  Hiccup uses keywords to represent the names of HTML elements.  

> Where HTML uses `<body>`, hiccup would expect `:body`. Where HTML uses `<title>`, hiccup uses
> `:title`. Because the keywords are enclosed in a vector, the closing of the HTML tag is unnecessary.  The closing of the surrounding vector signals where the HTML section ends.

#### Green, Refactor, Green
Ok before we fix some problems lets add some passing tests. I know right, well better late than never, and especially as a scaffold for our upcoming changes. Open `test/chatter/handler_test.clj` file and then replace the contents with:

```clojure
(ns chatter.handler-test
  (:use [kerodon.core]
        [kerodon.test]
        [clojure.test])
  (:require [chatter.handler :refer [app]]))

(deftest test-app
  (testing "main route"
    (-> (session app)
        (visit "/")
		(has (status? 200) "page is found")
		(has (some-text? "Our Chat App"))))

  (testing "not-found route"
    (-> (session app)
        (visit "/invalid")
		(has (status? 404) "page not found"))))
```

These are two interaction tests, two tests covering the projected generated scenarios, plus one additional to cover code we have already written.

```
$: lein test
```

Confirm that all three assertions have passed and that there are no failures or errors, then continue on to the refactor.

#### Refactor

A problem with our new `app-routes` is that it has two different functions right now. Its main role is to take the incoming request and decide what to do.  Right now it's doing that, but it is also generating a full HTML page. As we add more pages, this will become too complicated to manage. We'll get ahead of the game by splitting out the task of generating the HTML into a helper function. Add a function `index-view` and call it in the `GET` function:

```clojure
(defn index-view
  "This generates the HTML for displaying messages"
  []
  (page/html5
   [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]]))

(defroutes app-routes
  (GET "/" [] (index-view))
  (route/not-found "Not Found"))
```

The function `index-view` takes no arguments (for now!). It calls a hiccup function `page/html5` to generate html from a vector representing the `head` sections and a vector representing the `body` elements of the html.

Save `handler.clj`, and refresh the browser to make sure our page still works. From the outside, we shouldn't see a change. The page should still display "Our Chat App" and the html should be identical.

Confirm that all three assertions have passed and that there are no failures or errors, pat yourself on the back.

### Adding Messages

Our app is not displaying messages, nor do we have a way of adding messages. Let's make that happen now.

Let's change the app so it displays messages. We'll represent the messages as a vector of maps. Each map will have a `:name` and `:message`key and the corresponding value, in quotes.  For example, the code below will represent blue's first post.

```clojure
(def messages {:name "blue" :message "blue's first post"})
```

This is a map with two keys named `messages`. 

`:name` is "blue", because blue posted it.<br/>
`:message` is the content of the post.


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

Before your `index-view` function add:

```Clojure
(def chat-messages [{:name "Bob"    :message "hello, world"}
                    {:name "George" :message "What's up internet"}
                    {:name "Sally"  :message "Hungry for some pizza?"}])
```

Next, we'll modify the HTML to display the messages.  We will also add a parameter to the `index-view` function so that we can give it the messages we want displayed.

```clojure
(defn index-view
  "This generates the HTML for displaying messages"
  [messages]
  (page/html5
   [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]
    [:p messages]]))

(defroutes app-routes
  (GET "/" [] (index-view chat-messages))
  (route/not-found "Not Found"))
```

Save `handler.clj` and refresh the browser. 

This blows up spectacularly.

![](https://github.com/clojurebridge-minneapolis/track1-chatter/blob/master/images/illegal-argument-exception.jpg "illegal-argument-exception")

This is a stack trace - it gives us an idea what the program was doing when it hit the problem. Ignore all the files that aren't ones you wrote for the project. In my case, the first file of interest is
`handler.clj`, line 14, the `index-view` function.

The exception message on the top, `"... is not a valid element name"`, is a clue to what's wrong.  Elements are what fragments of html are called.  Hiccup is responsible for generating html from Clojure symbols. The problem is that we've got a map with symbols in it and hiccup thinks they're html.  They're not, so it creates an error.

We can fix the issue by converting our maps to strings.

```clojure
(defn index-view
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

Let's take the messages and put them in a bootstrap components.

```clojure
(defn message-view
 "This generates the HTML for displaying a single message"
 [message]
 [:div.panel.panel-default
   [:div.panel-heading (hiccup/h (:name message))]
   [:div.panel-body (hiccup/h (:message message))]])
```

Now our `index-view` looks like:   (we added boostrap css and called map around the list of messages).

```clojure
(defn index-view
  "This generates the HTML for the index page"
  [messages]
  (page/html5
   [:head
    [:title "chatter"]
    (page/include-css "https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css")]
   [:body
    [:div.container
     [:h1 "Our Chat App"]
     [:div.row (map message-view messages)]]]))
```

Save `handler.clj`, then refresh the browser.  Our hard-coded messages should now display in the page.

Next [Chapter 7: Add Form](/Pages/7-add-form.md)
