# Reorganization

We need to clean up a little bit to make the code better.

Change the `project.clj` to be like this:

```
(defproject chatter "0.1.0-SNAPSHOT"  
  :description "Clojure web app for displaying messages"  
  :url "https://github.com/AustinClojure/workshop"
  :dependencies [[org.clojure/clojure "1.8.0"]
                 [compojure "1.5.1"]                 
                 [ring/ring-jetty-adapter "1.5.0"]                 
                 [ring/ring-defaults "0.2.1"]
                 [hickory "0.5.4"]                 
                 [environ "1.0.0"]]
  :plugins [[lein-ring "0.9.7"]            
            [lein-environ "1.0.0"]]
  :ring {:handler chatter.handler/app}
  :profiles {
     :dev {:dependencies [[javax.servlet/servlet-api "2.5"]                               
                          [ring/ring-mock "0.3.0"]]}
                             
     :production {:ring {:open-browser? false
                         :stacktraces? false
                         :auto-reload? false}}
                  :env {production true}}
  :aot :all  
  :main chatter.main  
  :uberjar-name "chatter-standalone.jar")
```



Add a new file in the src directory, this moves the server stuff out of the `handler.clj` file:

```clojure
(ns chatter.main  
 (:require [chatter.handler :as chatter]            
           [environ.core :as env]            
           [ring.adapter.jetty :as jetty]))

(defn -main [& [port]]  
  (let [port (Integer. (or port (env/env :port) 3000))]    
    (jetty/run-jetty #'chatter/app {:port port :join? false})))
```

The things in the `[]` for this function indicates to match all args to the function and give the name of `port` to the first arg. If that turns out empty, it looks to see if there is an environment variable of `port`.

Then change your Handler file to be like this: 

```clojure
(ns chatter.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.params :as params]
            [ring.util.response :as response]))
```

remove the bottom (below `defroutes`) and replace with this:

```clojure
(def app (params/wrap-params app-routes))
```

We have now basically added all the libraries we will need and moved the server stuff to a new file.


