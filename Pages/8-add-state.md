### Saving form data to an Atom

We see the form now, but submitting it does nothing. The problem now is that we're extracting the params during the `POST` but aren't actually doing anything with them.  To fix this, we have to extract the parameters from the form, build a message, and store the message in our
messages vector.

We want to be able to add new messages to our `messages` vector. Clojure was designed from the ground up to make it easier to write concurrent programs. Concurrent programs are programs that do more than one thing at a time. It does that by having data structures that do not change. Variables can be changed to point to something else, but Clojure requires that doing so happens using particular functions, so it can ensur
e the program stays in a safe state. We're going to use the `atom` mechanism to allow us to update our `messages`.


> An `atom` is like a box that protects information from being changed in an unsafe way. You simply pass the information into the `atom`.

Instead of having the `chat-messages` variable point to our vector of messages, we're going to have it point to the `atom` protecting the vector.

Instead of:

```clojure
(def chat-messages [{:name "Bob"    :message "hello, world"}
                    {:name "George" :message "What's up internet"}
                    {:name "Sally"  :message "Hungry for some pizza?"}])
```

We'll use:

```clojure
(def chat-messages 
  (atom [{:name "Bob"    :message "hello, world"}
         {:name "George" :message "What's up internet"}
         {:name "Sally"  :message "Hungry for some pizza?"}]))
```

Now `chat-messages` is pointing to the `atom` protecting our vector of hashes instead of just a vector.

Because `chat-messages` is pointing to the `atom`, we can't simply `map` over it in `index-view`.  We have to dereference it first. We want to tell Clojure that we want to generate HTML for the contents of the atom. This allows Clojure to ensure the messages are always read in a consistent state, even though something could be modifying them.

> Reading what's stored in an `atom` is called "dereferencing" and is represented by the `@` character.

We will dereference the `chat-messages` atom just before it is passed to the `index-view` function. We can do this by changing our routes from:

```clojure
(defroutes app-routes
  (GET "/" [] (index-view chat-messages))
  (POST "/" [] (index-view chat-messages))
  (route/not-found "Not Found"))
```

to (adding @ in front of the var holding the atom):

```clojure
(defroutes app-routes
  (GET "/" [] (index-view @chat-messages))
  (POST "/" [] (index-view @chat-messages))
  (route/not-found "Not Found"))
```

If you save `handler.clj` and refresh the browser, the hard coded examples should display as before. We still won't see any new messages because we still need to extract the information from the form and modify `chat-messages`.

To add messages to `chat-messages`, we will need to introduce two more functions: `conj` and `swap!`.

#### Try this in the repl

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

We'll also make a helper function to make it easier to make a new message and save it to our atom.

```clojure
(defn build-message [name message]
  {:name name :message message})

(defn save-message!
  "This will update a message list atom"
  [messages new-chat-message]
  (swap! messages conj new-chat-message))
```

Now, we have to modify our `app-routes`. We have to make two changes; it needs to extract the form information when somebody `POST`s a new message, and it needs to add the new message to our `chat-messages` before returning the page to the user. Both of these changes need to happen in the `POST` route.

```clojure
(defn save-new-message
  "Add the message as a map into our vector of messages in the atom and redirects"
  [chat-messages name message]
  (save-message! chat-messages (build-message name message))
  (response/redirect "/"))
```

The new `app-routes` looks like this when we call `save-new-message`:

```clojure
(defroutes app-routes
  (GET "/" []
       (index-view @chat-messages))
  (POST "/" [name message]
        (save-new-message chat-messages name message))
  (route/resources "/")
  (route/not-found "Not Found"))
```

Save the `handler.clj` file, we will be able to use the form to add messages to the page.

Since we can add messages through the form now, we can remove our hard-coded messages. Change the messages to an empty vector.

```clojure
(def chat-messages (atom []))
```

Now, the app is taking our new messages, but it's adding new messages to the end. That's going to be hard to read.  We can fix that by changing from a vector to a list.


```clojure
(def chat-messages (atom '()))
```

> Like vectors, lists are sequential collections.  Vectors are better for accessing random elements fast (which we aren't doing). Lists are better at adding an element to the front, which we want to do. Since they are both collections, `conj` works with either.

##### Obligatory Tests (optional)

So, now that we are finished and we can see it happening let's proceed to lock it down with a functional interactions test that validates many of the application assumptinos. Let's write a test that will submit a message and verify its presence on index afterwards:

```clojure
  (testing "main route"
    (-> (session app)
        (visit "/")
		(has (status? 200) "page is found")
		(has (some-text? "Our Chat App"))
		(fill-in [:input.form-control] "Hooman")
		(fill-in [:input.form-control] "Greetings")
		(press [:input.btn.btn-primary])
		(follow-redirect)
		(has (status? 200) "message submitted successfully")
		(has (some-text? "Greetings"))))
```

Run the tests,

```
  $: lein test
```

Can you write an assertion to verify the message is still present upon reload of index?

Now, deploy your app to [heroku](/Pages/10-publish-github.md)
