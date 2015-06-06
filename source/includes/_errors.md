# Nested.errors
NestedTypes detect three error types in the runtime, which will be logged to console using console.error.

## Method overriden with value
```
[Type error](Model.extend) Property "name" conflicts with base class members.
```
It's forbidden for native properties to override members of the base Model. Since native properties are generated for Model.defaults elements, its also forbidden to have attribute names which are the same as members of the base Model.

## Wrong model.set argument
```
[Type Error](Model.set) Attribute hash is not an object: ...
```
First argument of Model.set must be either string, or literal object representing attribute hash.

## Wrong collection.set argument

## Attribute has ho default value
Attempt to set attribute which is not declared in defaults.

`[Type Error](Model.set) Attribute "name" has no default value`
