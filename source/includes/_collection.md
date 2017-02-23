# Record.Collection

```javascript
// Implicitly defined collection.
const books = new Book.Collection();

@define
class ComicsShelve extends Book.Collection {
    static itemEvents = {
        // List of model's events we want to be triggered on the collection
        'change:inMyReadingList' : true,
        'customEvent' : true
    }
}

@define
class Comics extends Book {
    // Use custom collection instead of the implicitly created one
    static Collection = ComicsShelve;

    // Extend record's attributes
    static attributes = {
        artist : Author
    }
}
```

Collections are ordered sets of records. You can bind "change" events to be notified when any record in the collection has been modified, listen for "add" and "remove" events, fetch the collection from the server, and use a full suite of Underscore.js methods.

<aside class="info">
Collection is implicitly defined for every record with a constructor accessible as `MyRecord.Collection`. In most cases, you
don't need to declare it manually.
</aside>

<aside class="warning">
Custom Record events and "change:attr" events are not triggered on the collection by default.
You have to use <b>itemEvents</b> spec to enable it for specific events.
</aside>

## model

```javascript
@define
class Library extends Record.Collection {
    static model = Book;
}
```

If defined, you can pass raw attributes objects (and arrays) to add, create, and reset, and the attributes will be converted into a model of the proper type.

This property is being set automatically for collection types referenced as `MyRecord.Collection`. In the majority of cases you don't need to define it explicitly.

```javascript
@define // Serializable abstract Record
class Document extends Record {
    static attrbutes = {
      type : String,
      // attrs definition...
    };

    // Factory method needs to be defined.
    static create( attrs, options ){
        switch( attrs.type ){
            case "public" : return new PublicDocument( attrs, options );
            case "private" : return new PrivateDocument( attrs, options );
        }
    }
}

const Library = Document.Collection;

@define
class PublicDocument extends Document {
  static attributes = {
    type : 'public',
    // attrs definition...
  }
}

@define
class PrivateDocument extends Document {
  static attributes = {
    type : 'private',
    // attrs definition...
  }
}
```

Collection may contain polimorphic records of different types if they are the subclass of the `model`.
You need to define static `Model.create` method to make an abstract model serializable.

<aside class="warning">
<b>Backbone compatibility issue</b>: <i>model</i> values other than Record constructors are <b>not allowed</b>.
</aside>

## constructor( models?, options? ) 

```javascript
var tabs = new TabSet([tab1, tab2, tab3]);
```

When creating a Collection, you may choose to pass in the initial array of models. The collection's comparator may be included as an option. Passing false as the comparator option will prevent sorting. If you define an initialize function, it will be invoked when the collection is created. There are a couple of options that, if provided, are attached to the collection directly: model and comparator.
Pass null for models to create an empty Collection with options.

## models 

Raw access to the JavaScript array of models inside of the collection. Usually you'll want to use get, at, or the Underscore methods to access model objects, but occasionally a direct reference to the array is desired.

## toJSON([options]) 

```javascript
@define
class Man extends Record {
  static attributes = {
    name : '',
    age : 0
  }
}

const collection = new Man.Collection([
  {name: "Tim", age: 5},
  {name: "Ida", age: 26},
  {name: "Rob", age: 55}
]);

alert(JSON.stringify(collection));
```

Return an array containing the attributes hash of each model (via toJSON) in the collection. This can be used to serialize and persist the collection as a whole. The name of this method is a bit confusing, because it conforms to JavaScript's JSON API.

## Underscore Methods (46) 

```javascript
books.each( book => book.publish() );

const titles = books.map( book => book.title );

const publishedBooks = books.filter({ published: true });

const alphabetical = books.sortBy( book => book.author.name.toLowerCase() );

const randomThree = books.sample(3);
```

Proxies to Underscore.js to provide 46 iteration functions on Record.Collection. They aren't all documented here, but you can take a look at the Underscore documentation for the full details…

Most methods can take an object or string to support model-attribute-style predicates or a function that receives the model instance as an argument.

forEach (each)
map (collect)
reduce (foldl, inject)
reduceRight (foldr)
find (detect)
findIndex
findLastIndex
filter (select)
reject
every (all)
some (any)
contains (includes)
invoke
max
min
sortBy
groupBy
shuffle
toArray
size
first (head, take)
initial
rest (tail, drop)
last
without
indexOf
lastIndexOf
isEmpty
chain
difference
sample
partition
countBy
indexBy

## add( models, options? )

Add a record (or an array of records) to the collection, firing an "add" event for each record, and an "update" event afterwards. If a record property is defined, you may also pass raw attributes objects, and have them be vivified as instances of the record. Returns the added (or preexisting, if duplicate) records. Pass {at: index} to splice the record into the collection at the specified index. If you're adding records to the collection that are already in the collection, they'll be ignored, unless you pass {merge: true}, in which case their attributes will be merged into the corresponding records, firing any appropriate "change" events.

var ships = new Backbone.Collection;

ships.on("add", function(ship) {
  alert("Ahoy " + ship.get("name") + "!");
});

ships.add([
  {name: "Flying Dutchman"},
  {name: "Black Pearl"}
]);
Note that adding the same record (a record with the same id) to a collection more than once 
is a no-op.

## remove( records, options? ) 

Remove a record (or an array of records) from the collection, and return them. Each record can be a record instance, an id string or a JS object, any value acceptable as the id argument of collection.get. Fires a "remove" event for each record, and a single "update" event afterwards, unless {silent: true} is passed. The record's index before removal is available to listeners as options.index.

## reset( records, options? )

Adding and removing records one at a time is all well and good, but sometimes you have so many records to change that you'd rather just update the collection in bulk. Use reset to replace a collection with a new list of records (or attribute hashes), triggering a single "reset" event on completion, and without triggering any add or remove events on any records. Returns the newly-set records.

Pass null for records to empty your Collection with options.

Here's an example using reset to bootstrap a collection during initial page load, in a Rails application:

<script>
  var accounts = new Backbone.Collection;
  accounts.reset(<%= @accounts.to_json %>);
</script>
Calling collection.reset() without passing any records as arguments will empty the entire collection.

## set( records, options? )
 
The set method performs a "smart" update of the collection with the passed list of records. If a record in the list isn't yet in the collection it will be added; if the record is already in the collection its attributes will be merged; and if the collection contains any records that aren't present in the list, they'll be removed. All of the appropriate "add", "remove", and "change" events are fired as this happens. Returns the touched records in the collection. If you'd like to customize the behavior, you can disable it with options: {add: false}, {remove: false}, or {merge: false}.

```javascript
const vanHalen = new Man.Collection([ eddie, alex, stone, roth ]);

vanHalen.set([ eddie, alex, stone, hagar ]);

// Fires a "remove" event for roth, and an "add" event for "hagar".
// Updates any of stone, alex, and eddie's attributes that may have
// changed over the years.
```

## get( id ) 
Get a record from a collection, specified by an `id`, a `cid`, or by passing in a record.

```javascript
const book = library.get(110);
```

## at( index ) 

Get a record from a collection, specified by index. Useful if your collection is sorted, and if your collection isn't sorted, at will still retrieve records in insertion order. When passed a negative index, it will retrieve the record from the back of the collection.

## push( record, options? )

Add a record at the end of a collection. Takes the same options as add.

## pop( options? ) 
Remove and return the last record from a collection. Takes the same options as remove.

## unshift( model, options? ) 

Add a model at the beginning of a collection. Takes the same options as add.

## shift( options? ) 
Remove and return the first model from a collection. Takes the same options as remove.

## slice( begin, end ) 

Return a shallow copy of this collection's models, using the same options as native Array#slice.

## length 
Like an array, a Collection maintains a length property, counting the number of models it contains.

## comparator 

By default there is no comparator for a collection. If you define a comparator, it will be used to maintain the collection in sorted order. This means that as models are added, they are inserted at the correct index in collection.models. A comparator can be defined as a sortBy (pass a function that takes a single argument), as a sort (pass a comparator function that expects two arguments), or as a string indicating the attribute to sort by.

"sortBy" comparator functions take a model and return a numeric or string value by which the model should be ordered relative to others. "sort" comparator functions take two models, and return -1 if the first model should come before the second, 0 if they are of the same rank and 1 if the first model should come after. Note that Backbone depends on the arity of your comparator function to determine between the two styles, so be careful if your comparator function is bound.

Note how even though all of the chapters in this example are added backwards, they come out in the proper order:

```javascript
var Chapter  = Backbone.Model;
var chapters = new Backbone.Collection;

chapters.comparator = 'page';

chapters.add(new Chapter({page: 9, title: "The End"}));
chapters.add(new Chapter({page: 5, title: "The Middle"}));
chapters.add(new Chapter({page: 1, title: "The Beginning"}));

alert(chapters.pluck('title'));
```

Collections with a comparator will not automatically re-sort if you later change model attributes, so you may wish to call sort after changing model attributes that would affect the order.

## sort( options? ) 

Force a collection to re-sort itself. You don't need to call this under normal circumstances, as a collection with a comparator will sort itself whenever a model is added. To disable sorting when adding a model, pass {sort: false} to add. Calling sort triggers a "sort" event on the collection.

## pluck( attr ) 

Pluck an attribute from each model in the collection. Equivalent to calling map and returning a single attribute from the iterator.

```javascript
var stooges = new Man.Collection([
  {name: "Curly"},
  {name: "Larry"},
  {name: "Moe"}
]);

var names = stooges.pluck("name");

alert(JSON.stringify(names));
```

## where( attributes ) 

Return an array of all the models in a collection that match the passed attributes. Useful for simple cases of filter.

```javascript
const friends = new Man.Collection([
  {name: "Athos",      job: "Musketeer"},
  {name: "Porthos",    job: "Musketeer"},
  {name: "Aramis",     job: "Musketeer"},
  {name: "d'Artagnan", job: "Guard"},
]);

const musketeers = friends.where({job: "Musketeer"});

alert(musketeers.length);
```

## findWhere( attributes ) 

Just like where, but directly returns only the first model in the collection that matches the passed attributes.

## parse( response, options ) 

parse is called by Backbone whenever a collection's models are returned by the server, in fetch. The function is passed the raw response object, and should return the array of model attributes to be added to the collection. The default implementation is a no-op, simply passing through the JSON response. Override this if you need to work with a preexisting API, or better namespace your responses.

```javascript
var Tweets = Backbone.Collection.extend({
  // The Twitter Search API returns tweets under "results".
  parse: function(response) {
    return response.results;
  }
});
```

## clone()

Returns a new instance of the collection with an identical list of models.