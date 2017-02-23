# Record

```javascript
// ⤹ required to make magic work  
@define class User extends Record {
    // ⤹ attribute's declaration
    static attributes = {
        firstName : '', // ⟵ String type is inferred from the default value
        lastName  : String, // ⟵ Or you can just mention its constructor
        email     : String.value( null ), //⟵ Or you can provide both
        createdAt : Date, // ⟵ And it works for any constructor.
        // And you can attach ⤹ metadata to fine-tune attribute's behavior
        lastLogin : Date.value( null ).has.toJSON( false ) // ⟵ not serializable
    }
}

const user = new User();
console.log( user.createdAt ); // ⟵ this is an instance of Date created for you.

const users = new User.Collection(); // ⟵ Collections are defined automatically. 
users.on( 'changes', () => updateUI( users ) ); // ⟵ listen to the changes.

users.set( json, { parse : true } ); // ⟵ parse raw JSON from the server.
users.updateEach( user => user.firstName = '' ); // ⟵ bulk update triggering 'changes' once
```

Record is the serializable class with typed attributes, observable changes, and custom validation checks.
It differs to the `Model` (which is the subclass of the Record) in the way that it do not implement
the REST endpoint functionality. Just the serialization.

Unlike in Backbone, Record is not the key-value hash. It's the static structure with the pre-defined
 set of attributes of known types. You have to create Record's subclass and declare attributes for every 
 Record of different shape in the way close to that for classes in statically typed languages. 
 The simplest form of attribute declaration specifies an attribute's default value and/or 
 its constructor function.
 
<aside class="warning">
It's not Backbone! <b>Do not</b> try to create Record directly, it won't work. Subclassing the Record class and defining its attributes is mandatory.
</aside>

### How attribute types can be helpful?

Knowledge about attribute types and their metadata is used in different ways. First, NestedTypes applies
a number of sophisticated optimization techniques which makes its data structures an _order of magnitude
faster_ than Backbone models and collections. Part of it is an assembling of an optimized update pipeline 
for every particular attribute based on its type.

Second, it actually _checks and coerces attribute types_ at runtime establishing the guarantee that attribute
always hold the value of declared type, protecting records from
inappropriate assignments. As Records are typically used to handle JSON interactions between client and 
server, they effectively guard the protocol from errors coming from both ends (with errors being reported to the 
console). _Yes, it still is an order of magnitude faster than Backbone with these checks_.

And third, information about attribute types is used for proper automatic serialization. All complex 
data structures you compose out of NestedTypes Records and Collections _are automatically serializable by default_.

NestedTypes attribute specifications are very powerful and allow you to customize every aspect of attribute 
behavior in declarative way by attaching different kinds of metadata. `Date.value( null ).has.toJSON( false )`
is the simplest example of what can be done.

### Aggregated Records and Collections

```javascript
// ⤹ Gonna define attributes later...
@predefine class Comment extends Record {}

Comment.define({ // ⟵ ...like that...
    attributes : {
        author : User,
        body : String,
        replies : Comment.Collection // ...because it references itself.
    }
});

const comment = new Comment(),
    json = {
        author : { firstName : 'John' },
        body : 'Hi',
        replies : [ { body : 'Hello!' }, { body : 'How are you?' } ]
    };

comment.set( json, { parse : true } );
console.log( comment.replies.first().body ); // 'Hello!'
```

Attributes may have any type including other records and collections. The tree formed
 of aggregated records/collections is called "ownership tree", and a lot of operations
 are performed on its elements recursively:

- When record is created, it creates all aggregated objects.
- record.dispose() will dispose aggregated members first.
- record.clone() will clone aggregated records and collections.
- record.isValid() will check the validness of the whole tree.
- record.toJSON() will serialize aggregated members as nested JSON.
- record.set( json, { parse : true }) will update the whole tree with nested JSON. 
- change of tree node will trigger change event on its parents up to the top.

<aside class="notice">
Aggregated Record or Collection might have just <b>one</b> owner.
You <b>cannot</b> assign aggregated record/collection from one record's attribute to another,
  <b>unless</b> the target attribute is <b>declared as shared</b> (and thus it is not a member of an ownership tree).
</aside>

### Shared Records and Collections

While NestedTypes `Record` _aggregates_ its attributes by default, it supports references to shared 
objects in attributes as well. Such an attribute is excluded from the record's `ownership tree`, 
can hold objects belonging to other ownership trees, and is not considered as an integral part of the record 
(not created/disposed/cloned as a part of its owner). There are three kinds of shared attribute types:

- _serializable id-reference_ to records from other collections. Defined as 
`Record.from( collection )` and `Collection.subsetOf( collection )` specs. Such an attributes
 are serialized as record's id and and array of record ids respectively. Nested changes are _not_ observed.
- _non-serializable reference_. Defined as `MyRecord.shared` or `MyCollection.shared`.
Such an attribute cannot be properly restored from the JSON, and thus is excluded from the serialization. 
Nested changes _are_ observed by default.
- _non-serializable collection of references_. Defined as `MyCollection.Refs` (which is actually the 
constructor function). It's the collection which does not take ownership on its elements.

<aside class="notice">
Strict ownership policy is enforced in NestedTypes. While requiring developer to think
 more about the memory management at the design time (which is rather unusual for the languages like JS),
  declarative ownerhip comntrol is proven to eliminate wide class of tricky errors and provide the dramatic 
  improvement in maintainability for the large systems.  
</aside>

## Transactional API

`Record` and `Collection` both extends `Transactional` class, sharing an important part of
API related to the behavior of ownership trees, such as.

## attributes

Static `attributes` member is used to declare record's attributes. For the details
on attributes definition syntax refer to the [Attribute Definition] section.

`record.attributes` instance member holds attribute's values. Must not be manipulated directly.

## constructor( attrs?, options? )

```javascript
@define class Book extends Record {
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
You may access attributes directly without <b>get()</b>. Just <b>note.title</b>.
</aside>

## set( attrs, options? ) 

```javascript
@define class Note extends Record {
    static attributes = {
        title : '',
        content : ''
    }
}

const note = new Note();
note.set({title: "March 20", content: "In his eyes she eclipses..."});

note.title = "A Scandal in Bohemia";
```

Set a hash of attributes (one or many) on the record.
If any of the attributes change the record's state, a `change` event will be triggered on the record.
Change events for specific attributes are also triggered, and you can bind to those as well,
 for example: `change:title`, and `change:content`.

<aside class="notice">
You may assign attributes directly without <b>set()</b>. Just <b>note.title = "A Scandal in Bohemia"</b>.
</aside>

`set()` takes the standard set of transactional `options`.

## id

```javascript
const author = new User();
author.id = 5;
author.name = 'Allan Moore';

users.add( author );
console.assert( users.get( 5 ) === author );
``` 

Predefined record's attribute, the `id` is an arbitrary string (integer id or UUID).
Records can be retrieved by `id` from collections, and there can be just one instance
of the record with the same `id` in the particular collection.

<aside class="notice">
Unlike in Backbone, <b>id</b> is an assignable property.
</aside>

## idAttribute

```javascript
@define class Meal extends Record {
  static idAttribute =  "_id";
  static attributes = {
      _id : Number,
      name : ''
  }
}

const cake = new Meal({ _id: 1, name: "Cake" });
alert("Cake id: " + cake.id);
```

A record's unique identifier is stored under the `id` attribute.
If you're directly communicating with a backend (CouchDB, MongoDB) that uses a different unique key, 
you may set a Record's `idAttribute` to transparently map from that key to id.

Record's `id` property will still be linked to Record's id, no matter which value `idAttribute` has.

## changed

The `changed` property is the internal hash containing all the attributes that have changed since its last set.
Please do not update `changed` directly since its state is internally maintained by `set()`.
A copy of `changed` can be acquired from `changedAttributes()`.

## defaults( attrs? )

```javascript
@define class Meal extends Record {
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

<aside class="warning">
It's not Backbone! <b>Do not</b> try to define <b>defaults()</b> manually. This method is <b>generated automatically</b> from the attribute spec.
</aside>

## toJSON( options? ) 

```javascript
@define class Artist extends Record {
    static attributes = {
        firstName: String,
        lastName: String,
        birthday : Date,
        code : String.has.toJSON( x => x.toLowerCase() )
    }
}

const artist = new Artist({
    firstName: "Wassily",
    lastName: "Kandinsky"
});

artist.birthday = new Date( 1866, 12, 16 );

alert( JSON.stringify( artist ) );
```

Create record's JSON representation. This can be used for persistence, serialization, or for augmentation before being sent to the server.
The name of this method is a bit confusing, as it doesn't actually return a JSON string — but I'm afraid that it's the way that the JavaScript API for JSON.stringify works.

It will, however, produce correct JSON for all the complex attribute types.

<aside class="notice">
Before overriding record's <b>toJSON()</b> method consider adding custom <b>.has.toJSON()</b> serializer to attribute's spec.
</aside>

## isNew()

Has this record been saved to the server yet? If the record does not yet have an `id`, it is considered to be new.

## hasChanged( attr )

```javascript
book.on("change", () => {
  if( book.hasChanged( "title" ) ){
    ...
  }
});
```

Has the record attribute changed during the last transaction? If an attribute is passed, returns true if that specific attribute has changed.

<aside class="notice">
This method, and the following change-related ones, are only useful during the course of a record's "change" event.
</aside>

## changedAttributes( attrs? ) 

Retrieve a hash of only the record's attributes that have changed since the last set,
or false if there are none. Optionally, an external attributes hash can be passed in,
returning the attributes in that hash which differ from the record.
This can be used to figure out which portions of a view should be updated,
or what calls need to be made to sync the changes to the server.

## previous( attr ) 

```javascript
@define class Person extends Record{
    static attributes = {
        name: ''
    }
}

const bill = new Person({
  name: "Bill Smith"
});

bill.on("change:name", ( record, name ) => {
  alert( `Changed name from ${ bill.previous('name') } to ${ name }`);
});

bill.name = "Bill Jones";
```

During a "change" event, this method can be used to get the previous value of a changed attribute.

## previousAttributes()

Return a copy of the record's previous attributes. Useful for getting a diff between versions of a record, or getting back to a valid state after an error occurs.

## Deprecated Backbone methods

<aside class="warning">
It's not Backbone!
</aside>

`has()`, `unset()`, and `clear()` Backbone methods are not supported due to the fact that in NestedTypes `Record` is not a dynamic hash, but static class with typed attributes.

Use `record.attr = void 0` to clear the value for the particular attribute. 

<aside class="notice">
Attributes with undefined value are excluded from serialization, thus such an assignment
behaves practically the same as removing the attribute from Backbone's model.
</aside>

