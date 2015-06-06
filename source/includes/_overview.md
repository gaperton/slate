#Overview
## What it is
```javascript
var User = Nested.Model.extend({
    urlRoot : '/api/users',

    defaults : {
        // Primitive types
        login    : String, // ""
        email    : String.value( null ), // null
        loginCount : Number.has.toJSON( false ) // 0, not serialized
        active   : Boolean.value( true ), // true

        created  : Date, // new Date()

        settings : Settings, // new Settings()

        // collection of models, received as an array of model ids
        roles    : Role.Collection.subsetOf( rolesCollection ),
        // reference to model, received as model id.
        office   : Office.from( officeCollection )
    }
});

var collection = new User.Collection(); // Collection is already there...
collection.fetch().done( function(){
    var user = collection.first();
    console.log( user.name ); // native properties
    console.log( user.office.name );
    console.log( user.roles.first().name );
});
```

[![Master Build Status](https://travis-ci.org/Volicon/backbone.nestedTypes.svg?branch=master)](https://travis-ci.org/Volicon/backbone.nestedTypes)
[![Develop Build Status](https://travis-ci.org/Volicon/backbone.nestedTypes.svg?branch=develop)](https://travis-ci.org/Volicon/backbone.nestedTypes)

NestedTypes is the type system for JavaScript, implemented on top of  Backbone. It solve common architectural problems of Backbone applications, providing simple yet powerful tools to deal with complex nested data structures. Brief feature list:

- Class and Integer types
- *Native properties* for Model attributes, Collection, and Class.
- Inline Collection definition syntax for Models.
- Model.defaults inheritance and deep copying.
- Type declarations and automatic type casts for Model attributes.
- Easy handling of Date attributes.
- *Nested models* and collections.
- *One-to-many* and *many-to-many* models relations.
- 'change' event bubbling for nested models and collections.
- Attribute-level control for parse/toJSON and event bubbling.
- Run-time type error detection and logging.


It feels much like statically typed programming language. Yet, it's vanilla JavaScript.

> Types are being checked in run-time on assignment, but instead of throwing exceptions it tries to cast values to defined types.

```javascript
    user.login = 1;
    console.assert( user.login === "1" );

    user.active = undefined;
    console.assert( user.active === false );

    user.loginCount = "hjkhjkhfjkhjkfd";
    console.assert( _.isNan( user.loginCount ) );

    user.settings = { timeZone : 180 }; // same as user.settings.set({ timeZone : 180 })
    console.assert( user.settings instanceof Settings );
```
## Why

## Features

## Performance

## Installation & Requirements
> CommonJS (node.js, browserify):

```javascript
var Nested = require( 'nestedtypes' );
```

> CommonJS/AMD (RequireJS).
> 'backbone' and 'underscore' modules must be defined in config paths.

```javascript
require([ 'nestedtypes' ], function( Nested ){ ... });
```

> Browser's script tag

```html
<script src="underscore.js" type="text/javascript"></script>
<script src="backbone.js" type="text/javascript"></script>
<script src="nestedtypes.js" type="text/javascript"></script>
<script> var Model = Nested.Model; ... </script>
```

### Supported JS environments
NestedTypes requires modern JS environment with support for native properties.
It's tested in `IE 9+`, `Chrome`, `Safari`, `Firefox`, which currently gives you about 95%
of all browsers being used for accessing the web.

`node.js` and `io.js` are also supported.

### Packaging and dependencies

NestedTypes itself is packaged as UMD (Universal Module Definition) module, and should load dependencies properly in any environment.

NestedTypes require `underscore` and `backbone` libraries. They either must be included globally with `<script>`tag or, if `CommonJS`/`AMD` loaders are used, be accessible by their standard module names.  

### bower

`bower install backbone.nested-types`

### npm

`npm install backbone.nested-types`

### Manual
Copy `nestedtypes.js` file to desired location.
