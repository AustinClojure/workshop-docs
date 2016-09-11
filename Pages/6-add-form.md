
### Adding a Form

We still don't have a way of adding new messages. This requires HTML forms and importing the form functions from hiccup. The form allows the user to send messages in the parameters of an HTML `POST`. We will need to extract the message and add it to our collection of messages. This will be the most complicated set of changes in our app.


> In HTML, a `form` is used to send input from the browser to the server. The `form` element contains a pair of attributes.
>
> `action` - which specifies the route that should handle the input.
>
> `method` - which specifies the type of request.

Up until now, we've only used `GET` to show the messages. To send messages, we'll need to add a `POST`.

> `forms` contain text and `input` elements. The `input` elements define the content the `form` will send to the server. `input` elements have a number of attributes:
>
> `id` - a way of identifying the input
>
>`name` - the name of the input
>
> `type` - the kind of input
>
> `value` - the default value for the input

We're going to use hiccup to generate html that looks like,

```html
<form action="/" method="POST">
  Name: <input id="name" name="name" type="text">
  Message: <input id="msg" name="msg" type="text">
  <input type="submit" value="Submit">
</form>
```

First, we need to import some libraries into our `handler.clj` file.  Add:

```clojure
[hiccup.form :as form]
```

to the `:require` section of the `ns` declaration.  It should now look like:

```clojure
(ns chatter.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [hiccup.page :as page]
            [hiccup.form :as form]))
```

Now that we have imported the hiccup form function, we can use it to generate the HTML form.

```clojure
(form/form-to
 [:post "/"]
 "Name: " (form/text-field "name")
 "Message: " (form/text-field "msg")
 (form/submit-button "Submit"))
```

`form/form-to` is a hiccup function for generating the form.

`[:post "/"]` is a vector with the keyword `:post` and the string "/". This tells hiccup to make the method a `POST` to the `/` location.

`"Name: "` is a string that will be the text displayed before the input field. 

`form/text-field` is a hiccup function for generating an input field of type "text". We're passing in the string "name".

`"Message: "` is a string that will be the text displayed before the input field.

`form/submit-button` is a hiccup function for generating the submit button.

We want to generate the form button below the title but above the list of messages in the `index-page` function.

Finally, we're going to change our definition of the app

from:

```clojure
(def app
  (wrap-defaults app-routes site-defaults))
```

to:

```clojure
(def app app-routes)
```

This is a simplification for the tutorial.

Now our code looks like:

```clojure
(ns chatter.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [hiccup.page :as page]
            [hiccup.form :as form]))

(def chat-messages [{:name "blue" :message "blue's first post"}
                    {:name "red" :message "red is my favorite color"}
                    {:name "green" :message "green makes it go faster"}])

(defn index-page
  "This generates the HTML for displaying messages"
  [messages]
  (page/html5
   [:head
    [:title "chatter"]]
   [:body
    [:h1 "Our Chat App"]
    [:p
     (form/form-to
      [:post "/"]
      "Name: " (form/text-field "name")
      "Message: " (form/text-field "msg")
      (form/submit-button "Submit"))]
    [:p
     [:table
      (map (fn [m] [:tr [:td (:name m)] [:td (:message m)]]) messages)]]]))

(defroutes app-routes
  (GET "/" [] (index-page chat-messages))
  (POST "/" [] (index-page chat-messages))
  (route/not-found "Not Found"))

(def app app-routes)
```

Save ```handler.clj``` and refresh the browser. We should now have a form on the page where a user could submit a new message.

![unwired form](https://github.com/clojurebridge-minneapolis/track1-chatter/blob/master/images/unwired-form.jpg "unwired form")


Next [Chapter 7: Add Atom](/Pages/7-add-atom.md)
