# Getting Started
## What it is
```javascript
var User = Nested.Model.extend({
    urlRoot : '/api/users',

    defaults : {
        // Primitive types
        login    : "", // String
        email    : String.value( null ), // null, but String
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

var collection = new User.Collection();
collection.fetch().done( function(){
    var user = collection.first();
    console.log( user.name ); // native properties
    console.log( user.office.name );
    console.log( user.roles.first().name );
});
```

[![Master Build Status](https://travis-ci.org/Volicon/backbone.nestedTypes.svg?branch=master)](https://travis-ci.org/Volicon/backbone.nestedTypes)
[![Develop Build Status](https://travis-ci.org/Volicon/backbone.nestedTypes.svg?branch=develop)](https://travis-ci.org/Volicon/backbone.nestedTypes)

NestedTypes is state-of-the-art backbonejs-compatible model framework. It gives you:
* Performance. Compared to backbonejs, model updates are about 20 times faster in Chrome/nodejs, and 4 times faster in other browsers.
* Safety. NestedTypes performs dynamic type casts on model update to ensure that attributes will always hold values of proper types.
* Complex attribute types such as Date, nested models, and collections.
* One-to-many and many-to-many model relationships.

It's achieved with attribute type annotations, which feels much like statically typed programming language. Yet, this annotations are vanilla JavaScript, no transpiler step is required.

NestedTypes was originally designed with an idea to make backbonejs more comfortable for newbiews. In short, we took  intuitive newbie approach to backbonejs, and turn it from the mistake to legal way of doing things.

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
