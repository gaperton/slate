# Nested.Model
## Overview
In NestedTypes model definition's `defaults` section is the *specification* of model's attributes.
`attributes` keyword may be used instead of `defaults`.

In `defaults` or `attributes`, you may specify attribute default value, its type, and different options of
attribute behavior. Refer to corresponding sections of the manual for details.

<aside class="warning">
 Every model attribute you're going to use <b>must</b> be mentioned in <b>Model.defaults</b>
</aside>

<aside class="warning">
<b>Model.defaults must</b> be an object. Functions are forbidden.
</aside>

When you try to set an attribute which doesn't have default value, you'll got an
error in the console.

## model.defaults( [ attrs ], [ options ] )
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

NestedTypes automatically created `defaults` function from model attribute's spec.
Base model attribute defaults will be inherited.

`defaults` function accepts optional `attrs` argument with attribute values hash
and fills missing attributes with default values. So, following statement can be used to
return every model to its original state:

`model.set( model.defaults() )`

### default values deep cloning

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

## model.id
In NestedTypes, `model.id` is assignable property, linked to `model.attributes[ model.idAttribute ]`.

`model.id = 5` has the same effect as `model.set( model.idAttribute, 5 )`

## model.attrName

NestedTypes creates native property for every attribute.

`model.attr = val;` has the same effect as `model.set( 'attr', val );`

`val = model.attr;` has the same effect as `val = model.get( 'attr' );`

<aside class="notice">
Accessing individual attributes with native properties is significantly
faster than using <b>get</b> and <b>set</b>.
</aside>

You still might need to use `model.set` in cases when you want to set multiple attributes
at once, or to pass some options.

## model.properties

Custom native properties specification. Refer to `Object.extend` manual section for details.

Most typical use case is calculated properties.

## model.set()

Set model attributes. In NestedTypes, this operation is *type safe*. It's guaranteed that
model attribute will always hold null or value of specified type.

1. Values are converted to proper types. For existing nested models and collections `deep update` may be
invoked. Refer to `Attribute Types` manual section for details.
2. Set hooks are being executed for changing attributes. Refer to `Attribute Options` section for details.
3. Events are being registered for changing attributes. `replace:attr` events are fired,
4. Attribute values are being set, firing regular change events.

On attempt to set an attribute which is not defined, warning message will be printed to console.

In NestedTypes, you can assign individual model attributes directly, and it's faster than using `set`:
    `model.attr = val;`

## model.get( 'attr' )

Get attribute value by name. Returned value can be modified with `get hook` in attribute definition.

In NestedTypes, you can access model attributes directly, and it's faster than `get`:
    `val = model.attr;`

## Deep clone

    `model.deepClone()` or `model.clone({ deep : true })`

Deeply clone model with all nested models, collections, and other complex types.

## Deep get and set

    `x = model.deepGet( 'attr1.attr2.modelId.attr3.objId' )`

Get attribute by dot-separated path.
Model attribute name, model.id or model.cid (for collection attribute), index (for array), or object property name ( for plain objects) may be used as an elements of the path.

If some model in the middle of path doesn't exists, it will return `undefined`.  

    `x = model.deepSet( 'attr1.attr2.modelId.attr3.objId', x )`

Set model value by dot-separated path. If model attribute in the middle of path equals to null, empty model will be created.

## Model.Collection

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
```

Every model definition has its own correct `Collection` type extending base `Model.Collection`.
 `Collection.model` and `Collection.url` properties are taken from model.

You could customize collection providing the spec in `Model.collection`, which then will be passed to `BaseModel.Collection.extend`.

## Model.define()
```javascript
var Tree = Nested.Mode.extend();

Tree.define({
    defaults : {
        branches : Tree.Collection
    }
});
```

Part of `Object.extend` forward declaration syntax. Refer to `Object.extend` manual section for details.

Forward declarations makes possible type-accurate recursive model definitions.

## Serialization
### model.toJSON()

All nested attributes will be serialized automatically.

<aside class="warning">
Normally, you don't need to override this method.
</aside>

You can control serialization of any attribute with `toJSON` attribute option. Most typical use case is to exclude attribute from those which are being sent to the server.

### model.parse()

All nested attributes will be parsed automatically.

<aside class="warning">
Normally, you don't need to override this method.
</aside>

You can control parsing of any attribute with `parse` attribute option. Most typical use case is to create proper model subclass for abstract model attribute.
