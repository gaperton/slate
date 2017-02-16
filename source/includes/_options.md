# Attribute metatypes

Record's attributes specification may contain additional metadata controlling
different aspects of attribute's behavior, such as:

* default value
* attribute's serialization
* reactions on attribute's changes
* custom events subscribtion
* etc.

 Object representing these metadata along
with attribute type is called _metatype_.

```javascript
@define
class M extends Record {
    static attributes = {
        time : Date.has.value( null ).toJSON( false )
    }
}
```

`Constructor.has` is used to create metatype wrapping every particular contructor function,
which can be populated with metadata using the chaiable calls of metatypes API.

In the example on the right there is `time` attribute defined of type Date, having the default value
`null`; it is excluded from serializaion.

```javascript
const ipPattern = /^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$/;

const IPAddress = String.has.check( x => !x || ipPattern.test( x ), 'Not valid IP address' ),
      Port = Number.has.check( x => !x || x >= 0 && x <= 65535, 'Must be between 0 and 65535' );

@define
class Server extends Record {
    static attributes = {
        ip : IPAddress.isRequired,
        port : Port.value( null )
    }
}
```

Metatypes are _immutable_ (every chained call will create new metatype), which means that you 
can create _reusable metatypes_. As in the `Server` example with `IPAddress` which is string with 
custom validation, and the `Port` which is the number with a range check.

## .has.value( value )
```javascript
var M = Nested.Model.extend({
    defaults : {
        a : Type.has.value( value ),
        b : Type.value( value )
    }
});
```

Attribute's default value. On model construction, `value` will be casted to `Type` applying usual type casting rules.

<aside class="notice">
`.value` option may be used without leading `.has`.
</aside>

## .has.toJSON( ( value, name ) => any | false )

```javascript
@define
class M extends Model {
    static attributes = {
        a : Type.has.toJSON( x => x.text ),
        b : Type.has.toJSON( false )
    }
}
```

Overrides the default serialization for the given attribute.
Given function will be executed in the context of the record.

Passing `false` will prevent attribute's serialization.

## .parse( ( value, name ) => any )
```javascript
var M = Nested.Model.extend({
    defaults : {
        a : Type.has.parse( function( value ){
            return Type.factory( value );
        })
    }
});
```

Perform custom value transformation before assignment if `{ parse : true }` transaction option
is set. `parse` option is set automatically when doing REST I/O, and can be set manually as well.
Check `set( data, options? )` method documentation for the `Record` and `Collection`.

Given function will be executed in the context of the record.

## .has.get( ( value, name ) => any )
```javascript
@define
class M extends Model {
    static attributes = {
        a : Number.has.get( value => value + 1 )
    }
}
```

Transform the returned value on attribute's read. Does not modify actual value stored in the record.

Given function will be executed in the context of the record.
Multiple get hooks are chainable, and will be applied in specified order.

## .has.set( function( value, name ) )
```javascript
var M = Nested.Model.extend({
    defaults : {
        a : Type.has.set( function( value, name ){
            return value;
        })
    }
});
```

Called during attribute's update in the context of the model *after* type cast but *before* an actual set, allowing you to modify set value.

<aside class="notice">
Set hook is only called when attribute value is changed. For nested models and collections case, it will be called <b>only in case</b> when instance will be replaced, not in case of <b>deep update</b>.
</aside>

Set hook function accepts attribute's `name` and `value` to be set, and returns modified value, or `undefined` to cancel attribute update.

Multiple set hooks are chainable, and will be applied in specified order.

Returned value will be casted to attribute's type applying standard convertion rules. So, it's guaranteed that attribute's value will always hold the correct type.

## .check( predicate, [ error ] )
```javascript
var M = Nested.Model.extend({
    defaults : {
        a : Number.has.check( x => x > 0 ),
        b : Number.has.check( x => x > 0, 'b should be positive' ),
        queue : Collection.has
                .check( x => x && x.length, 'Queue should not be empty' )
                .check( x => x.length <= 5, 'Not more than 5 elements in the queue' )
    }
});
```

Add validator to an attribute. Validators will be checked on `model.save`,
`model.isValid()`, and first access to `model.validationError`.

Validator may contain optional error message. Validators can be chained; in this
 case they are executed in sequence, until first one will fail.

See `validation` section for details.

## .events( eventsMap )
```javascript
var M = Nested.Model.extend({
    defaults : {
        a : Type.has.events({
            'isReady isNotReady' : function(){
                this.trigger( 'imwatchingyou' );
            }
        }),
    }
});
```

Automatically manage events subscription for nested attribute, capable of sending events. Event handlers will be called in the context of of the parent model.

## .changeEvents( String | false )
```javascript
var M = Nested.Model.extend({
    defaults : {
        a : ModelA.has.changeEvents( 'change myEvent' ),
        b : ModelB.has.changeEvents( false ),
    }
});
```
<aside class="notice">
Makes sense only for Model and Collection attributes.
</aside>

Override default list of events used for nested changes detection of selected attribute.

Pass `false` option to disable nested changes detection for this attribute.
