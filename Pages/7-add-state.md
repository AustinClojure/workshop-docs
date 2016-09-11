### Saving form data to an Atom

We see the form now, but submitting it does nothing. The problem now is that we're extracting the params during the `POST` but aren't actually doing anything with them.  To fix this, we have to extract the parameters from the form, build a message, and store the message in our
messages vector. This might be the hardest part of our app.

First, we need to import a library to extract the information sent by the form. Add the following to the ```:require``` section,￿

`[ring.middleware.params :refer [wrap-params]]`

Next, change the `app` definition from:

```clojure
(def app app-routes)
```

to:

```clojure
(def app (wrap-params app-routes))
```

This enables us to have access to the information sent back in our `form`.

We want to be able to add new messages to our `messages` vector. Clojure was designed from the ground up to make it easier to write concurrent programs. Concurrent programs are programs that do more than one thing at a time. It does that by having data structures that do not change. Variables can be changed to point to something else, but Clojure requires that doing so happens using particular functions, so it can ensure the program stays in a safe state. We're going to use the `atom` mechanism to allow us to update our `messages`.


> An `atom` is like a box that protects information from being changed in an unsafe way. You simply pass the information into the `atom`.

Instead of having the `chat-messages` variable point to our vector of messages, we're going to have it point to the `atom` protecting the vector.

Instead of:

```clojure
(def chat-messages [{:name "blue" :message "blue's first post"}
                    {:name "red" :message "red is my favorite color"}
                    {:name "green" :message "green makes it go faster"}])
```

We'll use:

```clojure
(def chat-messages
     (atom [{:name "blue" :message "blue's first post"}
            {:name "red" :message "red is my favorite color"}
            {:name "green" :message "green makes it go faster"}]))
```

Now `chat-messages` is pointing to the `atom` protecting our vector of hashes.

Because `chat-messages` is pointing to the `atom`, we can't simply `map` over it in `index-page`.  Now, we have to tell Clojure that we want to generate HTML for the contents of the atom. This allows Clojure to ensure the messages are always read in a consistent
state, even though something could be modifying them.

> Reading what's stored in an `atom` is called "dereferencing" and is represented by the `@` character.

We will dereference the `chat-messages` atom just before it is passed to the `index-page` function. We can do this by changing our routes from:

```clojure
(defroutes app-routes
  (GET "/" [] (index-page chat-messages))
  (POST "/" [] (index-page chat-messages))
  (route/not-found "Not Found"))
```

to:

```clojure
(defroutes app-routes
  (GET "/" [] (index-page @chat-messages))
  (POST "/" [] (index-page @chat-messages))
  (route/not-found "Not Found"))
```

If you save `handler.clj` and refresh the browser, the hard coded examples should display as before. We still won't see any new messages because we still need to extract the information from the form and modify `chat-messages`.

To add messages to `chat-messages`, we will need to introduce two more functions: `conj` and `swap!`.

> #### conj
> There are many ways to work with collections of values in Clojure.  One commonly used function is `conj`. The name is short for "conjoin". This function takes a collection and one or more item(s) to add to the collection. It then returns a _new_ collection without modifying the original collection.
>
> ```clojure
> (conj [:one :two] :three)
> => [:one :two :three]
>
> (conj [:one :two :three] :four :five)
> => [:one :two :three :four :five]
> ```

> #### swap!
> To modify an `atom`, Clojure provides `swap!`.
>
> ```clojure
> (swap! atom update-function arguments...)
> ```
>
> `atom` - the atom to be updated.
> `update-function` - the function that is applied to the value protected by the atom. It returns a new value which will replace the original.
>
> `arguments...` - zero or more arguments to be passed to the `update-function`.
>
> The `swap!` function will:
>  1. Dereference the atom
>  2. Pass this dereferenced value to the `update-function` along with any additional arguments. You can think of it like this: `(update-function @atom arguments...)`
>  3. Safely replace the inner content of the atom with the value returned from the `update-function`, and finally...
>  4. Return the new content of the atom.
>
> ```clojure
> (def a-number (atom 1))
>
> @a-number
> => 1
> (swap! a-number + 2)
> => 3
> @a-number
> => 3
> ```

In our case, we're going to "swap" the content of ```chat-messages``` by
"`conj`ing" a new message onto the vector of messages.

We'll also put it in a helper function to make it easier to maintain.

```clojure
(defn update-messages!
  "This will update a message list atom"
  [messages name new-message]
  (swap! messages conj {:name name :message new-message}))
```

Now, we have to modify our `app-routes`. We have to make two changes; it needs to extract the form information when somebody `POST`s a new message, and it needs to add the new message to our `chat-messages` before returning the page to the user. Both of these changes need to happen in the `POST` route.

The new `app-routes` looks like

```clojure
(defroutes app-routes
  (GET "/" [] (index-page @chat-messages))
  (POST "/" {params :params} (index-page
                               (update-messages! chat-messages (get params "name") (get params "msg"))))
  (route/not-found "Not Found"))
```
1. `{params :params}` is a shorthand notation that tells Clojure to extract all of the data submitted from the HTML form and call that data `params`.
2. `(get params "name")` and `(get params "msg")` extract the values of the "name" and "msg" fields from the form data.
3. The `update-messages!` function is then called with the `chat-messages` atom and the values of the "name" and "msg" fields from the form.
4. After `update-messages!` has added the new message to the inner content of `chat-messages` it returns the new, dereferenced, vector of messages held by the atom.
5. `index-page` is called with the updated collection of messages and builds the HTML response for the user.

Another way of writing this, which may make the intent more clear, is to name some of the intermediate values using `let`. This will allow us to temporarily provide names for the results of some of the expressions.

```clojure
(defroutes app-routes
  (GET "/" [] (index-page @chat-messages))
  (POST "/" {params :params}
    (let [name-param (get params "name")
          msg-param (get params "msg")
          new-messages (update-messages! chat-messages name-param msg-param)]
      (index-page new-messages)))
  (route/not-found "Not Found"))
```

1.  Extract the "name" field from the form data in `params` and name it `name-param`.
2.  Extract the "msg" field from the form data in `params` and name it `msg-param`.
3.  Execute the `update-messages!` function for the chat-messages atom and the values of the previously established `name-param` and `msg-param` names.
4.  Assign the name `new-messages` to the result of `update-messages!`.
5.  Execute `index-page` for the new collection of messages now called `new-messages`.
6.  Return the HTML produced by `index-page` and forget about the names `name-param`, `msg-param`, and `new-messages`.

> #### let
> `let` expressions are used to temporarily associate names with the results of other expressions, similar to how a function assigns names to its arguments. These named values can also be re-used without the cost of re-evaluating the expression that generated them.
>
> (let [name-one expression-one]
>       name-two expression-two]
>   (some-function name-one name-two))
>
> 1. `name-one` - a name for the result of evaluating `expression-one`.
> 2. `name-two` - a name for the result of evaluating `expression-two`.
> 3. Call `some-function` and pass it the values assigned to `name-one` and `name-two`.
> 4. Return the result of the last expression within the `let`, and forget about the names we had created.
>
> ```clojure
> (let [two   2
>       three (+ two 1)]
>   (* two three))
> => 6
> ```

Save the `handler.clj` file, we will be able to use the form to add messages to the page.

Since we can add messages through the form, we can remove our hard-coded messages. Change the messages to an empty vector.

```clojure
(def chat-messages (atom []))
```

Now, the app is taking our new messages, but it's adding new messages to the end. That's going to be hard to read.  We can fix that by changing from a vector to a list.


```clojure
(def chat-messages (atom '()))
```

> Like vectors, lists are sequential collections.  Vectors are better for accessing random elements fast (which we aren't doing). Lists are better at adding an element to the front, which we want to do. Since they are both collections, `conj` works with either.

Our app now looks like:


```clojure
(ns chatter.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [ring.middleware.params :refer [wrap-params]]
            [hiccup.page :as page]
            [hiccup.form :as form]))

(def chat-messages (atom '()))

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

(defn update-messages!
  "This will update a message list atom"
  [messages name message]
  (swap! messages conj  {:name name :message message}))

(defroutes app-routes
  (GET "/" [] (index-page @chat-messages))
  (POST "/" {params :params}
    (let [name-param (get params "name")
          msg-param (get params "msg")
          new-messages (update-messages! chat-messages name-param msg-param)]
      (index-page new-messages)
      ))
  (route/not-found "Not Found"))

(def app (wrap-params app-routes))
```


Next [Chapter 8: Add Bootstrap](/Pages/8-add-bootstrap.md)
