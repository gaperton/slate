#One-to-many and many-to-many model relations

Sometimes when you have one-to-many and many-to-many relationships between Models, it is suitable to transfer such a relationships from server as arrays of model ids. NestedTypes gives you special attribute data types for this situation.

```javascript
var User = Nested.Model.extend({
    defaults : {
        name : String,
        roles : RolesCollection.subsetOf( roles ) // <- serialized as array of model ids
        location : Location.from( locations ) // <- serialized as model id
    }
});

var user = new User({ id: 0 });
user.fetch(); // <- you'll receive from server "{ id: 0, name : 'john', roles : [ 1, 2, 3 ] }"
...
// however, user.roles behaves like normal collection of Roles.
assert( user.roles instanceof Collection );
assert( user.roles.first() instanceof Role );
```

Collection.subsetOf is a collection of models taken from existing collection. On first access of attribute of this type, it will resolve ids to real models from the given master collection.

If master collection is empty and thus references cannot be resolved, it will defer id resolution and just return empty collection or null. If master collection is not empty, it will filter out ids of non-existent models.

Master collection reference may be:
- direct reference to collection object
- string, designating reference to the current model's member relative to 'this'.
- function, which returns reference to collection and executed in the context of the current model.

```javascript
var User = Nested.Model.extend({
    defaults : {
        name : String,
        roles : Collection.subsetOf( 'collection.roles' ); // this.collection.roles
        location : Location.from( function(){ return this.collection.locations; }); // this.collection.locations
    }
});
```

Collection.subsetOf supports some additional methods:
- addAll() - add all models from master collection.
- removeAll() - same as reset().
- toggle( modelOrId ) - toggle specific model in set.
- justOne( modelOrId ) - reset subset to contain just specified model.

There's a global store for the collections, which might be useful in case of bi-directional relationships. It's available as a member of Model (this.store), and globally as Nested.store.

Store needs to be initialized with a hash of collections and models type specs. It can be initialized several times. On first access to every member of the store, it will fetch data from the server automatically. You need to take care of update events.

```javascript
Nested.store = {
    roles : Role.Collection,
    locations : Locations.Collection
}

var User = Nested.Model.extend({
    defaults : {
        name : String,
        roles : Collection.subsetOf( 'store.roles' ); // this.store.roles
        location : Location.from( function(){ return this.store.locations; }); // this.store.locations
    }
});
```

Store behaves as regular model, but provide some additional methods:
- fetch() will update all store members, which are loaded.
- fetch( 'name1', 'name2', ... ) will fetch listed members and return promise.
- clear() will clear all collections in store and return store to allow chained calls.
- clear( 'name1', 'name2', ... ) will clear listed members.

Note, that 'change' events won't be bubbled from models in Collection.subsetOf. Other collection's events will.

For Model.from attribute no model changes will be bubbled.
