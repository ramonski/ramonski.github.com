---
layout: post
title: "Introducing Sencha Cmd"
date: 2012-12-03 11:49
comments: true
categories: web, javascript, mvc, extjs
published: true
---

Sencha Cmd is a command line tool code generator for ExtJS.
It lets you create starter projects, deployment builds and a lot of more things
with ease.

**Download**:
[http://www.sencha.com/products/sencha-cmd/download](http://www.sencha.com/products/sencha-cmd/download)

**Documentation**:
[http://docs.sencha.com/ext-js/4-1/#!/guide/command](http://docs.sencha.com/ext-js/4-1/#!/guide/command)

<!-- more -->


## Prerequisits

You need to have at least
[ExtJS 4.1.1a](http://www.sencha.com/products/extjs/download/ext-js-4.1.1/1683)
downloaded.

On my machine, I downloaded and unpacked it into
`~/develop/js/Sencha/ext-4.1.1a`

You also need to have `ruby` prepared with `compass` and `sass`.

```
$ which ruby
/Users/rbartl/.rvm/rubies/ruby-1.9.3-p194/bin/ruby

$ ruby -v
ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-darwin11.3.0]

$ gem list

*** LOCAL GEMS ***

compass (0.12.2)
sass (3.2.3)
[...]
```

Take a look into [rvm](https://rvm.io)  which lets you manage local `ruby`
installations with sandboxed gemsets.

[rvm installation](https://rvm.io/rvm/install)

... in short:

``` bash
$ cd $HOME
$ \curl -L https://get.rvm.io | bash -s stable --ruby
$ source .rvm/bin/rvm

$ rvm list

rvm rubies

=* ruby-1.9.3-p194 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
```

## The sencha command

After you installed Sencha Cmd, you should have the `sencha` command available
in your terminal:

``` bash
$ which sencha
/Users/rbartl/bin/sencha
```
There are a bunch of arguments available, type `sencha help` to view them.


## Bootstrapping a new application

Now we can use the `sencha generate` command to generate a new MVC like ExtJS App.
[MVC Application Architecture](http://extjs-4.1.1.dev/docs/index.html#!/guide/application_architecture)

``` bash
$ mkdir /tmp/account_manager
$ cd /tmp/account_manager
$ sencha -s ~/develop/js/Sencha/ext-4.1.1a generate app AM .
```

This command copies over the ExtJS library for us and generates a basic
filesystem structure with example content.

``` bash
$ tree -L 2
.
├── app
│   ├── app.js
│   ├── controller
│   ├── model
│   ├── store
│   └── view
├── bootstrap.js
├── build.xml
├── ext
│   ├── bootstrap.js
│   ├── build.xml
│   ├── builds
│   ├── cmd
│   ├── ext-all-debug-w-comments.js
│   ├── ext-all-debug.js
│   ├── ext-all-dev.js
│   ├── ext-all.js
│   ├── ext-debug-w-comments.js
│   ├── ext-debug.js
│   ├── ext-dev.js
│   ├── ext-neptune-debug-w-comments.js
│   ├── ext-neptune-debug.js
│   ├── ext-neptune.js
│   ├── ext.js
│   ├── license.txt
│   ├── locale
│   ├── resources
│   └── src
├── index.html
└── resources
    ├── css
    ├── sass
    └── theme
```


## Serve the new App

A very simple way to serve your app is with this one-liner:

``` bash
python -c "from SimpleHTTPServer import test;test()"
```

Now open up a webbrowser of your choice and go to

[http://localhost:8000](http://localhost:8000)

You should see an empty [border layout](http://extjs-4.1.1.dev/docs/index.html#!/api/Ext.layout.container.Border)
with a west and center area. The center area should contain a single tab titled "Center Tab 1".


Congratulations, you just build a sencha app with one command:)


## Adding a model to the Application

As shown in the [MVC Application Architecture](http://extjs-4.1.1.dev/docs/index.html#!/guide/application_architecture)
tutorial, we're going to add a user model with the following command:

``` bash
$ sencha generate model User id:int,name,email
```

This command adds a `Model.js` to your `app/model` directory.
The content should look like this:

``` javascript
Ext.define('AM.model.User', {
    extend: 'Ext.data.Model',

    fields: [
        { name: 'id', type: 'int' },
        { name: 'name', type: 'auto' },
        { name: 'email', type: 'auto' }
    ]
});
```

## Adding a view to the Application

We would like to have a view for all the Users of our Application

``` bash
$ sencha generate view user.List
```

This will add a view stub called `List.js` in `app/view/user`.

The content of this really basic:

``` javascript
Ext.define("AM.view.user.List", {
    extend: 'Ext.Component',
    html: 'Hello, World!!'
});
```

We have to change it to be useful:

```
Ext.define('AM.view.user.List' ,{
    extend: 'Ext.grid.Panel',
    alias: 'widget.userlist',
    title: 'All Users',

    store: 'Users',

    initComponent: function() {
        this.columns = [
            {header: 'Name',  dataIndex: 'name',  flex: 1},
            {header: 'Email', dataIndex: 'email', flex: 1}
        ];
        this.callParent(arguments);
    }
});
```

As you can see, we reference a store here called `Users`.
There is no sencha command to create a store, so we're going to add it manually.

## Adding a store to the Application

Add a file `Users.js` to your `app/store` directory.
The code should look like this:

``` javascript
Ext.define('AM.store.Users', {
    extend: 'Ext.data.Store',
    model: 'AM.model.User',
    autoLoad: true,

    proxy: {
        type: 'ajax',
        url: '/data/users.json',
        reader: {
            type: 'json',
            root: 'users',
            successProperty: 'success'
        }
    }
});
```
Note the url for the data. it should be located at `/data/users.json`.

Create the direcotry `data` in your application root folder and add the file
`users.json` with the following contents:

``` json
{
    "success": true,
    "users": [
        {"id": 1, "name": 'Ed',    "email": "ed@sencha.com"},
        {"id": 2, "name": 'Tommy', "email": "tommy@sencha.com"}
    ]
}
```

## Putting everything together

What do we have so far:

- A model, which is capable to hold a single user
- A store, which knows how to fetch the data from the server and how to store
  the data internally as instances of our user model
- A grid view, which knows the users store to display a list of users
- Fake data in `data/users.json`

Now we need to get it somehow on the screen.

First we have to modify the `app.js` a little bit, so that it knows our store.
It should look like this:

``` javascript
Ext.application({
    models: ["User"],
    stores: ["Users"],
    controllers: ["Main"],
    views: ["user.List","Main"],
    name: 'AM',

    autoCreateViewport: true
});
```

As we gave our view an `alias` property of `widget.userlist`, we can simply use
the part after the dot directly as an `xtype` in our viewport:

``` javascript
Ext.define('AM.view.Viewport', {
    renderTo: Ext.getBody(),
    extend: 'Ext.container.Viewport',
    requires:[
        'Ext.tab.Panel',
        'Ext.layout.container.Border'
    ],

    layout: {
        type: 'border'
    },

    items: [{
        region: 'west',
        xtype: 'panel',
        title: 'west',
        width: 150
    },{
        region: 'center',
        xtype: 'tabpanel',
        items:[{
            xtype: 'userlist'
        }]
    }]
});
```

After reloading, we should see two user entries in our grid.


## Building a deployment package

Next step is to build an deployment package for our app.
Open up the developer tools console of Google Chrome.

Reload the application with the `Network` Tab opened.
At the bottom of all the requests, there should be a line which shows something like this:

``` plain
217 requests  ❘  3.90MB transferred  ❘  1.32s (onload: 1.32s, DOMContentLoaded: 270ms)
```

Stop the local Python web server and type in the follwing commands

``` bash
$ sencha app build
$ cd build/AM/production
$ ln -s ../../../data .
$ python -c "from SimpleHTTPServer import test;test()"
```

Reload the application with the `Network` Tab opened.
At the bottom of all the requests, there should be a line which shows something like this:

``` plain
8 requests  ❘  1.04MB transferred  ❘  601ms (onload: 373ms, DOMContentLoaded: 373ms)
```

With the deployment version, we reduced the requests from 217 to 8 and the size
is almost 4 times less than before.
