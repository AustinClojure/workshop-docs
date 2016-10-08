
### Heroku

Install Heroku

You can use brew: 

```
brew install heroku
```

Or other methods: 

[https://devcenter.heroku.com/articles/heroku-command-line](https://devcenter.heroku.com/articles/heroku-command-line)


Now we'll deploy to heroku with the following commands:
+   `heroku create`
+   `git push heroku master`
+   `heroku ps:scale web=1`
+   `heroku open`

The final command will open the browser and point it at your app on Heroku.

To delete the app from Heroku, select the app in the dash board, click settings, delete app is on the bottom.  Then,`git remote remove heroku` in the command line.

