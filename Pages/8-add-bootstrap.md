### Bootstrap

The app is working but is ugly. We can improve it by using CSS and JavaScript from a package of software called Twitter Bootstrap.


In the head section of our HTML, we're going to include Bootstrap:

```clojure
   [:head
    [:title "chatter"]
    (page/include-css "//maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css")
    (page/include-js  "//maxcdn.bootstrapcdn.com/bootstrap/3.3.1/js/bootstrap.min.js")]
```

[Start the server again](Page%203_Start%20project.md#creating-a-clojure-project), and you will see the fonts change. You'll also notice the table get smashed together.

Now, let's change the table element from `:table` to `:table#messages.table`.

```clojure
    [:table#messages.table
     (map (fn [m] [:tr [:td (:name m)] [:td (:message m)]]) messages)]
```

This tells hiccup that we want the table to have an id of `messages` and a class of `table`. CSS works by looking for combinations of classes and structure and changing the appearance when an element matches a pattern. Bootstrap uses a set of predefined CSS to look for a common set of classes. One of them is table. 

Save the file, then refresh the browser. It now looks better. Examine the HTML that's generated. You see an id and class field inside the table element.

Let's make the table entries stripped by adding an additional class. Change the table element to `:table#messages.table.table-striped`.

```clojure
    [:table#messages.table.table-striped
     (map (fn [m] [:tr [:td (:name m)] [:td (:message m)]]) messages)]
```


What do you think will happen if you change `table-striped` to `table-bordered`? Try it and refresh your browser to find out!

HTML elements can have multiple classes and CSS uses this to create more complex effects. Try adding `table-hover` to the table element: `:table#messages.table.table-bordered.table-hover`.


```clojure
    [:table#messages.table.table-hover
     (map (fn [m] [:tr [:td (:name m)] [:td (:message m)]]) messages)]
```

Now, when you move the mouse over a row, the entire row becomes highlighted. Dynamic effects in the browser are implemented using a language called JavaScript. We won't talk about JavaScript except to say that it exists and the JavaScript part of Bootstrap was imported into the page with the `include-js` call.

Let's create our own CSS file to center the heading.  Create a file `chatter.css` in the `resources/public` directory. Inside, paste:


```css
h1 {
    text-align: center;
}
```

CSS works using pattern matching. In this case, we're saying that if the element is an `h1` element, center the text. Save the file and add another `page/include-css` expression to `handler.clj` to pull in `chatter.css`.

```clojure
   [:head
    [:title "chatter"]
    (page/include-css "//maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css")
    (page/include-js  "//maxcdn.bootstrapcdn.com/bootstrap/3.3.1/js/bootstrap.min.js")
    (page/include-css "/chatter.css")]
```

Refresh the page. We want to see the `h1` tag centered, but you'll see it's not. Open Firebug and watch the traffic as you refresh the page. We're getting a 404 when it's trying to download the css.

The problem is in our `defroutes`. We have a route handling browser GET or POST requests, but anything else is falling through to our `route/not-found` call. We need to tell `defroutes` where to find our resources.  Change the defroutes to:

```clojure
(defroutes app-routes
  (GET "/" [] (index-page @chat-messages))
  (POST "/" {params :params}
    (let [name-param (get params "name")
          msg-param (get params "msg")
          new-messages (update-messages! chat-messages name-param msg-param)]
      (index-page new-messages)
      ))
  (route/resources "/")
  (route/not-found "Not Found"))
```


Next [Chapter 10](/Pages/10-publish-github.md), we will complete the project by pushing your Chatter app live so you can share with your friends!