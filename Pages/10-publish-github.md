
### Heroku

Up to this point, we've been running the server on our computer, `localhost`(*). This works great while writing or developing the program because it makes testing changes fast and easy. Eventually we'll want to put it on the internet for others to see and use.

To put it on the internet, we're going to use a company called Heroku for hosting(*). They're going to run our program on a machine visible to anybody on the Internet.  We will use use git to send them our program.

The advantage of using a company like Heroku is that they handle the work of actually maintaining a server. One of the  disadvantages is that Heroku can be expensive, but for a little program like ours it's free.

First, we need to make a couple of changes to our app so it can interact with Heroku.

In `handler.clj`, we will add a couple of functions to print log messages when Heroku starts and stops the program.


```clojure
(defn init []
  (println "chatter is starting"))


(defn destroy []
  (println "chatter is shutting down"))
```

Then we need to add a main function. This is what Heroku will actually invoke(*) to start our program.  Add this to the end of `handler.clj`.

```clojure
(defn -main [& [port]]
  (let [port (Integer. (or port (env :port) 5000))]
    (jetty/run-jetty #'app {:port port :join? false})))
```

Finally, we need to change our `project.clj` so it knows how to prepare our application for Heroku.

Our new `project.clj` should look like:

```clojure
(defproject chatter "0.1.0-SNAPSHOT"
  :description "clojure web app for displaying messages"
  :url "http://example.com/FIXME"
  :min-lein-version "2.0.0"
  :dependencies [[org.clojure/clojure "1.7.0"]
                 [compojure "1.4.0"]
                 [ring/ring-defaults "0.1.5"]
                 [ring/ring-jetty-adapter "1.3.2"]
                 [hiccup "1.0.5"]
                 [hickory "0.5.4"]
                 [environ "1.0.0"]]
  :plugins [[lein-ring "0.9.7"]
            [lein-environ "1.0.0"]]
  :ring {:handler chatter.handler/app
         :init chatter.handler/init
         :destroy chatter.handler/destroy}
  :aot :all
  :main chatter.handler
  :profiles
  {:dev
   {:dependencies [[javax.servlet/servlet-api "2.5"]
                   [ring-mock "0.3.0"]]}
   :production
   {:ring
    {:open-browser? false, :stacktraces? false, :auto-reload? false}
    :env {production true}}}
  :uberjar-name "chatter-standalone.jar")
```

The `ns` of `handler.clj` should look like:

```clojure
(ns chatter.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [ring.middleware.params :refer [wrap-params]]
            [ring.adapter.jetty :as jetty]
            [hiccup.page :as page]
            [hiccup.form :as form]
            [ring.util.anti-forgery :as anti-forgery]
            [environ.core :refer [env]]))
```

We will still be able to start the app using `lein ring server` but now we can also start by entering `lein run -m chatter.handler` in the command line. When we start it this way, it defaults to port 5000, and you should see that in the browser. Port 3000 does not work anymore, but you see the app if you switch to 5000.

If we create a jar with `lein uberjar`, we can also start the app with `java $JVM_OPTS -cp target/chatter-standalone.jar clojure.main -m chatter.handler`

These new methods of starting the app are closer to what Heroku will use to start the app.

We also need a Procfile in the top directory containing the line `web: java $JVM_OPTS -cp target/chatter-standalone.jar clojure.main -m chatter.handler`

Stop the server using control-c, then restart with the command:

```lein with-profile trampoline run```

If the local web page still works; add and commit your changes, then merge your branch back into master.  Push the master branch to github.

Now we'll deploy to heroku with the following commands:
+   `heroku create`
+   `git push heroku master`
+   `heroku ps:scale web=1`
+   `heroku open`

The final command will open the browser and point it at your app on Heroku.

To delete the app from Heroku, select the app in the dash board, click settings, delete app is on the bottom.  Then,`git remote remove heroku` in the command line.

