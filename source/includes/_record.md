# Record

```javascript
@define
class Role extends Record {
    static attributes = {
        name : ''
    }
}

@define
class User extends Record {
    static attributes = {
        name      : 'John Dow',
        email     : '',
        isActive  : true,
        roles     : Role.Collection,
        lastLogin : Date.has.value( null ).toJSON( false )
    }
}

const users = new User.Collection();
users.on( 'changes', () => updateUI( users ) );

users.set( json, { parse : true } );
users.updateEach( user => user.roles = [] );
```

## attributes

Holds attribute's values. Must not be manipulated directly.

## constructor( attrs?, options? )

```javascript
@define
class Book extends Record {
    static attributes = {
        title  : '',
        author : ''
    }
}

const book = new Book({
  title: "One Thousand and One Nights",
  author: "Scheherazade"
});
```

When creating an instance of a record, you can pass in the initial values of the attributes,
 which will be set on the record.

If `{parse: true}` is passed as an option, the attributes will first be converted
 by record's and attribute-level `parse()` before being set on the record.

<aside class="notice">
Always use <b>{parse: true}</b> option when construct records from JSON.
</aside>

## initialize( attrs?, options? )

Called at the end of the `Record` constructor when all attributes are assigned
and record's inner state is properly initialized. Takes the same arguments as
a constructor.

## get( attr ) 

Get the current value of an attribute from the record. For example: `note.get("title")`

<aside class="notice">
You may access attributes directly without `get`. Just `note.title`.
</aside>

## set( attrs, options? ) 

Set a hash of attributes (one or many) on the record.
If any of the attributes change the record's state, a `change` event will be triggered on the record.
Change events for specific attributes are also triggered, and you can bind to those as well,
 for example: `change:title`, and `change:content`.

<aside class="notice">
You may assign attributes directly without `set`. Just `note.title = "A Scandal in Bohemia"`.
</aside>

```javascript
@define
class Note extends Record {
    static attributes = {
        title : '',
        content : ''
    }
}

const note = new Note();
note.set({title: "March 20", content: "In his eyes she eclipses..."});

note.title = "A Scandal in Bohemia";
```

## id

```javascript
const author = new User();
author.id = 5;
author.name = 'Allan Moore';

users.add( author );
console.assert( users.get( 5 ) === author );
``` 

Predefined record's attribute, the id is an arbitrary string (integer id or UUID).
Records can be retrieved by id from collections, and there might be just one instance
of the record with the same id in the particular collection.

<aside class="notice">
Unlike in Backbone, <b>id</b> is an assignable property.
</aside>

## idAttribute

A record's unique identifier is stored under the id attribute.
If you're directly communicating with a backend (CouchDB, MongoDB) that uses a different unique key, 
you may set a Record's `idAttribute` to transparently map from that key to id.

Record's `id` property will still be linked to Record's id, no matter which value `idAttribute` has.

```javascript
@define
class Meal extends Model {
  static idAttribute =  "_id";
  static attributes = {
      _id : Number,
      name : ''
  }
}

const cake = new Meal({ _id: 1, name: "Cake" });
alert("Cake id: " + cake.id);
```

## changed

The changed property is the internal hash containing all the attributes that have changed since its last set.
Please do not update changed directly since its state is internally maintained by set.
A copy of changed can be acquired from changedAttributes.

## defaults( attrs? )

```javascript
@define
class Meal extends Record {
  static attributes = {
    appetizer :  "caesar salad",
    entree :     "ravioli",
    dessert :    "cheesecake"
  }
}

const m = new Meal();
m.dessert = 'nothing';

m.set( m.defaults({ entree : 'pasta' }) );

alert("Dessert will be " + m.dessert );
```

The defaults method returns the hash with attribute's default values.
When called with `attrs` argument, it will fill the missing attribute keys with default values.

## toJSON( options? ) 

Create record's JSON representation. This can be used for persistence, serialization, or for augmentation before being sent to the server.
The name of this method is a bit confusing, as it doesn't actually return a JSON string — but I'm afraid that it's the way that the JavaScript API for JSON.stringify works.

It will, however, produce correct JSON for all the complex attribute types.

```javascript
@define
class Artist extends Record {
    static attributes = {
        firstName: String,
        lastName: String,
        birthday : Date
    }
});

const artist = new Artist({
    firstName: "Wassily",
    lastName: "Kandinsky"
});

artist.birthday = new Date( 1866, 12, 16 );

alert( JSON.stringify( artist ) );
```

## isNew()

Has this model been saved to the server yet? If the model does not yet have an id, it is considered to be new.

## hasChanged( attr )

Has the model attribute changed during the last transaction? If an attribute is passed, returns true if that specific attribute has changed.

<aside class="notice">
This method, and the following change-related ones, are only useful during the course of a record's "change" event.
</aside>

```javascript
book.on("change", () => {
  if( book.hasChanged( "title" ) ) {
    ...
  }
});
```

## changedAttributes( attrs? ) 

Retrieve a hash of only the model's attributes that have changed since the last set,
or false if there are none. Optionally, an external attributes hash can be passed in,
returning the attributes in that hash which differ from the model.
This can be used to figure out which portions of a view should be updated,
or what calls need to be made to sync the changes to the server.

## previous( attr ) 

During a "change" event, this method can be used to get the previous value of a changed attribute.

var bill = new Backbone.Model({
  name: "Bill Smith"
});

bill.on("change:name", function(model, name) {
  alert("Changed name from " + bill.previous("name") + " to " + name);
});

bill.set({name : "Bill Jones"});

## previousAttributes()

Return a copy of the model's previous attributes. Useful for getting a diff between versions of a model, or getting back to a valid state after an error occurs.

## Deprecated methods

`has()`, `unset()`, and `clear()` Backbone methods are not supported due to the fact that in NestedTypes `Record` is not a dynamic hash, but static class with typed attributes.

Use `record.attr = void 0` to clear the value for the particular attribute. Attributes with `undefined` value are excleded from serialization in `toJSON()`.