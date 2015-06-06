# Nested Models and Collections
## Overview
- automatic parsing and serialization
- 'deep updates' and 'deep clone'
- 'change' event bubbling

To define nested model or collection, just annotate attributes with Model or Collection type.

Note, that Backbone's .clone() method will create shallow copy of the root model, while Model.deepClone() and Collection.deepClone() will clone model and collection with all subitems.

```javascript
var User = Nested.Model.extend({
    defaults : {
        name        : String,
        created     : Date,
        group       : GroupModel,
        permissions : PermissionCollection
    }
});

var a = new User(),
    b = a.deepClone();
```

Model/Collection type cast behavior depends on attribute value before assignment:
- If attribute value is null, Model/Collection constructor will be invoked as for usual types.
- If attribute already holds model or collection, *deep update* will be performed instead.

"Deep update" means that model/collection object itself will remain in place, and 'set' method will be used to perform an update.

I.e. this code:

```javascript
var user = new User();
user.group = {
    name: "Admin"
};

user.permissions = [{ id: 5, type: 'full' }];
```

is equivalent of:

```javascript
user.group.set({
   name: "Admin"
};

user.permissions.set( [{ id: 5, type: 'full' }] );
```

This mechanics of 'set' allows you to work with JSON from in case of deeply nested models and collections without the need to override 'parse'. This code (considering that nested attributes defined as models):

```javascript
user.group = {
    nestedModel : {
        deeplyNestedModel : {
            attr : 'value'
        },

        attr : 5
    }
};
```

is almost equivalent of:

```javascript
user.group.nestedModel.deeplyNestedModel.set( 'attr', 'value' );
user.group.nestedModel.set( 'attr', 'value' );
```

but it will fire just single `change` event.

Change events will be bubbled from nested models and collections.
- `change` and `change:attribute` events for any changes in nested models and collections. Multiple `change` events from submodels during bulk updates are carefully joined together, which make it suitable to subscribe View.render to the top model's `change`.
- `replace:attribute` event when model or collection is replaced with new object. You might need it to subscribe for events from submodels.
- It's possible to control event bubbling for every attribute. You can completely disable it, or override the list of events which would be counted as change:

```javascript
var M = Nested.Model.extend({
	defaults: {
		bubbleChanges : ModelOrCollection,

		dontBubble : ModelOrCollection.options({ triggerWhanChanged : false })
		}),

		bubbleCustomEvents : ModelOrCollection.options({
            triggerWhanChanged : 'event1 event2 whatever'
		}),
	}
});
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
