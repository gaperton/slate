# Nested.Model
## Overview
> Two naive model definitions, which would be wrong in plain Backbone.

```javascript
    var UserInfo = Nested.Model.extend({
        defaults : {
            name : 'test'
        }
    });

    var DetailedUserInfo = UserInfo.extend({
        attributes : { // <- alternative syntax for 'defaults'
            login : '',
            roles : [ 'user' ]
        }
    });

    var user = new DetailedUserInfo();
```

> In Backbone, 'name' attr is not inherited and would be undefined.
> In NestedTypes it's inherited, and you can access it directly.

```javascript
    console.assert( user.name === 'test' );
    user.name = 'admin';
```

> In Backbone all models will share the same instance of [ 'user' ] array. Bug.
> In NestedTypes, user.roles is deep copied on creation. Good practice.

```javascript
    user.roles.push( 'admin' );
```

> This is the ultimate point of NestedTypes. Naive way of doing things is not a bug any more.

This is the most important difference between `Backbone.Model` and `Nested.Model`.

In NestedTypes `defaults` section in not just the place where you may assign
some attributes with defaults values (or may not).
This is, rather, the *specification* of model's attributes. Therefore:

<aside class="warning">
 Every model attribute you're going to use <b>must</b> be mentioned in <b>Model.defaults</b>
</aside>

<aside class="warning">
<b>Model.defaults must</b> be an object. Functions are forbidden.
</aside>

And when you try to set an attribute which doesn't have default value, you'll got an
error in the console.

If you're backbone.js pro, that might sound like a bad news for you, but wait a bit.
Knowing attributes, NestedTypes can do really wonderful things.

If you're not, just don't mind. NestedTypes is designed to make things very comfortable for you,
everything will work just as you might expect.

### Attribute's Native Properties

`Nested.Model` creates native properties
for every attribute mentioned in `defaults`, so you can access them directly
as if they would be regular object members.

<aside class="notice">
You are not required to use <b>model.get</b> and <b>model.set</b> any more.
</aside>

Well, you might need to use `model.set`
in cases when you want to set multiple attributes at once, or to pass some options.

<aside class="notice">
Direct attribute assignment is significantly faster than <b>model.set</b> in most cases.
</aside>

### Defaults deep cloning

When new model is being created, NestedTypes will deep clone
all items (including objects and arrays) from `defaults` object.

<aside class="notice">
You don't need to wrap <b>defaults</b> object in function any more.
</aside>

### Correct defaults inheritance

When extending some existing model definition, NestedTypes will property
merge base model's `defaults`.

<aside class="notice">
You don't need to do manual tricks for <b>defaults</b> inheritance.
</aside>

## model.defaults( [ attrs ], [ options ] )
NestedTypes automatically compile `defaults` object to function.

Generated `defaults` function accepts optional `attrs` argument with attribute values hash
and fills missing attributes with default values. So, following statement can be used to
return every model to its original state:

`model.set( model.defaults() )`

## Model.Collection

```javascript
    var users = new UserInfo.Collection();
    var detailedUsers = new DetailedUserInfo.Collection();
```

Every model definition has its own correct `Collection` type extending base `Model.Collection`.
 Collection.model and Collection.url properties are taken from model.

You could customize collection providing the spec in `Model.collection`, which then will be passed to `BaseModel.Collection.extend`.

```javascript
var DetailedUserInfo = UserInfo.extend({
    urlBase : '/api/detailed_user/',

    defaults : {
        login : '',
        roles : [ 'user' ]
    },

    collection : {
        initialize : function(){
            this.fetch();
        }
    }
});

/*
    DetailedUserInfo.Collection = UserInfo.Collection.extend({
        url : '/api/detailed_user/',
        model : DetailedUserInfo,

        initialize: function(){
            this.fetch();
        }
    });
*/
```

## Inline nested Models and Collections definitions

Simple models and collections can be defined with special shortened syntax.

It's useful in case of deeply nested JS objects, when you previously preferred plain objects and arrays in place of models and collections. Now you could easily convert them to nested types, enjoying nested changes detection and 'deep update' features.

```javascript
var M = Nested.Model.extend({
    defaults :{
        nestedModel : Nested.defaults({ // define model extending base Nested.Model
            a : 1,
            b : MyModel.defaults({ //define model extending specified model
                items : Collection.defaults({ // define collection of nested models
                    a : 1,
                    b : 2
                })

            })
        })
    }
})

```
