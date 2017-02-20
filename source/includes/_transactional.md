# Record and Collection

```
  Transactional  <- Messenger <- Mixable
    |        |
    V        V
Record     Collection
```

There's the set of common functionality for a `Record` and `Collection`
which relates to their behavior as a part of ownership tree.

- as a Messenger, it can send and receive events.
- it can be a part of ownership tree.
- it can be transactionally changed.
- notifies listeners and owner on changes.
- can be serialized to JSON.
- can be validated.

## Ownership tree

dede

## set( data, options? )

## toJSON()

## parse( data, options )

## clone( options )

## validate()

## validationError

## isValid()

## cid