---
layout: post
title: Introduction to Backbone (part 1-2)
class: backbone
date: 2015-09-21 00:00:00
---

This is the first part of the full tutorial [here][backbone-repo].

Unrelated to this tutorial is a book-in-progress ["Backbone Fundamentals"][osmani-repo] by [Addy Osmani][osmani-blog].

## The Overall Point of Backbone

If you're reading this, then it's likely that your ultimate goal is to understand how to make interactive web sites. One of the difficulties in writing larger and more complex sites is keeping all the code principled and well-organized. For example, consider the way that in Jade templates you can have something like

```Jade
    body
      if someVariable
        p (HTML to render if the variable was true)
      else
        p (HTML to render if the variable wasn't true')
```

or you could have, at the javascript level, an if-statement like

```JavaScript
    if (someVariable){
        response.render('one-thing');
    }
    else {
        response.render('another-thing');
    }
```

There are two different places where we could make the decision on what HTML to render by examining the same variable's value. Is one better than the other? That's a matter of opinion. What's less controversial is that you **don't** want to have a mix of logic in different places. If your project is putting lots of logic in the templates, you don't want to add a new page and have its display logic in the route handler or visa versa because your teammates (or yourself in a week's time) won't know where to look. 

This problem becomes much more pronounced when we consider that sites these days deal with many different kinds of *data*. If we're Twitter, there's users, tweets, followers, lists etc. If we're a photosharing site such as Flickr or Instagram then there's going to be users and images as some of the most important data stored, and there's tags on the images that can be used to search. Essentially any website you'll be using has some kind of data that needs to be recorded in a database, loaded from the server to the client, displayed by rendering HTML, and then modified using the UI of the site, which we can look at as a reversal of the same path back from the DOM down to the server.

For each kind of data, tweets, users, images, etc. that entire pipeline of

    server <===> client code <===> DOM

needs to be handled. There's many possibilities for **how** you organize the code that creates this pipeline, though, and potentially a lot of duplicated code when trying to create a framework that works for all the kinds of data in your application. This is where the concept of *separation of concerns* comes in. We're going to further divide the central "client code" part of our diagram into two components: the *model* and the *view*, which means our diagram is going to instead look like

    server <===> model <===> view <===> DOM

where the *model* is the part that contains **a particular piece of data** on the client side (a tweet, an image, a user profile) and controls syncing the client and server data together, and the *view* is the part that creates the HTML for display the data **and** handles UI actions that will modify the data in the model. 

If we divide things this way, the job of knowing how to write our code gets a bit simpler. If we're writing code that's syncing data with the server, that needs to go into our "model" code for that type of data. If we're writing code that's building the user interface dynamically from data, such as a list of tweets, then that needs to go in our "view" code. 

Of course, there's going to be a lot of replication if we perform this model/view division for all the data in our web app and we **certainly** don't want to write a new framework for each application we work on. We also need to write the glue code that allows the server to communicate with the models, the models communicate with the views, and the views communicate with the DOM. This is where Backbone comes in. 

Backbone is a library that comes with pre-defined model and view classes that you can easily customize for your particular application, with built in code and techniques to keep the models and views in sync with each other. As such, Backbone actually has a lot of the tedious code to setup this model/view dichotomy written once and for all, leaving you to worry about the specifics of your application and not the general problem of separating concerns.

In this tutorial, we'll be covering how to use Backbone for applications. We'll be covering how to 
-   extend Backbone models
-   extend Backbone views
-   connect views to models
-   use events to synchronize models and views
-   use models to communicate with the server
-   render views and insert them into the DOM

The structure of this tutorial is that we'll start first by leaving off the server, so instead we'll be examining just this picture

    model <===> view <===> DOM

and then after getting more comfortable with that picture, we'll add in a server and a database. 

## Installation

In order to get started, you need to download the following files and place them in the `../js` directory of this repository:

-   [Backbone](http://backbonejs.org/backbone.js)
-   [Underscore](http://underscorejs.org/underscore.js)
-   [jQuery](http://code.jquery.com/jquery-2.1.4.js)

or, at least on Linux but possibly OS X if you have "wget" installed, you should be able to run the following shell command to install all of this software locally:

```bash
    mkdir js &&
    cd js && 
    wget http://backbonejs.org/backbone.js && 
    wget http://underscorejs.org/underscore.js && 
    wget http://code.jquery.com/jquery-2.1.4.js
```

## Your First Backbone Project: A Simple Counter

### Outline

In this brief project, we're going to create a client side application that will
-   display a number
-   provide a button that allows you to *increase* the number in the counter

What we're going to cover in this section is: 
-   How to create Backbone models and views
    -   Learn about the specific `get` and `set` methods for Backbone models
-   How to render HTML using a view
-   How to connect a model to a view
-   How to use events to ensure that the **view** updates when the **model** changes and the **model** changes when inputs in the **view** are used

The basic outline is that we'll
1.  create a model
2.  create a view connected to this model
3.  install our event handlers

### Lesson and Example Code

First things first, we need to have our base HTML for the application. In this case, we're going to have a rather simple HTML page that initially contains a `<div>` where we're going to place our counter and a button that we'll use to increment the counter.

file: counter.html

```HTML
    <!doctype html>
    <html>
      <head>
        <title>A Counter Example</title>
        <script type="text/javascript" src="js/jquery-2.1.4.js"></script>
        <script type="text/javascript" src="js/underscore.js"></script>
        <script type="text/javascript" src="js/backbone.js"></script>
        <script type="text/javascript" src="counter.js"></script>
      </head>
      <body>
        <div id="counterdiv"></div>
      </body>
    </html>
```

Now, in our javascript file `counter.js`, the first thing we're going to do is build our *model*. As discussed in our introduction, a model is the thing that **contains** data in our application. All models are built by calling `Backbone.Model.extend(some-object-with-built-in-data)`. We'll talk about the kinds of things we put in `Backbone.Model.extend` as we need them, but to begin with we're going to have a very **simple** model: our goal is to have a single special property called "value" that will contain the value of the counter and is going to be modified by our button. To that end, we are going to include the single property `defaults`, which is a list of default values for the special data of our application. 

file: counter.js

```JavaScript
    var Counter = Backbone.Model.extend({
        defaults : {"value" : 0}
    });
```

You might wonder why we're using `defaults` and not just, say, creating a property of Counter called `value` like in the following code

```JavaScript
    var Counter = Backbone.Model.extend();
    Counter.prototype.value = 0;
```

thus causing any instance of `Counter` to have a property `value` which defaults to 0. The basic reason is that we want to use Backbone's *events* to synchronize the model and the view together. In order to use Backbone events, we don't want to use the built in syntax for object properties but rather the `.get()` and `.set()` methods instead.

The next thing we do in our code is make a *view*, which is going to be similar to be very similar to a model with the exception that we need to define its `render` function, which actually generates HTML from the data in the associated model. We've already decided, using our `defaults` property when creating the `Counter` class, that all counters are going to have a property called `value` which holds the value of the counter. 

```JavaScript
    var CounterView = Backbone.View.extend({
        render: function () {
            var val = this.model.get("value");
            var btn = '<button>Increment</button>';
            this.$el.html('<p>'+val+'</p>' + btn);
        }
    });
```

The next thing we need to do is actually create instances of both our model and a view attached to said model.  The instances need to use DOM elements, so they should be wrapped in a `$(document).ready(function () {})`.

```JavaScript
$(document).ready(function() {
    var counterModel = new Counter();
    
    var counterView = new CounterView({model : counterModel});
    counterView.render();
```

We're almost done, but we still need to set our event handlers. The first one that we're going to do is the `model` event "change", which will fire whenever an attribute of the model changes:

```JavaScript
    counterModel.on("change", function () {
        counterView.render();
    });
```

Specifically, we're saying that whenever the model changes the only thing we need to do is re-render the associated view. This takes care of the direction of 

    model ===> view ===> DOM

but what about the reverse direction?
To do that, we're going to install an event handler on the button so that whenever it is clicked, the counter will increment

```JavaScript
    counterView.$el.on("click","button", function () {
        var mod = counterView.model;
        var currVal = mod.get("value");
        mod.set("value",currVal+1);
    });
```

Finally, we run the code that inserts the `$el` element of the view into the DOM

```JavaScript
    $("#counterdiv").append(counterView.$el);
```

Now, all that's left is to load our page and take a look!

### Exercises

#### Subtraction Button

For this exercise, take the counter example we walked through above and add another button that will *decrement* the counter instead. You'll need to 
1.  modify the render function
2.  modify the existing event handler for the increment function to be more specific
3.  make a new decrement button event handler

1.  Bonus Challenge

    Ensure that the counter **is not decremented** if its value is equal to zero. In other words, not only should the counter's value not dip below 0 but the `change` event in the model shouldn't be triggered if the value is 0. Test and ensure it's not firing by placing a `console.log` statement in the `change` event handler

#### Clear Button

In addition to or perhaps in lieu of the previous exercise, add a button that resets the counter back to 0. Like the previous exercise, you'll need to
1.  modify the render function
2.  modify the existing event handler for the increment button
3.  make a new button to reset the counter

#### Concatenating Text Field

In this exercise, you should start **from scratch** and write a new application that will have
-   an input text field,
-   a button labled concatenate,
-   a place for the entered text to be displayed.
    Whenever the button is pushed, the displayed text (stored in a model) should append to itself whatever string was input.

### Cleaning Up Our Code

There's a little bit of ugliness in our code that was there for the sake of pedagogical order: we're **manually** connecting the event handler for the model back to the view and we're also including too much logic of the **model** in the **view** event handlers. This wasn't so bad for our tiny example, but what if we want to have more than one instance of the model? It's going to be annoying to connect everything together correctly and rewrite the model handling code in each view. We're going to present a bit of a cleaned up version of the code that will be better refactored and show that it's easier to insert multiple model/view pairs into the application. We're going to go a little bit faster than the previous time.

file counterClean.js

```JavaScript
    var Counter = Backbone.Model.extend({
        defaults : {"value" : 0}
    });
    
    Counter.prototype.inc = function () {
        var val = this.get("value");
        this.set("value", val+1);
    }
```

The first thing we're doing is including a method in the `Counter` class for handling the incrementing. The next thing we're going to do is give the `CounterView` class an initialize method that will install the right event handler on the model that will cause the view to be updated whenever the model changes. For convenience, we're also going to use the "events" property of the view to make sure that we install the right event handler for the view upon its creation. 

```JavaScript
    var CounterView = Backbone.View.extend({
        render: function () {
            var val = this.model.get("value");
            var btn = '<button>Increment</button>';
            this.$el.html('<p>'+val+'</p>' + btn);
        },
        initialize: function () {
            this.model.on("change", this.render, this);
        },
        events : {
            'click button' : 'increment'
        },
        increment : function () {
            this.model.inc();
        }
    });
```

Now, within the `$(document).ready` wrapper, we can go ahead and make our models and views and insert them into the DOM.

```JavaScript
    $(document).ready( function () {
      var counterModel1 = new Counter();
      var counterModel2 = new Counter();
    
      var counterView1 = new CounterView({model : counterModel1});
      var counterView2 = new CounterView({model : counterModel2});
    
      counterView1.render();
      counterView2.render();
    
      $("#counterdiv").append(counterView1.$el);
      $("#counterdiv").append(counterView2.$el);
    });
```

#### Questions To Think About

1.  Why do we include the increment button in the view and not the base HTML?
2.  Think about sites you use frequently and sketch out how they might be divided into
    -   models
    -   views
    -   events

### Initialization Objects

Subclasses of Backbone objects (`CounterView` for example) can receive initialization arguments when instances are made by calling the constructor, as follows:
```Javascript
var view = CounterView({model:somemodel});
```

But only certain property names are installed automatically (e.g. 'model', 'collection', 'id').  Custom properties must be manually extracted in the initialize` method, as the follow demo shows:

```Javascript
var MyView = Backbone.View.extend({
    props: function() {
        return Object.keys(this).join(' ');
    }
});

// Subclasses (inherit props):
var MyView2 = MyView.extend({
    initialize: function(opts) {
        // grab particular options
        if (opts)
            this.special = opts.special;
            this.a = opts.a; //...
    }
});

var MyView3 = MyView.extend({
    initialize: function(opts) {
        // grab all options
        _.extend(this,opts); //means merge, not subclass
    }
});

var opts = {a:'a',b:'b',id:'id',model:'mod',special:'yay'};

var view0 = new MyView();
var view1 = new MyView(opts);
var view2 = new MyView2(opts);
var view3 = new MyView3(opts);

view0.props();
view1.props();
view2.props();
view3.props();
```

## Collections Project: Text Lists

### Outline

In this project, we're going to again create a *client side only* application that
-   displays a list of items
-   contains a text field and a submit button that will add the entered text to the list

What we're going to cover in this section is:
-   How to create a Backbone *collection* of models
-   How to create view for a collection
-   How to make the collection's view delegate to individual views
-   How to use the collection specific events to keep the view in-sync

### Lesson and Code

When you're dealing with sites like twitter, or instagram, or anything of that ilk there tend to be **collections** of things. You're reading a *list* of tweets, looking at a *list* of search results, examining a *list* of photos that match a tag, checking a *list* of followers etc. 

In other words, there's a lot of "list-like" things in the data that we're seeing constantly online. This is such a common pattern that Backbone has, built-in, a *Collection* class that allows you to have "lists" of models that can listen for special list-specific events such as adding or removing from the list. 

The basic way that Backbone *collections* work is that you associate to each collection the kind of **model** that it's a list of. You still have individual views for each model, though, and we leave the bulk of the work for handling the display and manipulation of data to the **individual** model/view pairs. We'll also have a view for the **collection**, that will handle how the list is displayed. To this end, we're going to proceed by

1.  writing our base html
2.  defining the model and view for our text data
3.  define the collection for the text data model
    -   this part will be rather simple and bare bones in comparison to the view
4.  define the *view* for our collection
    -   the view will include the framework for displaying the list
    -   the view will also include the button that adds a new element to the collection
        -   this will trigger the `add` event for the collection

We're going to start our application very similar to how our previous project started: with some very simple HTML. 

```HTML
    <!doctype html>
    <html>
      <head>
        <title>Text in Lists</title>
        <script type="text/javascript" src="js/jquery-2.1.4.js"></script>
        <script type="text/javascript" src="js/underscore.js"></script>
        <script type="text/javascript" src="js/backbone.js"></script>
        <script type="text/javascript" src="textlist.js"></script>
      </head>
      <body>
        <div id="listdiv"></div>
      </body>
    </html>
```

Next, we'll start with our basic model of a piece of text. It'll have a "replace" method that will replace the text inside it. Its individual view is going to be an input with the default text of the input set to the value of the model and a "clear" button that will set the text of the model to the empty string ~" "~ . This part is basically the same as our previous project, except that we're going to use a different **kind** of event, `keypress`, for setting the value of the text of the model. In particular, if the key pressed in the input field is the "enter" key, then we call the `replace` operator of the view, which will in turn call the `replace` method of the model.

```JavaScript
    var TextModel = Backbone.Model.extend({
        defaults : {"value" : ""},
        replace : function (str) {
          this.set("value", str);
        }
    });
    
    var TextView = Backbone.View.extend({
        render: function () {
            var textVal = this.model.get("value");
            var btn = '<button>Clear</button>';
            var input = '<input type="text" value="' + textVal + '" />';
            this.$el.html(textVal+"<br><div>" + input + btn + "</div>");
        },
        initialize: function () {
            this.model.on("change", this.render, this);
            // last argument 'this' ensures that render's
            // 'this' means the view, not the model
        },
        events : {
            "click button" : "clear",
            "keypress input" : "updateOnEnter"
        },
        replace : function () {
            var str = this.$el.find("input").val();
            this.model.replace(str);
        },
        clear: function () {
            this.model.replace("");
        },
        updateOnEnter: function (e){
            if(e.keyCode == 13) {
                this.replace();
            }
        }
    });
```
Next, we actually define the collection. This is pretty similar to all the other Backbone classes that we extend, just with the special attribute `model` that we need to match up to the kind of model we want to store in this collection.

```JavaScript
    var TextCollection = Backbone.Collection.extend({
        model : TextModel
    });
```
After this, we need to make our view for the **collection** and write our event handlers for the collection. This is going to be the bulk of our moving parts for this program. The view for the collection will display all of our individual views as well have a button that will add a new "blank" text field into our page (with the default text "Enter something here"). 

```JavaScript
    var TextCollectionView = Backbone.View.extend({
        render : function () {
            var btn = '<button id="addbutton">Add Text</button>';
            var div = '<div id="text-list"></div>';
            this.$el.html(div + btn);
        },
        initialize : function () {
            this.listenTo(this.collection, 'add', this.addView);
        },
        events : {
            "click #addbutton" : "addModel"
        },
        addModel : function () {
            this.collection.add({});
            // collection adds a model, fires add event, then listener calls this.addView(model)
        },
        addView : function (newModel) {
            newModel.set("value","Enter something here...");
            var view = new TextView({model : newModel});
            view.render();
            this.$("#text-list").append(view.$el);
        },
    });
```

There's a few pieces here that we should explain in a bit more detail. First, we're using the more convenient function `listenTo` this time, which in this case means that `this.collection` is now listening on the `add` event and, when it fires, will run `this.addView` with the new model as an argument.  Also, `listenTo` calls the event handler **in the context of the view, not the collection**, so that `this` means the view. Basically, this just lets us avoid including the extra `this` parameter like in our individual model. Calling `addView` takes the newly added model, creates a view for it, renders it, then adds it to the list of views. We use `events` to listen for when the button is clicked and then we run `addModel`. In turn, `addModel` will call the `add` method of the collection. The importance of `add` is that it will simultaneously make a new model and add it to the collection, triggering the `add` event that we're already listening for. 

<aside>
Collections also have a *create* method, which works just like *add* but also persists the new model to a server if there is one.
</aside>

Note that we don't have to say **anything** in the view for the collection about how the view of the individual model works. We just call that individual view's render function and allow it to take care of everything. 

Finally, we go ahead and run the code we need to initialize the whole application:

```JavaScript
    var textCollection = new TextCollection();
    
    var textCollectionView = new TextCollectionView({ collection : textCollection});
    
    textCollectionView.render();
    
    $("#listdiv").append(textCollectionView.$el);
```

### Exercises

#### Delete Button

In this exercise, we're going to add a "delete" button that will erase the bottom element of the list of elements. To do that, you're going to need to 
-   add a delete button to the view of the **collection**;
-   add a event handler that listens for the "remove" event for the collection and refreshes the list, removing the corresponding view from the DOM.
    There's more than one way you could do this, but a simple way might be to use CSS pseudo-selectors to select only the last div in the collection

#### Edited Count

In this exercise, you're going to add a new piece of data to the **base** model: the number of times that it's been edited. Every time the field is edited, it should increment this number. In this case, "edited" means **either** cleared or you've pressed enter while in the input field. You'll need to also modify the view for the base model. 
Question: will you need to modify the view for the collection?

1.  Extra Credit

    To be a little more challenging, make sure that the number-of-times-incremented only increases if the text has actually changed.


[backbone-repo]: https://github.com/portlandcodeschool/backbone-tutorials
[osmani-blog]: https://addyosmani.com/blog/backbone-fundamentals/
[osmani-repo]: https://github.com/addyosmani/backbone-fundamentals/