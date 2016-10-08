
### Adding a Form

We still don't have a way of adding new messages. This requires HTML forms and importing the form functions from hiccup. The form allows the user to send messages in the parameters of an HTML `POST`. We will need to extract the message and add it to our collection of messages. This will be the most complicated set of changes in our app.


First the form, using bootstrap components and styles:

```clojure
(defn form-view
 "This generates the HTML for new messages"
  []
  [:div.panel.panel-default
   [:div.panel-body
    [:form.form-horizontal {:action "/" :method "POST"}
     [:div.form-group
      [:label.col-sm-2 "Name:"]
      [:div.col-sm-10
       [:input.form-control {:type "text" :name "name"}]]]
     [:div.form-group
      [:label.col-sm-2 "Message:"]
      [:div.col-sm-10
       [:input.form-control {:type "text" :name "message"}]]]
     [:input.btn.btn-primary {:type "submit" :value "Save"}]]]])
```


Call the form in your `index-view` function just after the `[:h1 "Our Chat App"]`

Next we will wire it up and make it actually work.


Next [Chapter 8: Add State](/Pages/8-add-state.md)
